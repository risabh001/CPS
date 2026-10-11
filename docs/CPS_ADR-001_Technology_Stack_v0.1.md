# ADR-001 — CPS Technology Stack
## Status: PROPOSED FOR FREEZE
## Date: 2026-10-08

### Decision

CPS will use a deliberately small, portable stack:

| Layer | Decision |
|---|---|
| Web UI | React + TypeScript |
| Frontend build | Vite |
| Backend/API | Node.js + TypeScript + Fastify |
| Database | PostgreSQL |
| Database access | Drizzle ORM + PostgreSQL driver |
| VPS Agent | Go |
| Monorepo | pnpm workspace |
| Terminal UI | xterm.js |
| Code editor | Monaco Editor |
| Primary deployment | Docker / Docker Compose |
| API style | HTTP API + WebSocket where realtime communication is required |
| Source control | GitHub |

---

## 1. Why This Stack

### React + TypeScript

Chosen for the web panel because CPS is a large interactive UI with:

- file manager
- terminal
- editor
- monitoring
- server management
- RBAC
- plugin UI

TypeScript gives the frontend and backend a shared language and allows common type definitions to be reused.

### Vite

Vite provides a straightforward React/TypeScript development and build workflow and supports a React TypeScript template. citeturn0search14

### Node.js + TypeScript + Fastify

Fastify is selected for the CPS API.

Reasons:

- TypeScript support
- low-overhead Node.js framework
- plugin architecture
- suitable HTTP API foundation
- WebSocket integration can be added without replacing the backend architecture

Fastify officially documents TypeScript support and its plugin model. citeturn0search0turn0search4

The exact Node.js major version will be selected from a supported LTS release when the repository is initialized and then pinned through the project tooling. We will not hard-code an obsolete version in this ADR.

### PostgreSQL

PostgreSQL is the primary persistent database.

Reasons:

- mature relational database
- strong transactional model
- suitable for users, organizations, roles, permissions, servers, sessions and audit records
- widely available in self-hosted and managed environments

PostgreSQL 18 is currently the documented current major release, with multiple supported major versions. citeturn0search1turn0search6

CPS will avoid depending on PostgreSQL-version-specific features unless required.

### Drizzle ORM

Drizzle is selected as the TypeScript database layer.

Reasons:

- native PostgreSQL support
- TypeScript-first schema/query model
- migration tooling
- relatively lightweight abstraction

Drizzle officially documents PostgreSQL support through `node-postgres` and `postgres.js`. citeturn1search0turn1search4

The underlying PostgreSQL database remains the source of truth; the ORM must not hide important database constraints or security decisions.

### Go Agent

The VPS Agent will be written in Go.

Reasons:

- compiled native binary
- low runtime overhead
- good fit for Linux server software
- strong standard library
- concurrency support
- easy distribution as a single binary
- suitable for filesystem, process, PTY and system integration

Go is officially described as a compiled, statically typed language with built-in concurrency mechanisms and a robust standard library. citeturn0search9turn0search13

### pnpm Workspace

The JavaScript/TypeScript portion will use a pnpm workspace.

This allows:

```text
apps/web
apps/api
packages/*
```

to share packages and types while remaining a single repository.

pnpm officially supports monorepositories through workspaces. citeturn1search5

We will not add a larger monorepo orchestration tool such as Turborepo until there is a measured need.

### xterm.js

The browser terminal UI will use xterm.js or a compatible terminal-emulator library selected during implementation.

The terminal backend remains a real PTY. A browser terminal component must not be mistaken for the PTY itself.

### Monaco Editor

The file editor will use Monaco Editor or an equivalent browser code editor.

The exact editor integration is not required for Phase 1 and can be finalized during the editor phase.

---

## 2. Runtime Architecture

Target:

```text
Browser
   |
HTTPS
   |
CPS Web/API
   |
PostgreSQL
```

For realtime operations:

```text
Browser
   |
WebSocket
   |
CPS API / session layer
   |
Agent
   |
PTY / monitoring / realtime operation
```

The API and WebSocket layers must share the same authentication and authorization model.

---

## 3. Agent Architecture

The Agent is intentionally separated from the Node.js control plane.

```text
CPS API
   |
Agent protocol
   |
Go Agent
   |
Linux
```

The Go Agent handles server-local operations.

The Node.js API handles control-plane responsibilities.

This separation prevents the control-plane runtime from becoming responsible for directly implementing every Linux-specific operation.

---

## 4. Database Architecture

Initial database:

```text
PostgreSQL
```

Expected domains:

```text
users
organizations
roles
permissions
user_roles
role_permissions
scopes
servers
agents
server_capabilities
sessions
audit_logs
plugins
settings
```

