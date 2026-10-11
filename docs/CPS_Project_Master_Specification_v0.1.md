# CPS — Project Master Specification & Development Context
## Version 0.1 — Foundation / Pre-Coding

> This document is the working source of project context for the CPS open-source VPS management platform.  
> GitHub remains the authoritative source for code. This document is a planning/context artifact.

---

# 1. Project Identity

## Temporary Project Name

**CPS**

The permanent public name has not yet been finalized.

## Product Category

Universal, open-source, self-hostable Linux VPS/server control and management platform.

## Core Idea

CPS should provide a web-based management experience for Linux VPSs/servers, similar in convenience to a desktop file manager combined with an interactive SSH terminal and server-management tools.

The initial connection should be simple:

1. User adds a VPS.
2. User provides SSH connection credentials.
3. CPS connects to the VPS.
4. CPS discovers the server environment.
5. CPS builds a normalized VPS profile.
6. CPS detects available capabilities.
7. The UI adapts to the discovered capabilities.
8. CPS can optionally bootstrap/install a lightweight agent for richer persistent functionality.

The project must be designed as a general-purpose open-source product, not as a MineDrop-only panel.

MineDrop is the first serious deployment and testing environment.

---

# 2. Product Vision

## Vision

Make Linux VPS/server management accessible through a clean web interface without removing the power and flexibility of SSH.

The user should be able to:

- Browse and manage files.
- Open files in an editor.
- Open a terminal directly in the current directory.
- Run a real interactive shell.
- Monitor server resources.
- Inspect and manage services.
- Manage Docker when available.
- Manage multiple VPSs.
- Create users and roles.
- Apply scoped permissions.
- Review audit logs.
- Install/use plugins for detected software.
- Optionally use an AI assistant.
- Self-host the entire platform.

## Core Product Promise

> Connect a Linux server through SSH, let CPS discover it, and manage it through a unified web interface.

---

# 3. Important Project Principles

## 3.1 Open Source First

CPS is open source from the beginning.

Do not build a MineDrop-specific product and attempt to generalize it later.

Instead:

- CPS Core is vendor-neutral.
- MineDrop-specific functionality is optional integration/plugin functionality.
- Other users and organizations should be able to deploy CPS independently.

## 3.2 GitHub Is the Source of Truth

All development must ultimately live in the GitHub repository.

External AI/build tools are workers, not the authoritative project repository.

Tools may be used for:

- UI generation
- Coding
- Prototyping
- Debugging
- Testing
- Review

But generated work must be reviewed and brought back into the GitHub workflow.

The project must remain portable if any one AI platform disappears.

## 3.3 AI Agents Are Workers

Development model:

```text
Architecture
    ↓
Specification
    ↓
Small implementation task
    ↓
AI coding/build agent
    ↓
Tests
    ↓
Review
    ↓
Security review
    ↓
Merge
```

Do not give an AI agent an unrestricted instruction such as "build the entire VPS panel" and blindly accept the result.

Tasks should be small, explicit, testable, and reviewable.

## 3.4 Security Is a Release Gate

CPS can execute commands and manipulate files on servers. Security is therefore a core architectural requirement, not a later cosmetic feature.

Any release with:

- arbitrary command execution vulnerabilities
- filesystem escape
- RBAC bypass
- credential leakage
- agent impersonation
- session hijacking
- privilege escalation

must be blocked until resolved.

## 3.5 Portable Architecture

Do not design CPS around one hosting vendor.

The architecture must work with:

- self-hosted VPS
- Docker
- local development
- PaaS deployments where technically compatible
- split frontend/backend deployments
- multiple connected VPSs

Do not promise universal compatibility without testing.

---

# 4. Initial Resources / Development Tools

Current resources available for the project include:

## GitHub

Primary source repository and version control.

## Freebuff

Use for:

- cloud development
- rapid implementation
- frontend experimentation
- repository-connected development
- live previews where useful

Freebuff is not the source of truth.

## Google AI Studio / Gemini

Use for:

- architecture assistance
- debugging
- code analysis
- documentation
- implementation assistance
- optional future AI integration research

AI must remain optional in the CPS product itself.

## Bolt

Use primarily for:

- UI prototypes
- layout experiments
- rapid frontend concepts

Useful code should be moved into the real GitHub project.

## Lovable

Use primarily for:

- UI/product prototypes
- frontend experimentation
- rapid interface concepts

Useful output should be reviewed and integrated into the GitHub project.

