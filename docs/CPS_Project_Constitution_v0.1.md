# CPS — Project Constitution v0.1

## Status
FOUNDATION / PRE-CODING

This document freezes the project's current high-level rules and decisions.
Technical details that are not explicitly frozen here remain open until a technical decision is made.

---

## 1. Mission

CPS is an open-source, self-hostable, universal Linux VPS/server management platform.

Primary workflow:

SSH credentials → connect → discover → build server profile → detect capabilities → manage the server through a web interface.

MineDrop is the first serious deployment/use case, but CPS Core must remain vendor-neutral.

---

## 2. Non-Goals for the Initial Release

CPS Core will NOT initially be:

- a MineDrop-only management panel
- a Pterodactyl replacement
- a billing platform
- a cloud-provider provisioning system
- an AI-only control system
- a full enterprise monitoring platform
- a mobile application
- a marketplace

These may be considered later as integrations or separate products.

---

## 3. Source of Truth

GitHub is the authoritative source for:

- source code
- issues
- pull requests
- releases
- documentation
- project history

AI/build platforms are development workers and must not become the sole source of project state.

---

## 4. Development Model

The project will be built through controlled AI-assisted development.

Workflow:

1. Architecture/specification is defined.
2. A small task is created.
3. An AI coding agent implements the task.
4. Automated tests run.
5. Code is reviewed.
6. Security implications are reviewed.
7. The change is merged only after acceptance.

No large uncontrolled "build the whole product" prompts.

---

## 5. Core Architectural Principles

### 5.1 Vendor Neutrality

CPS Core must not depend on MineDrop, a specific VPS provider, or a specific hosting platform.

### 5.2 Capability Driven

The UI and available management modules should be based on capabilities discovered on the connected server.

### 5.3 Least Privilege

A user receives only the permissions and server/path scope required.

### 5.4 Secure by Default

Operations that can affect a server must pass authentication, authorization, scope validation, and audit requirements.

### 5.5 Portable

The system should be deployable without requiring a single hosting vendor.

### 5.6 Modular

Optional integrations belong in plugins/modules instead of being hardcoded into CPS Core.

---

## 6. Target Architecture

```text
                    CPS WEB UI
                         |
                    HTTPS / WSS
                         |
                 +-------v-------+
                 |    CPS API    |
                 |               |
                 | Auth          |
                 | RBAC          |
                 | Server Mgmt   |
                 | Discovery     |
                 | Audit         |
                 | Plugins       |
                 +-------+-------+
                         |
                  SSH / Agent
                         |
          +--------------+--------------+
          |              |              |
        VPS A          VPS B          VPS C
          |              |              |
        Agent          Agent          Agent
```

Initial connection should support SSH-based discovery.

A lightweight CPS Agent can then be bootstrapped for richer/persistent functionality.

---

## 7. Major Components

### Web UI

Responsible for:

- dashboard
- server selection
- file manager
- terminal
- editor
- monitoring
- service management
- Docker integration
- users
- roles
- audit logs
- settings
- plugins

### CPS API

Responsible for:

- authentication
- authorization
- VPS registry
- SSH coordination
- discovery
- capability management
- agent management
- sessions
- audit logging
- plugins

### CPS Agent

Responsible for server-local operations such as:

- PTY
- filesystem operations
- monitoring
- service operations
- Docker integration
- local discovery

The exact agent protocol remains a technical design task.

### Discovery Engine

Responsible for detecting, where permissions allow:

- OS/distribution/version
- kernel
- architecture
- hostname
- CPU
- RAM
- swap
- storage
- mounts
- network interfaces
- addresses
- package manager
- installed software
- services
- systemd
- Docker/Podman
- common runtimes/databases/web servers
- current-user privileges
- sudo availability

---

## 8. Capability Model

Discovery produces capabilities such as:

```text
filesystem
terminal
monitoring
systemd
docker
nginx
apache
postgresql
mysql
mariadb
redis
node
python
java
php
```

The exact capability list is extensible.

The UI must not assume a capability exists until it is detected and authorized.

---

## 9. VPS Profile

Every managed server should have a normalized profile containing:

- identity
- resources
- network
- software
- services
- permissions
- capabilities

The profile becomes the common data layer used by the UI and integrations.

---

## 10. Agent Bootstrap

Preferred lifecycle:

```text
SSH credentials
      |
      v
Initial SSH connection
      |
      v
Discovery
      |
      v
Capability detection
      |
      v
Optional agent bootstrap
      |
      v
Agent authentication
      |
      v
Persistent management
```

The agent must have its own authenticated identity.

