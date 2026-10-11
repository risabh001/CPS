# CPS Documentation and Decision Register v0.1

- **Status:** Working register
- **Last updated:** 2026-10-11
- **Purpose:** Track CPS's foundational documents, architecture decisions, unresolved questions, and readiness for implementation.
- **Source of truth:** The CPS GitHub repository. Local/generated copies are working artifacts until committed and reviewed in the repository.

---

## 1. Document register

| ID | Document | Purpose | Current status |
|---|---|---|---|
| DOC-001 | `README.md` | Project introduction and entry point | Created in GitHub; content review pending |
| DOC-002 | `LICENSE` | Open-source license | Created in GitHub; license choice/content should be verified |
| DOC-003 | `CONTRIBUTING.md` | Contribution workflow | Created in GitHub; content review pending |
| DOC-004 | `SECURITY.md` | Security reporting and policy | Created in GitHub; content review pending |
| DOC-005 | `CODE_OF_CONDUCT.md` | Contributor conduct | Created in GitHub; content review pending |
| SPEC-001 | `CPS_Project_Master_Specification_v0.1.md` | Full product vision, scope, architecture, security, phases, and workflow | Draft created; repository commit/approval not confirmed |
| CONST-001 | `CPS_Project_Constitution_v0.1.md` | Foundational principles and constraints | Draft created; repository commit/approval not confirmed |
| ADR-001 | `CPS_ADR-001_Technology_Stack_v0.1.md` | Technology stack | Proposed for freeze; explicit approval not confirmed |
| ADR-002 | `CPS_ADR-002_SSH_and_Agent_Architecture_v0.1.md` | SSH onboarding and agent lifecycle | Proposed for review; explicit approval not confirmed |
| ADR-003 | `CPS_ADR-003_Discovery_and_Capability_Detection_v0.1.md` | Discovery and dynamic capabilities | Proposed for review; explicit approval not confirmed |
| ADR-004 | `CPS_ADR-004_Authentication_and_RBAC_v0.1.md` | Authentication, authorization, roles, and scopes | Proposed for review; explicit approval not confirmed |
| ADR-005 | `CPS_ADR-005_Deployment_Architecture_v0.1.md` | Deployment modes and self-hosting | Proposed for review; explicit approval not confirmed |
| REG-001 | `CPS_Documentation_and_Decision_Register_v0.1.md` | Index and status tracker for foundational documentation | This document; add to repository after review |

**Important:** “Created” or “Done” means the file/task was prepared by the user/assistant; it does not by itself confirm that the file is committed, reviewed, or formally approved in GitHub.

## 2. Architecture decision register

| Decision ID | Topic | Current direction | Open items before acceptance |
|---|---|---|---|
| ADR-001 | Technology stack | React + TypeScript; Vite; Node.js + TypeScript + Fastify; PostgreSQL; Drizzle; Go agent; pnpm workspace; xterm.js; Monaco; Docker Compose; HTTP + WebSocket; GitHub as source of truth | Confirm selected libraries and versions during implementation; review and explicitly approve the ADR |
| ADR-002 | SSH and agent architecture | Initial SSH connection and bounded discovery; optional explicit agent bootstrap; outbound authenticated TLS connection preferred; versioned protocol; operation-level authorization | Credential lifecycle; durable agent identity and rotation; privilege model; protocol schema; transport compatibility; SSH-only feature matrix |
| ADR-003 | Discovery and capability detection | Bounded metadata discovery; normalized profile; capability states `available`, `unavailable`, `restricted`, `unknown`, `stale`; UI gates modules but does not authorize operations | Final profile schema; probe list and limits; capability IDs; freshness rules; retention policy |
| ADR-004 | Authentication and RBAC | Deny-by-default; organization/membership/role/permission/server/resource scopes; server-side checks; WebSocket and terminal reauthorization; audit events | Authentication/session approach; role grants; permission registry; scope and conflict semantics; session revocation; audit retention |
| ADR-005 | Deployment architecture | Docker Compose on Linux as primary self-hosted target; portable core; deployment support verified by mode; explicit persistence and secrets | Compose topology; ports/proxy/TLS; secret injection; migration/backup/restore/upgrade/rollback; verified platform support list |

The “current direction” column summarizes the drafts and is not a declaration that they are frozen.

## 3. Cross-document consistency checks

Run these checks before implementation begins:

- [ ] **Source of truth:** Confirm the GitHub repository is authoritative and local/AI-generated copies are not treated as approved automatically.
- [ ] **Naming:** Use `CPS` consistently as the current project name; identify it as temporary if that remains the project decision.
- [ ] **Technology:** Ensure the Master Specification, Constitution, and ADR-001 agree on the stack and agent language.
- [ ] **Connection lifecycle:** Ensure ADR-002 and ADR-003 agree on SSH-first discovery and the transition to agent-based capability checks.
- [ ] **Authorization:** Ensure all modules, APIs, WebSocket sessions, background jobs, and agent operations follow ADR-004.
- [ ] **Capability versus permission:** Ensure a detected capability never grants a user permission.
- [ ] **Deployment:** Ensure ADR-002's outbound agent transport is compatible with the deployment modes claimed in ADR-005.
- [ ] **Secrets:** Ensure SSH credentials, bootstrap tokens, agent identity, session secrets, and database credentials have documented lifecycles.
- [ ] **Audit:** Ensure sensitive actions are audited without logging secrets or terminal content by default.
- [ ] **Persistence:** Ensure PostgreSQL persistence, migrations, backup, restore, and upgrades are accounted for.
- [ ] **Security gates:** Ensure tests for tenant isolation, path traversal, terminal session authorization, agent spoofing/replay, and secret leakage are required before release.
- [ ] **Scope discipline:** Ensure no optional feature or hosting platform is described as supported until its requirements and tests are satisfied.