Redis is explicitly **not** part of the mandatory initial stack.

It may be introduced later if a measured requirement appears, such as:

- distributed coordination
- high-volume ephemeral state
- queueing
- caching

No Redis dependency should be added merely because it is common in other architectures.

---

## 5. API Technology

The API will be HTTP-based.

Use HTTP endpoints for:

- authentication
- users
- roles
- permissions
- server registration
- discovery
- file metadata
- settings
- audit logs
- plugin metadata

Use WebSockets for operations requiring realtime bidirectional communication, especially:

- terminal input/output
- live metrics where appropriate
- agent/session events

The exact WebSocket library/protocol framing remains part of the agent/protocol ADR.

---

## 6. File Manager Architecture

The browser must never receive unrestricted filesystem access.

The intended path is:

```text
Browser
 ↓
CPS authorization
 ↓
Agent/session
 ↓
filesystem
```

Every filesystem operation must be checked against:

- authenticated user
- role
- permission
- server scope
- filesystem/path scope

Path traversal and symlink escape protections are mandatory.

---

## 7. Terminal Architecture

The terminal must be a real PTY.

```text
Browser terminal UI
        |
      WSS
        |
CPS terminal session
        |
      Agent
        |
      PTY
        |
      Shell
```

The terminal UI library does not provide security or authorization.

Authorization is enforced by CPS before the session is established and remains associated with the session.

---

## 8. Deployment

Primary supported deployment:

```text
Docker Compose
```

Target conceptual deployment:

```text
cps-web/api
postgres
```

The Agent is installed on managed VPSs.

Optional future topology:

```text
CPS
 |
 +-- VPS Agent A
 +-- VPS Agent B
 +-- VPS Agent C
```

The application must not require local persistent filesystem storage for core database state.

---

## 9. Development Environment

The VyuhStack VPS is the current CPS test/sandbox environment:

```text
1 vCPU
1 GB RAM
10 GB storage
```

Because of its limited resources, the development stack should remain lightweight.

Do not introduce additional infrastructure services unless required.

---

## 10. What Is NOT Frozen Yet

The following remain separate technical decisions:

### ADR-002
SSH + Agent Architecture

Questions:

- SSH library
- credential handling
- agent bootstrap
- agent identity
- agent authentication
- persistent connection protocol
- reconnect behavior
- offline behavior

### ADR-003
VPS Discovery + Capability System

Questions:

- discovery commands
- OS detection
- privilege handling
- command portability
- capability schema
- caching
- refresh behavior

### ADR-004
Authentication + RBAC

Questions:

- local authentication
- OAuth/OIDC
- sessions
- token strategy
- organization model
- role inheritance
- scope model
- permission evaluation

### ADR-005
Deployment Architecture

Questions:

- Docker images
- Compose topology
- environment configuration
- database initialization
- reverse proxy
- PaaS compatibility
- Vercel/Render deployment boundaries

---

## 11. Version Policy

Do not hard-code dependency versions in this ADR.

When implementation begins:

1. Select supported current versions.
2. Pin versions in package manifests/lockfiles.
3. Record runtime requirements.
4. Test the exact versions in CI.
5. Update through controlled dependency changes.

The architecture should remain stable even when dependency versions change.

---

## 12. Rejected Alternatives

### Full JavaScript/TypeScript Agent

Rejected for the initial design because the VPS Agent is system-level infrastructure and should be a lightweight native binary.

### Python Agent

Not selected for the initial design because it adds a runtime dependency and packaging burden on managed VPSs.

### Java Agent

Not selected because a JVM is unnecessary overhead for the lightweight system agent.

### Redis as a Required Dependency

Rejected. No current requirement justifies making Redis mandatory.

### Kubernetes as a Requirement

Rejected. CPS is intended to run on ordinary VPSs and should not require Kubernetes.

### Vendor-Specific Backend

Rejected. CPS must remain self-hostable and vendor-neutral.

### MineDrop-Specific Architecture

Rejected. MineDrop is a deployment/integration, not CPS Core.

---

## 13. Acceptance Criteria

ADR-001 is accepted when:

- the stack is documented
- the stack can be implemented in the planned monorepo
- the stack supports self-hosting
- the stack supports the VPS Agent architecture
- the stack supports a real PTY terminal
- the stack supports PostgreSQL-backed RBAC/audit data
- the stack does not require Redis
- the stack does not require a specific cloud provider
- unresolved architecture questions are moved into later ADRs

---

## Status

**PROPOSED FOR FREEZE**

Next:

**ADR-002 — SSH + Agent Architecture**