## Vercel

Potentially useful for:

- frontend deployments
- preview deployments
- demos
- compatible application components

Do not make CPS core dependent on Vercel-specific infrastructure.

## Render

Potentially useful for:

- backend testing
- WebSocket testing
- Docker deployment testing
- public demos
- temporary environments

Do not make CPS architecture dependent on Render-specific behavior.

## VyuhStack VPS

Current development/test resource:

- 1 vCPU
- 1 GB RAM
- 10 GB storage

Purpose:

**CPS development/test/sandbox environment.**

It is not production infrastructure.

Use it to test:

- SSH
- discovery
- agent
- filesystem operations
- terminal
- permissions
- monitoring
- services
- Docker if available
- deployment

---

# 5. Repository Foundation

The GitHub repository has already been created.

The initial five files have been created:

```text
README.md
LICENSE
CONTRIBUTING.md
SECURITY.md
CODE_OF_CONDUCT.md
```

Do not recreate these unnecessarily.

The next task is to establish the Project Constitution and then begin implementation in controlled phases.

---

# 6. High-Level Architecture

Target architecture:

```text
                         CPS WEB UI
                             │
                         HTTPS/WSS
                             │
                    ┌────────▼────────┐
                    │     CPS API     │
                    │                 │
                    │ Authentication  │
                    │ RBAC            │
                    │ VPS Manager     │
                    │ Discovery       │
                    │ Audit           │
                    │ Plugins         │
                    └────────┬────────┘
                             │
                    SSH / Agent Protocol
                             │
             ┌───────────────┼────────────────┐
             │               │                │
             ▼               ▼                ▼
          VPS #1          VPS #2           VPS #3
             │               │                │
           Agent           Agent            Agent
             │               │                │
        Files/PTY/etc.  Files/PTY/etc.   Files/PTY/etc.
```

The agent should not necessarily be required for the first SSH discovery.

Preferred lifecycle:

```text
Initial connection
    ↓
SSH
    ↓
Discovery
    ↓
Capability detection
    ↓
Optional agent bootstrap
    ↓
Persistent agent connection
```

---

# 7. Core Components

## 7.1 Web UI

Responsible for:

- dashboard
- server selection
- file manager
- terminal
- editor
- monitoring
- services
- Docker
- users
- roles
- audit logs
- settings
- plugin UI

## 7.2 CPS API / Control Plane

Responsible for:

- authentication
- authorization
- VPS registry
- SSH coordination
- discovery
- capability management
- agent management
- session management
- audit logging
- plugin management
- API endpoints

## 7.3 CPS Agent

A lightweight component running on managed Linux VPSs.

Potential responsibilities:

- PTY/terminal
- filesystem operations
- monitoring
- service control
- Docker integration
- local capability discovery
- secure communication with CPS

The agent should be lightweight and appropriate for low-resource servers.

## 7.4 Discovery Engine

Responsible for discovering:

- operating system
- distribution/version
- kernel
- architecture
- hostname
- uptime
- CPU
- RAM
- swap
- storage
- mounts
- network interfaces
- IP addresses
- listening ports where permitted
- package manager
- installed software
- running services
- systemd
- Docker/Podman
- common databases/web servers/runtimes
- current-user privileges
- sudo availability
- filesystem access

Discovery must gracefully handle limited privileges.

---

# 8. VPS Onboarding Flow

Target user experience:

```text
Add Server
    ↓
Name
Host/IP
SSH Port
Username
Authentication Method
Credential
    ↓
Test Connection
    ↓
SSH Connection
    ↓
Discovery
    ↓
Permission/Privilege Detection
    ↓
Capability Detection
    ↓
VPS Profile
    ↓
Optional Agent Bootstrap
    ↓
Dashboard
```

Example:

```text
Connecting...
✓ SSH connection established

Detecting operating system...
✓ Ubuntu 24.04

Detecting architecture...
✓ x86_64

Detecting resources...
✓ 8 CPU
✓ 16 GB RAM
✓ 200 GB storage

Detecting services...
✓ Docker
✓ Nginx
✓ PostgreSQL

Checking permissions...
✓ sudo
✓ required filesystem access

Building server profile...
✓ Complete
```

---

# 9. Discovery and Capability Architecture

CPS must not assume a fixed software stack.

For example, one server may have:

```text
Ubuntu
Docker
Nginx
PostgreSQL
Redis
```

Another may have:

```text
Debian
Apache
MariaDB
PHP
```

The UI should adapt based on detected capabilities.

## Example capability set

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

Capabilities should be represented as structured data rather than scattered hardcoded UI checks.

---

# 10. VPS Profile

Each managed server should have a normalized profile.

Conceptual structure:

```text
VPS Profile
├── Identity
│   ├── hostname
│   ├── OS
│   ├── version
│   ├── kernel
│   └── architecture
│
├── Resources
│   ├── CPU
│   ├── RAM
│   ├── swap
│   ├── storage
│   └── load
│
├── Network
│   ├── interfaces
│   ├── addresses
│   └── routes
│
├── Software
│   ├── Docker
│   ├── Node
│   ├── Python
│   ├── Java
│   └── other detected software
│
├── Services
│   ├── systemd
│   ├── Nginx
│   ├── Redis
│   ├── PostgreSQL
│   └── other detected services
│
├── Permissions
│   ├── current user
│   ├── sudo
│   └── accessible paths
│
└── Capabilities
    ├── filesystem
    ├── terminal
    ├── monitoring
    └── detected integrations
```

---

# 11. File Manager

This is one of the primary CPS features.

## MVP operations

- browse directories
- open files/directories
- create directory
- create file
- rename
- copy
- move
- delete
- upload
- download
- search
- sorting
- hidden files where permitted

## Advanced operations

- permissions
- ownership
- chmod
- chown
- archive
- extract
- file preview
- file metadata
- hashes
- editor integration

## Context actions

For a file/folder:

```text
Open
Open in Editor
Open in Terminal
Download
Rename
Copy
Move
Permissions
Delete
```

---

# 12. Filesystem Safety

CPS must not blindly scan/download/read every file on a server.

Discovery should identify filesystem structure and metadata where possible.

Large or sensitive directories should be inspected lazily.

Examples:

```text
/var/lib/docker
/root
/home/*
/etc
```

must respect permissions.

Never assume the user has unrestricted root access.

---

# 13. Terminal

The terminal must be a real interactive PTY, not a simple request/response command box.

Target architecture:

```text
Browser
   │
 WebSocket / secure WebSocket
   │
CPS API / session layer
   │
CPS Agent
   │
 PTY
   │
Shell
```

It should support interactive commands/applications such as:

```text
bash
sh
htop
top
tail -f
docker logs -f
nano
vim
```

subject to user permissions.

---

# 14. File Manager ↔ Terminal Integration

Signature feature:

From:

```text
/opt/myapp/
```

select:

```text
Open Terminal Here
```

Terminal starts in:

```bash
cd /opt/myapp
```

Conversely, the terminal should be able to expose:

```text
Open Current Directory
```

which sends the current path to the file manager.

---

# 15. File Editor

The editor should eventually support common formats:

```text
YAML
JSON
JavaScript
TypeScript
Java
Python
Bash
HTML
CSS
Markdown
SQL
```

Expected features:

- syntax highlighting
- search
- replace
- save
- unsaved-change detection
- basic error handling

The editor must not silently overwrite files without appropriate safeguards.

---

# 16. Monitoring

Initial monitoring:

```text
CPU
RAM
Swap
Disk
Network
Load
Uptime
Processes
```

Historical metrics and advanced graphs can be added later.

Do not over-engineer monitoring before the core server-management workflow is stable.

---

# 17. Service Management

When systemd/service-control capabilities are available:

```text
Start
Stop
Restart
Enable
Disable
Status
Logs
```

The UI should clearly show when a service-management capability is unavailable.

---

# 18. Docker Management

Docker support should be capability-based.

If Docker exists:

```text
Containers
Images
Networks
Volumes
Logs
Exec
Start
Stop
Restart
```

If Docker does not exist:

```text
Docker not detected
```

Do not make Docker a required dependency for CPS.

---

# 19. Hierarchical RBAC

CPS must use a proper permission model rather than only simple admin/user roles.

Conceptual hierarchy:

```text
Organization
├── Owner
├── Administrator
├── Developer
├── Operator
└── Custom Roles
```

Roles are collections of permissions.

Example permissions:

```text
files.read
files.write
files.create
files.rename
files.move
files.delete
files.upload
files.download
files.chmod
files.chown

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

users.read
users.create
users.edit
users.delete

roles.read
roles.create
roles.edit
roles.delete
```

---

# 20. Scoped Permissions

A role should not automatically grant access to everything.

