# CPS Architecture Decision Record 005
## Deployment Architecture

- **Status:** Proposed for review
- **Decision date:** 2026-10-11
- **Decision owners:** CPS maintainers
- **Related documents:** CPS Project Master Specification v0.1; CPS Project Constitution v0.1; ADR-001 — Technology Stack v0.1; ADR-002 — SSH Onboarding and Agent Architecture; ADR-003 — Discovery and Capability Detection; ADR-004 — Authentication and RBAC
- **Scope:** How CPS is packaged and deployed, which deployment path is the primary supported target, and how the architecture avoids unnecessary vendor lock-in.

---

## 1. Context

CPS consists of a web frontend, API/control plane, PostgreSQL database, and optional managed-VPS agents. Users should be able to self-host CPS on infrastructure they control, while developers should be able to run it locally and deploy test instances to compatible hosting platforms.

Deployment options differ in their support for persistent processes, writable storage, database connectivity, WebSockets, background tasks, and long-lived agent connections. CPS must not claim compatibility with every platform without verifying the requirements of each deployment mode.

## 2. Decision

### 2.1 Docker Compose is the primary deployment target

The primary self-hosted deployment path will be Docker Compose on a supported Linux host.

The repository should provide a documented Compose-based setup for the CPS control plane and its required services. The exact service split and production hardening profile must be defined during implementation.

The default deployment should make the persistent data locations and configuration explicit, including PostgreSQL data, application configuration, and required secrets. Upgrades and backups must be documented before a production-ready release.

### 2.2 Keep the application portable

Application code must not depend on a specific hosting vendor's proprietary APIs for core functionality.

Use configuration and documented service interfaces for:
- Database connectivity
- Public URL and trusted proxy configuration
- TLS termination or an explicitly documented TLS arrangement
- Persistent storage where required
- Logging and health checks
- Background processing, where needed
- WebSocket/agent transport settings
- Secrets supplied through an appropriate deployment mechanism

Vendor-specific adapters may be added later when they provide clear value, but core behavior must remain usable without them.

### 2.3 Distinguish supported deployment modes

CPS should document deployment support by mode rather than saying “runs everywhere.” Initial modes to evaluate are:

1. **Docker Compose on a Linux VPS** — primary self-hosted target.
2. **Local development** — run frontend, backend, and dependencies in a reproducible developer environment.
3. **Direct Linux deployment** — possible later, but requires documented process management, upgrades, secrets, database, and rollback procedures.
4. **PaaS deployment** — evaluate compatible platforms individually for persistent connections, database, filesystem, and background-task requirements.
5. **Split deployment** — frontend and control plane may be hosted separately only when CORS, authentication cookies/tokens, TLS, trusted origins, and WebSocket connectivity are securely configured.

Each mode must be labelled as supported, experimental, or unsupported based on tests and documented requirements.

### 2.4 Treat frontend hosting and control-plane hosting separately

A static frontend may be deployable to a static hosting platform, but this does not mean the entire CPS control plane can run there.

The backend/control plane must provide the required API, authorization, database access, audit logging, and agent transport. Platforms with execution or connection limits may be unsuitable for the persistent agent connection or other long-running workloads.

A split deployment must use HTTPS/WSS, an explicit trusted-origin configuration, and secure session handling. It must not disable authentication or relax CORS to make a deployment work.

### 2.5 Define transport requirements without assuming a vendor

The agent communication path proposed in ADR-002 requires authenticated outbound connectivity from the managed VPS to the configured CPS endpoint. If WebSocket over TLS is selected, deployment targets must be tested for:
- Long-lived WebSocket support and idle timeouts
- Proxy/load-balancer behavior
- Maximum connection duration or connection quotas
- Reconnect behavior after deploys and restarts
- Routing to the correct CPS instance
- TLS certificate validation
- Required inbound exposure on the control-plane side

A hosting platform that cannot meet these requirements must not be described as supporting persistent agent management unless an alternative, reviewed transport is implemented.

### 2.6 Keep state and secrets explicit

Deployment configuration must clearly separate:
- Non-secret configuration, such as public base URL and log level
- Secrets, such as database credentials, session-signing material, and agent-enrollment signing material
- Persistent state, such as database data and any required application-managed files