Agent security design must be completed before production use.

---

## 11. File Manager

Initial operations:

- browse
- open
- create
- rename
- copy
- move
- delete
- upload
- download
- search

Later:

- permissions
- ownership
- archive/extract
- preview
- metadata
- hashes

Signature interaction:

**Open in Terminal**

---

## 12. Terminal

The terminal must use a real interactive PTY.

Target path:

```text
Browser
  |
WebSocket
  |
CPS session layer
  |
Agent
  |
PTY
  |
Shell
```

It must not be implemented as a simple command/response textbox.

---

## 13. File ↔ Terminal Integration

Required UX:

```text
File Manager
    |
Open Terminal Here
    |
Terminal starts in that directory
```

And:

```text
Terminal
    |
Open Current Directory
    |
File Manager opens the current path
```

---

## 14. Editor

Initial intended formats:

- YAML
- JSON
- JavaScript
- TypeScript
- Java
- Python
- Bash
- HTML
- CSS
- Markdown
- SQL

Required baseline:

- syntax highlighting
- search
- replace
- save
- unsaved-change protection

---

## 15. Monitoring

Initial metrics:

- CPU
- RAM
- swap
- disk
- network
- load
- uptime
- processes

Advanced historical monitoring is later scope.

---

## 16. Service Management

When available and authorized:

- status
- start
- stop
- restart
- enable
- disable
- logs

Systemd is a capability, not a mandatory dependency.

---

## 17. Docker

Docker is optional.

If detected and authorized:

- containers
- images
- networks
- volumes
- logs
- exec
- start
- stop
- restart

CPS must remain functional without Docker.

---

## 18. RBAC

Hierarchy:

```text
Organization
├── Owner
├── Administrator
├── Developer
├── Operator
└── Custom Roles
```

Permissions are granular.

Examples:

```text
files.read
files.write
files.create
files.rename
files.move
files.delete
files.upload
files.download

terminal.create
terminal.view
terminal.kill

docker.read
docker.start
docker.stop
docker.restart
docker.logs
docker.exec

services.read
services.start
services.stop
services.restart

server.reboot
server.shutdown
```

---

## 19. Permission Scope

Authorization must evaluate:

```text
User
  ↓
Role
  ↓
Permission
  ↓
Server Scope
  ↓
Resource/Path Scope
```

Example:

```text
Developer
files.read
files.write
terminal.create

Scope:
/opt/myapp/
```

This does not automatically grant access to `/etc`, `/root`, or unrelated users' data.

---

## 20. Audit

Important operations must be auditable.

Minimum fields:

- timestamp
- user
- action
- server
- target
- result
- relevant metadata

Examples:

```text
LOGIN
SERVER_CONNECT
FILE_UPLOAD
FILE_DELETE
FILE_MODIFY
TERMINAL_OPEN
SERVICE_RESTART
DOCKER_RESTART
USER_CREATE
ROLE_CHANGE
PERMISSION_CHANGE
```

---

## 21. Authentication

CPS must use an authentication abstraction.

Possible providers may include:

- local accounts
- OAuth
- OIDC

Exact providers are not frozen yet.

SSH credential handling must avoid casual plaintext storage.

---

## 22. Database

Target primary database:

**PostgreSQL**

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

Exact schema is not frozen yet.

Redis is not a required dependency for the initial release.

---

## 23. Plugin Architecture

Core should remain small.

Potential plugins:

```text
Docker
Nginx
Apache
PostgreSQL
MySQL
MariaDB
Redis
MongoDB
PM2
Pterodactyl
Minecraft
```

Community plugins should eventually be possible.

Plugin security and API compatibility must be designed before public plugin support.

---

## 24. AI

AI is optional.

CPS must work completely without AI.

Potential AI functions:

- explain logs
- explain errors
- summarize server health
- identify disk usage
- explain ports/services

AI must not bypass CPS authentication, authorization, scope validation, or auditing.

---

## 25. Deployment

Primary self-hosting target:

**Docker / Docker Compose**

Other deployment modes may include:

- direct Linux installation
- VPS/cloud VM
- compatible PaaS
- frontend/backend split

No hosting provider should be a hard dependency.

---

## 26. Development/Test Environment

VyuhStack VPS:

```text
1 vCPU
1 GB RAM
10 GB storage
```

Purpose:

**CPS development/test/sandbox.**

Not production.

Use it for real integration testing of:

- SSH
- discovery
- agent
- filesystem
- terminal
- permissions
- monitoring
- services
- Docker where available

---

## 27. External Development Tools