Permission model:

```text
WHO
 ↓
ROLE
 ↓
PERMISSION
 ↓
SCOPE
```

Example:

```text
Developer
├── files.read
├── files.write
├── terminal.create
└── Scope: /opt/myapp/
```

The developer should not automatically receive access to:

```text
/etc
/root
other users' home directories
```

unless explicitly authorized.

This scope model is essential to the product.

---

# 21. Audit Logging

Important operations must be auditable.

Example:

```text
Timestamp
User
Action
Server
Target
Result
Metadata
```

Examples:

```text
LOGIN
SERVER_CONNECT
FILE_UPLOAD
FILE_DELETE
FILE_MODIFY
TERMINAL_OPEN
TERMINAL_CLOSE
SERVICE_RESTART
DOCKER_RESTART
USER_CREATE
ROLE_CHANGE
PERMISSION_CHANGE
```

Sensitive actions should have sufficient context for later investigation.

---

# 22. Authentication

CPS should be designed around an authentication abstraction.

Potential providers:

- local accounts
- OAuth
- OIDC
- other supported identity providers

Exact providers should be finalized based on practical implementation and security requirements.

Managed VPS SSH authentication should support appropriate methods such as:

- password
- SSH private key
- SSH agent where technically supported
- other secure SSH authentication methods

SSH credentials must not be casually stored in plaintext.

---

# 23. Agent Security

The agent must authenticate itself to CPS.

Do not rely only on:

```text
IP address
hostname
```

Use cryptographic identity/registration.

Potential lifecycle:

```text
SSH bootstrap
 ↓
Generate/register agent identity
 ↓
CPS verifies identity
 ↓
Secure persistent connection
```

The exact protocol is to be designed and security-reviewed before implementation.

---

# 24. Plugin System

Core CPS should remain small.

Conceptual structure:

```text
CPS Core
├── Authentication
├── RBAC
├── Server connections
├── Discovery
├── Files
├── Terminal
├── Monitoring
├── Audit
└── Plugin system
```

Optional integrations:

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

Community developers should eventually be able to create plugins without changing CPS Core.

---

# 25. AI Assistant

AI is optional.

CPS must function normally with AI disabled.

Potential future AI functions:

```text
Why is my disk usage high?
What is using port 8080?
Explain this Docker error.
Why is nginx failing?
Summarize server health.
Explain this log.
```

The AI should preferably receive structured/limited information rather than unrestricted raw server access.

AI should not become a hidden privileged command executor.

Any AI action that can modify a server must go through the same authorization/security layer as normal users.

---

# 26. Database

Primary database target:

**PostgreSQL**

Conceptual entities:

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

The exact schema must be designed and reviewed before implementation.

Do not add Redis until a real requirement exists.

---

# 27. Deployment Philosophy

CPS should be deployable in multiple ways.

## Primary

Docker / Docker Compose.

## Other targets

Potentially:

- direct Linux/VPS installation
- cloud VM
- compatible PaaS
- frontend/backend split deployment
- local development

Vercel and Render can be used for suitable parts/testing, but CPS must not become vendor-locked.

---

# 28. Deployment Architecture

## All-in-one

```text
VPS
├── CPS
├── PostgreSQL
└── optional Agent
```

## CPS on one server, managed VPSs elsewhere

```text
CPS
├── VPS A Agent
├── VPS B Agent
└── VPS C Agent
```

## PaaS-style

```text
Frontend
   ↓
CPS API
   ↓
Persistent external database
   ↓
Managed VPS agents
```

The application must not depend on ephemeral local filesystem storage for persistent state.

---

# 29. Development Phases

## Phase 0 — Constitution

Finalize:

- project identity
- goals/non-goals
- architecture
- stack
- deployment model
- agent model
- discovery model
- capability model
- RBAC
- security model
- plugin model
- testing strategy

**Current phase.**

## Phase 1 — Core Backend

Implement:

- database foundation
- authentication
- users
- organizations
- roles
- permissions
- server registry
- audit foundation

## Phase 2 — SSH + Discovery

Implement:

- SSH connection
- credential handling
- OS discovery
- hardware discovery
- network discovery
- software discovery
- service discovery
- permission detection
- capability engine
- VPS profile

**First major technical milestone.**

## Phase 3 — Agent

Implement:

- agent bootstrap
- agent identity
- authentication
- heartbeat
- command framework
- filesystem API
- PTY API
- monitoring API