Do not bake production secrets into images, commit them to the repository, or expose them to the frontend bundle. Example environment files must use placeholders and be safe to publish.

The exact secrets manager or secret-injection method may vary by deployment environment, but the minimum secure configuration must be documented.

### 2.7 Health checks and operational visibility

The deployed system should provide:
- A liveness check that indicates whether the process is running
- A readiness check that indicates whether required dependencies are usable
- Structured logs with secret redaction
- Clear startup and migration errors
- Documented backup and restore procedures
- A documented upgrade and rollback procedure
- Graceful shutdown behavior for active sessions and agent connections

Health endpoints must not reveal credentials, internal tokens, sensitive configuration, or detailed infrastructure information to unauthenticated users.

### 2.8 Database and migration lifecycle

PostgreSQL is the primary database according to ADR-001. Deployment documentation must specify how database schema migrations are applied, how failures are surfaced, and how operators back up and restore data.

Do not assume that a container filesystem is durable. Database persistence must use a documented volume or an explicitly supported external database configuration.

Schema changes must be reviewed for compatibility with upgrades and rollback. Automatic destructive migrations must not run without a documented, safe migration strategy.

### 2.9 Production and development are separate

Development defaults must not be reused blindly in production. Production documentation should cover:
- Strong unique secrets
- TLS and trusted proxy configuration
- Database network exposure
- Least-privilege service accounts and filesystem permissions
- Firewall and port requirements
- Backup protection and restore verification
- Log rotation and retention
- Upgrade planning and recovery
- Secure handling of agent bootstrap and identity material

CPS should provide a usable secure baseline without requiring users to understand every internal component, but must not silently configure insecure shortcuts.

## 3. Consequences

### Benefits
- Docker Compose gives self-hosters a repeatable baseline.
- Core code remains portable across hosting environments.
- Frontend hosting is not confused with hosting the full control plane.
- Compatibility claims are based on documented tests.
- Operators get explicit guidance for persistence, secrets, upgrades, and backups.

### Costs and trade-offs
- Compose files and deployment documentation need ongoing maintenance.
- PaaS support requires platform-by-platform verification.
- Persistent agent connectivity may constrain some serverless or short-lived environments.
- Split deployments add cross-origin and session-security complexity.
- Upgrade and backup procedures are required before production use.

## 4. Explicit non-decisions

This ADR does not yet freeze:
- The exact Compose service topology
- The reverse proxy or TLS termination component
- Whether a bundled PostgreSQL service or external PostgreSQL is the default in every mode
- The exact backup tooling and retention defaults
- The exact migration command and deployment orchestration
- A list of officially supported PaaS vendors
- Whether direct Linux deployment is supported in the first release
- Whether alternative agent transports will be supported

These must be decided through implementation and verified tests. Do not advertise unverified compatibility.

## 5. Acceptance criteria

Before this ADR is marked accepted:
- [ ] Review it against the Master Specification, Constitution, and ADR-001 through ADR-004.
- [ ] Define the initial Docker Compose topology and required volumes.
- [ ] Document required ports, TLS, proxy, and WebSocket configuration.
- [ ] Document production secret injection and rotation expectations.
- [ ] Define migration, backup, restore, upgrade, and rollback procedures.
- [ ] Define health checks and safe diagnostics.
- [ ] Test a clean install and upgrade on the primary supported Linux target.
- [ ] Evaluate PaaS compatibility individually before listing it as supported.
- [ ] Approve the ADR in the repository.

## 6. Implementation sequence

1. Define the Compose services, network boundaries, and persistent volumes.
2. Define environment configuration and secret handling.
3. Add health/readiness checks and structured redacted logs.
4. Implement repeatable database migration startup behavior.
5. Document TLS, proxy, firewall, and agent transport requirements.
6. Add backup/restore and upgrade/rollback documentation.
7. Test clean installation, restart, backup/restore, and upgrade on a supported Linux VPS.
8. Evaluate optional deployment targets independently.

**Current status: PROPOSED FOR REVIEW — not yet frozen.**