### Freebuff
Repository-connected cloud development, rapid implementation, previews.

### Google AI Studio
Reasoning, architecture assistance, debugging, review, documentation, optional AI work.

### Bolt
Rapid UI prototypes and frontend experiments.

### Lovable
Rapid UI/product prototypes.

### Vercel
Frontend/preview/demo deployment where compatible.

### Render
Backend/WebSocket/Docker/public test deployment where compatible.

These are replaceable tools. CPS architecture must not depend on them.

---

## 28. Testing Requirements

### Unit Tests

Required for critical logic such as:

- RBAC
- path validation
- discovery parsing
- authentication
- protocol logic

### Integration Tests

Required for:

- API/database
- API/SSH
- API/agent
- agent/filesystem
- agent/PTY
- discovery

### E2E Tests

Required flow:

```text
Add VPS
→ connect
→ discover
→ view profile
→ browse files
→ edit file
→ open terminal
→ perform authorized action
→ verify audit log
```

### Security Tests

Required before public beta:

- path traversal
- command injection
- shell injection
- RBAC bypass
- privilege escalation
- credential exposure
- session hijacking
- WebSocket authorization
- CSRF
- XSS
- SQL injection
- filesystem escape
- cross-user access
- cross-server access
- agent spoofing
- replay attacks
- archive/symlink attacks

---

## 29. Release Gates

A feature is not complete merely because the UI works.

Before merge:

```text
Implementation
→ Tests
→ Review
→ Security review where applicable
→ Documentation
→ Acceptance
```

Before public beta:

```text
Core tests
+ integration tests
+ E2E tests
+ security audit
+ installation tests
+ documentation
```

---

## 30. Development Phases

### Phase 0
Constitution and technical decisions.

### Phase 1
Core backend:

- database
- authentication
- users
- organizations
- roles
- permissions
- server registry
- audit foundation

### Phase 2
SSH + discovery:

- SSH
- credentials
- system discovery
- hardware discovery
- network discovery
- software discovery
- service discovery
- permission detection
- capability engine
- VPS profile

### Phase 3
Agent.

### Phase 4
File manager.

### Phase 5
Real terminal.

### Phase 6
Editor.

### Phase 7
Monitoring.

### Phase 8
Service management.

### Phase 9
Docker.

### Phase 10
Plugin system.

### Phase 11
Optional AI.

### Phase 12
Multi-VPS/organizations.

### Phase 13
Security audit.

### Phase 14
Open beta.

---

## 31. First Technical Milestone

The first major milestone is NOT the full dashboard.

It is:

```text
Add Server
   ↓
SSH connection
   ↓
Discovery
   ↓
VPS profile
   ↓
Capability list
   ↓
Server visible in CPS
```

Acceptance requires accurate handling of both successful and restricted/failed access.

---

## 32. Current Repository State

Already completed:

```text
GitHub repository                 ✓
README.md                         ✓
LICENSE                           ✓
CONTRIBUTING.md                   ✓
SECURITY.md                       ✓
CODE_OF_CONDUCT.md                ✓
```

Current project state:

```text
Pre-coding / architecture
```

---

## 33. Immediate Next Work

After this Constitution:

1. Technical Decision Record — stack.
2. Technical Decision Record — SSH/agent architecture.
3. Technical Decision Record — discovery/capability system.
4. Technical Decision Record — authentication/RBAC.
5. Technical Decision Record — deployment.
6. Monorepo initialization.
7. CI foundation.
8. Phase 1 implementation.

No full UI generation before these foundations are settled.

---

## 34. Change Control

This Constitution is versioned.

If a frozen decision changes:

1. Record the reason.
2. Update the relevant technical decision.
3. Update the Constitution version if the change affects project principles or architecture.
4. Ensure implementation and documentation remain consistent.

---

## 35. Current Frozen Summary

```text
Open source                       YES
Universal Linux VPS focus        YES
MineDrop-specific core           NO
GitHub source of truth            YES
AI coding workers                 YES
Freebuff                          YES
Docker-first self-hosting        YES
SSH initial connection            YES
Automatic discovery               YES
Capability-driven UI              YES
Optional agent                    YES
Real PTY terminal                 YES
File manager                      YES
File ↔ terminal integration      YES
Hierarchical RBAC                 YES
Scoped permissions                YES
Audit logs                        YES
Plugin architecture               YES
Optional AI                       YES
MineDrop as first deployment     YES
Security as release gate          YES
Redis mandatory                   NO
AI mandatory                      NO
Docker mandatory                  NO
```

## END — CPS PROJECT CONSTITUTION v0.1
