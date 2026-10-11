# CPS Architecture Decision Record 002
## SSH Onboarding and Agent Architecture

- **Status:** Proposed for review
- **Decision date:** 2026-10-11
- **Related documents:** CPS Project Master Specification v0.1; CPS Project Constitution v0.1; ADR-001 — Technology Stack v0.1
- **Scope:** Initial SSH access, discovery, optional agent installation, persistent agent communication, and secure operation execution.

## 1. Context

CPS must manage Linux VPS instances across different providers and deployment environments. A VPS may not have the CPS agent installed, may be behind NAT or a restrictive firewall, and may use a different Linux distribution or service manager.

CPS therefore needs an initial connection path that does not assume an agent exists, plus a long-term path that does not require exposing a new inbound management port on every managed VPS. The design must support least privilege, explicit authorization, auditable operations, safe reconnects, and clear failure behavior.

## 2. Decisions proposed

### 2.1 SSH is the initial onboarding path

The user supplies the host/IP, SSH port, username, authentication method, and authorization to connect and perform discovery. Supported authentication methods and their exact implementation details must be confirmed during implementation.

Credentials must never appear in logs or be stored in plaintext. The credential storage, access, rotation, and deletion lifecycle must be documented before release.

### 2.2 Discovery happens before agent installation

After successful SSH authentication, CPS performs bounded, metadata-oriented discovery and builds a normalized VPS profile. It must not recursively read or download the server's files.

Discovery must:
- Use bounded commands or scripts with timeouts and output limits.
- Gracefully handle missing commands, restricted users, and partial permissions.
- Avoid assuming `systemd`, Docker, `sudo`, or a specific Linux distribution exists.
- Record detected, unavailable, failed, and unknown capabilities distinctly.
- Avoid collecting secrets, file contents, or unrelated application data.
- Support cancellation where practical.

CPS may offer limited SSH-only functionality if agent installation is declined or unsupported. The exact SSH-only feature matrix remains to be specified.

### 2.3 The agent is a separate Go binary

Consistent with ADR-001, the VPS agent is a separate native Go binary, not embedded in the frontend or API process. It exposes a narrow, versioned operation interface and must not become an unaudited generic root-command execution service.

Agent installation or upgrade requires an explicit user-authorized action. Before execution, CPS should show the intended action and relevant privilege implications.

### 2.4 Prefer an authenticated outbound agent connection

After authorized installation and registration, the agent initiates an authenticated outbound connection to CPS. Normal operation should not require a new inbound listening port on each managed VPS.

Authenticated TLS is required. WebSocket over TLS is the proposed initial transport because the selected stack includes HTTP and WebSocket, but the protocol libraries and compatibility of each supported deployment target must be verified before this decision is frozen.

The agent reconnects after transient failures using bounded exponential backoff with jitter. If heartbeats stop, CPS marks the agent disconnected or stale and must not present stale data as current.

### 2.5 Use a one-time bootstrap token and durable agent identity

Proposed enrollment lifecycle:
1. An authorized user requests agent installation.
2. CPS creates a high-entropy, short-lived, single-use bootstrap token bound to the intended server record and enrollment attempt.
3. The agent is installed and started through the authorized SSH bootstrap process.
4. The agent uses the token to enroll and establish a durable cryptographic identity.
5. The bootstrap token is invalidated after enrollment or expiry.
6. CPS records the enrollment and binds the identity to the intended server.
7. The durable identity can be revoked and replaced through an audited procedure.

Bootstrap tokens must be excluded from logs. Hostname, IP address, or a user-supplied server name is not sufficient proof of agent identity. The exact identity format, storage, rotation, and trust model remain open pending security review.

### 2.6 Authorize every operation

A valid agent connection does not grant permission to perform every action. CPS authorizes each operation using the authenticated user, role, permission, server scope, and resource/path scope.

The agent validates request structure and enforces its own safety boundaries. Control-plane authorization remains authoritative; agent-side checks are defense in depth. Privileged operations must be explicit, limited to necessary permissions, and auditable. The entire control plane must not run as root on managed servers.

### 2.7 Keep terminal and filesystem operations logically separate

Interactive terminal flow:

`Browser terminal → authenticated CPS session → agent session → PTY/shell`

The terminal must be a real interactive PTY, not a command textbox. Session creation, resizing, input/output, termination, and reconnect behavior must be defined and tested.

File operation flow:

`Browser → CPS authorization → typed agent file operation → filesystem`

File operations must not concatenate user-controlled paths into shell commands. They must normalize and validate paths, enforce resource/path scope, and define safe handling for traversal, symlinks, special files, and filesystem boundaries.

Terminal access is inherently powerful. CPS must explain that shell permissions depend on the configured OS/agent privilege model; UI path restrictions do not sandbox arbitrary shell commands.

### 2.8 Version the protocol

The agent protocol must support explicit version negotiation. Incompatible versions must produce a clear error rather than silently attempting unsupported operations. Agent upgrades must be authenticated, explicit, auditable, and recoverable where practical. An upgrade must not silently broaden permissions or change the configured privilege model.

### 2.9 Audit security-relevant events

Audit at least:
- SSH onboarding attempts and results
- Discovery start, completion, and failure
- Agent bootstrap, enrollment, reconnect, version mismatch, revocation, and upgrade
- Terminal session creation and termination
- Sensitive file and service operations
- Authorization denials and policy-relevant failures

Logs must not contain passwords, private keys, bootstrap tokens, session secrets, or unnecessary file contents.

If CPS or the network is unavailable, the agent must not accept unauthenticated commands or invent authorization decisions. The initial implementation should not queue privileged operations for later execution.

## 3. Security requirements and release gates

Implementation tests or documented controls must cover:
- Bootstrap-token expiry, single-use enforcement, replay, and server binding
- Agent identity verification, revocation, and impersonation attempts
- TLS verification and secret leakage
- Cross-tenant and cross-server agent association
- Authorization on every operation and session/WebSocket message
- Reconnect behavior after revocation or credential rotation
- Path traversal, symlink escape, special files, and archive extraction if supported
- Terminal isolation, session hijacking, and unauthorized attachment
- Command injection through discovery inputs or file paths
- Discovery and request time, output, and resource limits
- Malformed, oversized, duplicated, or out-of-order protocol messages
- Audit-log redaction

These are release gates, not optional enhancements.

## 4. Consequences

### Benefits
- Onboarding works before the agent is installed.
- A new inbound agent port is not required on each VPS for normal operation.
- Discovery and installation remain separate, explicit steps.
- The design can support VPS instances behind NAT and restrictive firewalls, subject to the control-plane deployment's transport capabilities.
- Operations can be versioned, authorized, and audited individually.

### Costs and trade-offs
- CPS must securely handle SSH credentials and bootstrap execution.
- Persistent connections introduce reconnect, identity, and protocol-versioning complexity.
- Agent privilege boundaries and terminal semantics require careful security design.
- Some capabilities may be unavailable in SSH-only mode or where agent installation is unsupported or not authorized.
- PaaS/WebSocket compatibility, proxy timeouts, and long-lived connection limits must be verified for each supported deployment target.

## 5. Explicit non-decisions

This ADR does not yet freeze:
- The exact SSH client library or credential-vault implementation
- Agent identity format, certificate authority, or key-rotation mechanism
- Message schema or serialization format
- The initial set of supported SSH authentication methods
- The exact OS privilege model for the agent
- The SSH-only feature matrix
- Transport behavior on every PaaS or reverse-proxy platform

These require focused implementation research and security review. Do not treat them as already decided.

## 6. Acceptance checklist

- [ ] Review this ADR against the Master Specification and Constitution.
- [ ] Document SSH credential storage, access, rotation, and deletion.
- [ ] Define agent identity, enrollment, revocation, and rotation.
- [ ] Define the minimum agent privilege model and privileged-operation boundary.
- [ ] Define a versioned request/response and event protocol.
- [ ] Verify outbound WebSocket/TLS behavior for intended deployment targets.
- [ ] Define SSH-only capabilities and limitations.
- [ ] Create threat-model notes and tests for the security requirements above.
- [ ] Approve the ADR in the repository.

## 7. Implementation sequence

1. Define SSH credential and session lifecycle.
2. Implement SSH connectivity checks with strict timeouts and redacted errors.
3. Implement bounded discovery and the normalized VPS profile.
4. Add explicit, auditable agent bootstrap.
5. Implement single-use enrollment and durable agent identity.
6. Implement authenticated outbound transport, heartbeat, reconnect, and revocation.
7. Add typed operation authorization and audit events.
8. Implement filesystem operations and PTY sessions only after corresponding security tests exist.

**Current status: PROPOSED FOR REVIEW — not yet frozen.**