## 4. Open decision backlog

Resolve these items in focused design tasks; do not silently assume answers.

### P0 — Before implementation of the affected subsystem
1. **Authentication and sessions:** Choose a vetted implementation approach and define session expiry/revocation.
2. **Agent identity:** Define enrollment, identity storage, revocation, and rotation.
3. **Agent privilege model:** Define which operations require elevated OS privileges and how the agent limits them.
4. **Authorization semantics:** Define role grants, scope matching, and conflict behavior.
5. **Protocol contract:** Define versioned agent messages, validation, size limits, errors, and correlation IDs.
6. **Credential lifecycle:** Define how SSH credentials are stored, used, redacted, rotated, and deleted.
7. **Deployment baseline:** Define Compose topology, required ports, secrets, persistent volumes, and migrations.

### P1 — Before related features leave their initial implementation phase
1. Final normalized VPS profile schema.
2. Discovery probes, timeouts, output limits, and distro coverage.
3. Capability registry and freshness/TTL policy.
4. Audit schema, retention, and access controls.
5. File path and symlink boundary rules.
6. Terminal session revocation and reconnect behavior.
7. Backup, restore, upgrade, and rollback procedure.
8. Official deployment support matrix.

### P2 — Before beta or third-party extensibility
1. Plugin API and capability-provider contract.
2. Direct Linux installation support, if still desired.
3. PaaS-specific support claims, each backed by tests.
4. Optional AI integration and its permission boundary.
5. Additional identity providers/MFA, if included in scope.

## 5. Implementation readiness gate

Do not begin broad product implementation until all of the following are true:

- [ ] Foundational documents are committed to GitHub.
- [ ] The project owner has explicitly accepted or rejected each ADR.
- [ ] Conflicts between the Specification, Constitution, and ADRs are resolved or recorded as open decisions.
- [ ] The first milestone has a written scope and measurable acceptance criteria.
- [ ] The repository's build, test, lint, and type-check commands are documented.
- [ ] CI runs the required checks on pull requests.
- [ ] Security-sensitive work has test requirements before coding begins.
- [ ] Every coding-agent task is small, bounded, and independently reviewable.

This gate does not prohibit a small repository/tooling setup task; it prevents asking an agent to build the whole product before the architecture and tests are ready.

## 6. Proposed first implementation milestone

### Milestone 1 — Repository and backend foundation

**Goal:** Create a reproducible development baseline without implementing remote VPS management features prematurely.

Candidate scope:
1. Set up the pnpm workspace and repository structure.
2. Create minimal frontend and Fastify API applications.
3. Add TypeScript configuration and consistent formatting/linting.
4. Add unit-test infrastructure and a small smoke test for each app.
5. Add PostgreSQL development configuration and a migration workflow, once the database schema decision is ready.
6. Add CI for install, type-check, lint, tests, and build.
7. Document local development commands and environment configuration.
8. Ensure no secrets are committed and example environment files contain placeholders only.

**Out of scope for this milestone:** SSH connection, agent bootstrap, filesystem operations, terminal, Docker control, production authentication, and broad UI feature work. These should be implemented in later, separately reviewed tasks after their contracts are accepted.

**Acceptance criteria:**
- A clean checkout can install dependencies using the documented command.
- Frontend and API can start locally using documented commands.
- Type-check, lint, tests, and builds pass in CI.
- Environment examples contain no real credentials.
- The pull request includes tests and a concise summary of files and decisions changed.

This milestone is a proposal and should be approved before a coding agent starts it.

## 7. Suggested repository layout

The layout is illustrative and should be adjusted to the accepted stack and actual repository:

```text
/
├── apps/
│   ├── web/
│   └── api/
├── packages/
│   ├── shared/
│   └── config/
├── agent/
├── docs/
│   ├── CPS_Project_Master_Specification_v0.1.md
│   ├── CPS_Project_Constitution_v0.1.md
│   ├── adr/
│   │   ├── CPS_ADR-001_Technology_Stack_v0.1.md
│   │   ├── CPS_ADR-002_SSH_and_Agent_Architecture_v0.1.md
│   │   ├── CPS_ADR-003_Discovery_and_Capability_Detection_v0.1.md
│   │   ├── CPS_ADR-004_Authentication_and_RBAC_v0.1.md
│   │   └── CPS_ADR-005_Deployment_Architecture_v0.1.md
│   └── CPS_Documentation_and_Decision_Register_v0.1.md
├── docker-compose.yml
├── pnpm-workspace.yaml
├── package.json
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── SECURITY.md
└── CODE_OF_CONDUCT.md
```

Do not create empty applications or an agent just to match this tree. Add directories when their implementation starts.

## 8. Change-control rules

- A proposal is not accepted until explicitly approved and committed.
- Accepted ADRs should be changed through a documented amendment or superseding ADR.
- If implementation discovers a contradiction, pause the affected task and record the issue rather than silently changing architecture.
- Each coding-agent task must state scope, files/components expected to change, prohibited scope, tests, and acceptance criteria.
- The assistant and coding agents are contributors, not the source of truth; the GitHub repository and approved decisions are authoritative.

**Current status: WORKING REGISTER — review and commit to GitHub before relying on it as the official register.**