## Phase 4 — File Manager

Implement the basic file-management workflow.

## Phase 5 — Terminal

Implement real PTY + WebSocket terminal.

## Phase 6 — Editor

Implement text/code editor.

## Phase 7 — Monitoring

Implement live server metrics.

## Phase 8 — Services

Implement service/systemd management.

## Phase 9 — Docker

Implement Docker capability/plugin.

## Phase 10 — Plugin System

Formalize plugin APIs and documentation.

## Phase 11 — AI

Optional assistant layer.

## Phase 12 — Multi-VPS / Organizations

Complete multi-server and organization workflows.

## Phase 13 — Security Audit

Red-team the entire application.

## Phase 14 — Open Beta

Prepare:

- documentation
- Docker deployment
- installation scripts
- demo
- contribution guide
- security policy
- issue templates
- release process

---

# 30. Security Testing Checklist

Before public beta, test at minimum:

```text
Path traversal
Command injection
Shell injection
RBAC bypass
Privilege escalation
Credential leakage
SSH key exposure
Session hijacking
WebSocket authorization
CSRF
XSS
SQL injection
Filesystem escape
Cross-user access
Cross-server access
Agent spoofing
Agent authentication bypass
Replay attacks
API abuse
Rate limiting
File upload abuse
Malicious archive extraction
Symlink attacks
```

Particular test:

A developer scoped to:

```text
/opt/myapp/
```

must not escape to:

```text
/etc
/root
/home/other-user
```

through:

- `../`
- symlinks
- archive extraction
- editor operations
- terminal commands
- API manipulation

---

# 31. Testing Strategy

Testing should exist at multiple levels.

## Unit

For:

- permissions
- discovery parsers
- protocol logic
- path validation
- authentication
- utility functions

## Integration

For:

- API ↔ database
- API ↔ SSH
- API ↔ Agent
- Agent ↔ filesystem
- Agent ↔ PTY
- discovery

## End-to-End

For:

```text
Add VPS
→ connect
→ discover
→ dashboard
→ browse files
→ edit
→ terminal
→ service action
→ audit log
```

## Security

Dedicated security tests must run before release.

---

# 32. AI Development Team

Suggested worker roles:

## Architect Agent

Responsible for:

- architecture drafts
- API contracts
- database proposals
- protocol proposals

## Backend Agent

Responsible for:

- API
- authentication
- RBAC
- database
- discovery

## Agent/System Agent

Responsible for:

- Go agent
- SSH
- PTY
- filesystem
- Linux integration

## Frontend Agent

Responsible for:

- UI
- dashboard
- file manager
- terminal UI
- responsive design

## QA Agent

Responsible for:

- unit tests
- integration tests
- E2E tests
- regression tests

## Security Agent

Responsible for:

- adversarial testing
- security review
- privilege boundary tests
- RBAC bypass attempts

Humans remain the architecture/review/release authority.

---

# 33. Tool Usage Strategy

## Freebuff

Best suited to:

- connected repository work
- cloud development
- rapid implementation
- frontend prototypes
- live preview

## Google AI Studio

Best suited to:

- reasoning
- architecture
- debugging
- code review
- documentation
- AI-related work

## Bolt

Best suited to:

- fast UI generation
- interface experiments

## Lovable

Best suited to:

- rapid product/frontend prototypes

## Vercel

Best suited to:

- frontend/preview/demo deployment where compatible

## Render

Best suited to:

- backend
- WebSocket testing
- Docker
- public test environments

## VyuhStack

Best suited to:

- real VPS integration testing
- SSH
- discovery
- agent
- filesystem
- terminal
- permissions
- deployment tests

---

# 34. What NOT To Do Yet

Do not begin with:

- fancy animations
- AI assistant
- Docker UI
- huge monitoring system
- marketplace
- billing
- mobile app
- MineDrop-specific features
- dozens of plugins

Do not hardcode:

- MineDrop domains
- MineDrop users
- MineDrop infrastructure
- MineDrop server names
- MineDrop-specific permissions

The core must remain general.

---

# 35. Immediate Next Steps

The repository already exists with:

```text
README.md
LICENSE
CONTRIBUTING.md
SECURITY.md
CODE_OF_CONDUCT.md
```

Next:

### Step A
Finalize this Constitution.

### Step B
Create a project architecture document in the repository.

### Step C
Create a technical decision record for:

- stack
- agent language
- communication protocol
- authentication
- database
- deployment

### Step D
Create the initial monorepo structure.

### Step E
Create CI.

### Step F
Implement Phase 1 only.

No large application generation yet.

---

# 36. First Major Acceptance Test

The first meaningful CPS milestone should be:

```text
User
 ↓
Add Server
 ↓
Enter SSH credentials
 ↓
Connect
 ↓
Discover
 ↓
Generate VPS Profile
 ↓
Display:
  OS
  CPU
  RAM
  Disk
  Network
  Software
  Services
  Permissions
  Capabilities
 ↓
Server appears in CPS dashboard
```

If this works reliably, the project has a solid foundation.

---

# 37. Long-Term Product Structure

Eventually:

```text
                         CPS
                          │
        ┌─────────────────┼──────────────────┐
        │                 │                  │
      Core              Plugins            AI
        │                 │                  │
   ┌────┼────┐       ┌────┼────┐       Optional
   │    │    │       │    │    │
 Files PTY RBAC    Docker Nginx DB
   │
Monitoring
   │
Services
   │
Audit
```

CPS should be useful without any plugin or AI.

---

# 38. MineDrop's Role

MineDrop is:

- first serious deployment
- development/testing environment
- real-world validation environment
- possible source of integrations/plugins

MineDrop must not define CPS Core.

Potential future MineDrop integration:

```text
CPS
└── MineDrop Plugin
    ├── MineDrop infrastructure
    ├── Minecraft services
    └── MineDrop-specific operations
```

---

# 39. Definition of Success

CPS is successful when an independent user can:

1. Deploy CPS themselves.
2. Connect a Linux VPS through SSH.
3. Let CPS discover the VPS.
4. See an accurate server profile.
5. Browse/manage permitted files.
6. Open a real terminal.
7. Open a terminal in the current file-manager directory.
8. Monitor resources.
9. Manage supported services.
10. Manage Docker if installed.
11. Create users/roles.
12. Apply scoped permissions.
13. Review audit logs.
14. Add additional VPSs.
15. Extend functionality through plugins.
16. Run the platform without MineDrop.
17. Run it without AI.
18. Run it without being locked to one hosting provider.

---

# 40. Current Project State

```text
Project: CPS
Status: Pre-coding / Architecture

GitHub repository: CREATED
Foundation files: CREATED

README.md           ✓
LICENSE             ✓
CONTRIBUTING.md     ✓
SECURITY.md         ✓
CODE_OF_CONDUCT.md  ✓

VyuhStack test VPS: AVAILABLE

Development resources:
- Freebuff
- Google AI Studio
- Bolt
- Lovable
- Vercel
- Render
- VyuhStack

Primary source of truth:
GitHub

Primary deployment target:
Docker / self-hosted

Current milestone:
Project Constitution → Architecture → Technical Decisions → Monorepo → Phase 1

Next implementation milestone:
SSH + VPS Discovery + VPS Profile
```

---

# 41. Working Rule for This Project Chat

This project discussion should remain:

- practical
- technical
- architecture-focused
- implementation-focused
- security-conscious
- concise where possible

Avoid unsupported assumptions.

When a current external tool/platform capability matters, verify it before relying on it.

Do not invent APIs, deployment capabilities, pricing, limits, or product features.

When uncertain, explicitly mark the item as:

```text
Needs verification
```

rather than guessing.

---

# 42. Current Decision Summary

```text
OPEN SOURCE                  ✓
UNIVERSAL VPS PRODUCT        ✓
MINEdrop-SPECIFIC CORE       ✗
GITHUB SOURCE OF TRUTH       ✓
AI CODING WORKERS            ✓
FREEBUFF INCLUDED            ✓
DOCKER-FIRST                 ✓
SELF-HOSTABLE                ✓
SSH INITIAL CONNECTION       ✓
AUTOMATIC DISCOVERY          ✓
CAPABILITY-DRIVEN UI         ✓
OPTIONAL AGENT               ✓
REAL PTY TERMINAL            ✓
FILE MANAGER                 ✓
FILE ↔ TERMINAL              ✓
HIERARCHICAL RBAC            ✓
SCOPED PERMISSIONS           ✓
AUDIT LOGS                   ✓
PLUGIN SYSTEM                ✓
OPTIONAL AI                  ✓
MINEdrop AS FIRST DEPLOYMENT ✓
SECURITY AS RELEASE GATE     ✓
```

---

## END OF CPS MASTER SPECIFICATION v0.1
