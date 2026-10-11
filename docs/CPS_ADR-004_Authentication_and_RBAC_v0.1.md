# CPS Architecture Decision Record 004
## Authentication, Authorization, and Role-Based Access Control

- **Status:** Proposed for review
- **Decision date:** 2026-10-11
- **Decision owners:** CPS maintainers
- **Related documents:** CPS Project Master Specification v0.1; CPS Project Constitution v0.1; ADR-001 — Technology Stack v0.1; ADR-002 — SSH Onboarding and Agent Architecture; ADR-003 — Discovery and Capability Detection
- **Scope:** How CPS authenticates dashboard users, models organizations and roles, authorizes actions, scopes access to managed servers and resources, and records security-relevant events.

---

## 1. Context

CPS manages remote Linux VPS instances and can expose powerful operations such as terminal sessions, file modification, service control, and Docker management. Authentication alone is not enough: each action must be authorized for the authenticated user, the target server, and the requested resource.

CPS is intended to support multiple users and, eventually, organizations managing multiple servers. Its access model must support least privilege, scoped permissions, and reliable isolation between organizations and servers. Frontend controls must never be treated as the security boundary.

## 2. Decision

### 2.1 Separate authentication from authorization

- **Authentication** establishes who the user is.
- **Authorization** determines what that user may do, where, and under which conditions.
- **Capability detection** reports what the target host can technically support; it does not grant user permission.
- **Audit logging** records security-relevant actions and outcomes.

These concerns must remain separate in code and data models.

### 2.2 Use a user, organization, membership, role, permission, and scope model

The conceptual authorization chain is:

`WHO → ORGANIZATION/MEMBERSHIP → ROLE → PERMISSION → SERVER SCOPE → RESOURCE/PATH SCOPE`

The initial model should support:
- Users
- Organizations or workspaces
- Organization memberships
- Roles assigned through memberships
- Stable permission identifiers
- Server-level scope
- Resource or path scope where applicable
- Audit events

A user may belong to more than one organization. A request must identify the active organization context, and the backend must verify that the user is a member and that the target server belongs to that organization before authorizing the action.

The exact database schema and organization-creation workflow remain implementation details and must be reviewed before they are frozen.

### 2.3 Use deny-by-default authorization

If a permission is absent, ambiguous, malformed, or cannot be evaluated, the request is denied. A missing capability, stale capability, disconnected agent, or unsupported operation must not be interpreted as permission to proceed.

Every protected backend operation must evaluate authorization server-side. Hiding a button or disabling a UI module is not an authorization control.

Authorization must be checked at the time of the operation, not only when a page loads or a WebSocket connection is first established. Long-lived sessions must re-check relevant permissions for sensitive messages and operations, and must be closed or restricted when access is revoked.

### 2.4 Use stable, granular permission identifiers

Permissions should be explicit and machine-readable, grouped by domain. Initial candidate examples include:

- `servers.read`
- `servers.create`
- `servers.update`
- `servers.delete`
- `servers.connect`
- `discovery.run`
- `files.read`
- `files.write`
- `files.delete`
- `files.download`
- `files.upload`
- `terminal.create`
- `terminal.input`
- `terminal.resize`
- `terminal.terminate`
- `metrics.read`
- `services.read`
- `services.manage`
- `docker.read`
- `docker.manage`
- `roles.read`
- `roles.manage`
- `audit.read`
- `agent.install`
- `agent.upgrade`
- `agent.revoke`

This is a proposed starter vocabulary, not a frozen exhaustive registry. Permissions should be as narrow as practical, and write/destructive operations should not be bundled into a broad “server admin” permission when meaningful separation is possible.

The backend should use a centralized authorization service or policy layer rather than scattering ad hoc role-name checks throughout routes and handlers.

### 2.5 Roles bundle permissions; roles do not replace scopes

CPS should support configurable roles that bundle permissions. Candidate built-in roles for discussion are:
- **Owner:** manages the organization and its security settings.
- **Administrator:** manages servers and members within assigned organizational authority.
- **Operator:** performs operational actions explicitly granted by permissions.
- **Developer:** accesses only assigned servers and permitted resources or paths.
- **Viewer:** reads permitted server metadata, metrics, and audit information.

These are starting examples, not final immutable defaults. The permission set and scope of every built-in role must be documented and reviewed. Custom roles should be supported when safe to implement.

A role name must never act as a hidden bypass. For example, a user labeled “Developer” does not automatically receive access to all files or terminals.

### 2.6 Scope permissions to organization, server, and resource

Authorization must verify, as applicable:
1. The authenticated user is active.
2. The user has a valid membership in the active organization.
3. The target server belongs to that organization.
4. The user's role or grants include the requested permission.
5. The grant applies to the target server.
6. Any resource-specific restriction permits the requested target.

For filesystem operations, path restrictions must be enforced by trusted backend/agent logic using normalized paths and safe filesystem handling, not merely by the UI. A sample policy could allow `files.read`, `files.write`, and `terminal.create` on one assigned server while limiting file operations to `/opt/myapp/`. That example does not imply that a path allowlist safely confines an unrestricted shell; terminal access must be treated as a separate powerful permission.

The exact policy-combination semantics—such as role grants, direct grants, explicit denies, and inheritance—must be decided before implementation. Avoid ambiguous conflict rules.

### 2.7 Secure sessions and credentials

Authentication/session implementation must:
- Use a vetted authentication approach and secure password handling if CPS manages passwords.
- Store session secrets only in appropriately protected form where applicable.
- Apply secure cookie/session settings for browser sessions.
- Protect state-changing browser requests against CSRF where cookie-based authentication is used.
- Rate-limit and monitor authentication attempts.
- Support session expiration and revocation.
- Avoid putting tokens or credentials in URLs, logs, error messages, or client-side persistent storage without a reviewed design.
- Protect account recovery and any future MFA enrollment/recovery flows.

The exact authentication provider, password/session library, token format, MFA scope, and recovery workflow are not frozen by this ADR. Select and verify them during implementation rather than inventing a custom cryptographic scheme.

### 2.8 Secure WebSocket and terminal authorization

A WebSocket handshake does not permanently authorize every message sent through that connection.

For each WebSocket or terminal session, CPS must:
- Authenticate the initiating user.
- Verify organization and server scope.
- Verify the relevant permission before creating the session.
- Bind the session to the authenticated user, organization, server, and session ID.
- Reject attempts to attach to another user's session without explicit permission.
- Validate message type, size, sequence/state, and allowed operations.
- Re-check permissions for sensitive operations and respond to revocation.
- Terminate sessions when authentication expires or the relevant access is revoked, according to the documented session policy.

Do not rely solely on an unpredictable session ID as authorization.

### 2.9 Agent and server trust are separate from user access

A verified agent identity proves that CPS is communicating with an enrolled agent; it does not prove that the requesting user is allowed to use it.

Agent requests must be associated with the correct CPS server record and organization. The control plane must not let a user-selected server ID redirect an authorized operation to an agent belonging to another organization.

Agent enrollment, credential rotation, and revocation must follow ADR-002. Every operation must be checked against user authorization and the target host's current capability state.

### 2.10 Audit security-relevant actions

Audit events should include, where relevant:
- Actor user ID
- Organization ID
- Target server ID
- Action and permission identifier
- Resource identifier or safely represented path
- Result: allowed, denied, succeeded, failed, or cancelled
- Timestamp
- Request/correlation ID
- Relevant session or agent identity reference
- A bounded, redacted reason

Audit events must not store passwords, private keys, session tokens, bootstrap tokens, or unnecessary file contents. Avoid logging terminal input/output by default. If terminal recording is ever proposed, it requires a separate privacy, retention, consent, and access-control decision.

Audit-log access must itself be permission-controlled. Important security events should not be editable by ordinary users, and deletion/retention policy must be explicit.

## 3. Required authorization flow

For a protected operation, the backend should conceptually perform:

1. Authenticate the request.
2. Resolve the active organization context.
3. Verify membership and account/session status.
4. Resolve the target server and confirm organization ownership.
5. Check the required permission.
6. Check server and resource/path scope.
7. Confirm the operation is supported and its capability state is sufficiently fresh.
8. Validate the request and enforce operation-specific safety constraints.
9. Execute through the appropriate subsystem/agent.
10. Record the outcome in the audit trail.

This is a logical sequence, not a requirement to implement one monolithic function. Shared policy logic should be testable and consistently applied across HTTP, WebSocket, background jobs, and agent-mediated operations.

## 4. Security requirements and tests

The release gate must include tests for:
- Unauthenticated access to protected routes
- Cross-organization and cross-server access
- Missing, malformed, and revoked permissions
- Role escalation and unauthorized role/member management
- Resource/path-scope bypass
- WebSocket handshake and per-message authorization bypass
- Session hijacking, replay, expiry, and revocation
- CSRF where applicable, plus XSS and injection defenses
- Insecure direct object references and guessed IDs
- Unauthorized agent/server association
- Capability-state confusion or stale-state misuse
- Audit-log access, tampering boundaries, and secret redaction
- Concurrent role changes during active sessions
- Background jobs executing after a user's permission has been revoked

## 5. Consequences

### Benefits
- Authorization is consistent across the dashboard, API, terminal, file manager, plugins, and agent-mediated actions.
- Organizations and servers have explicit isolation boundaries.
- Roles remain configurable bundles rather than special-case code paths.
- Resource scopes can support least-privilege access for developers and operators.
- Security-sensitive actions become auditable.

### Costs and trade-offs
- A centralized policy layer and extensive tests are required.
- Resource/path scoping is more complex than simple role checks.
- Long-lived terminal and WebSocket sessions require explicit revocation behavior.
- Built-in roles and permission vocabularies need maintenance as modules are added.

## 6. Explicit non-decisions

This ADR does not yet freeze:
- The identity provider or exact login mechanism
- Password hashing/session libraries and configuration
- Whether MFA is required for the first release
- The final built-in role definitions
- The exhaustive permission registry
- The exact database schema
- Role-grant inheritance, direct grants, explicit-deny, or conflict semantics
- The path-scope matching algorithm
- Audit retention periods and any terminal-recording policy

These must be decided and documented before the corresponding feature ships.

## 7. Acceptance criteria

Before this ADR is marked accepted:
- [ ] Review it against the Master Specification, Constitution, and ADR-001 through ADR-003.
- [ ] Select a vetted authentication/session approach and document credential lifecycle.
- [ ] Finalize the initial permission registry and built-in role grants.
- [ ] Define role/grant conflict and inheritance semantics.
- [ ] Define organization/server/resource scope semantics.
- [ ] Define session expiry and revocation behavior for HTTP and WebSocket/PTY sessions.
- [ ] Define audit event schema, access controls, and retention.
- [ ] Implement centralized authorization tests, including cross-tenant cases.
- [ ] Approve the ADR in the repository.

## 8. Implementation sequence

1. Define identity, organization, membership, role, permission, and scope data models.
2. Select the authentication/session approach.
3. Implement centralized server-side authorization and policy tests.
4. Add organization/server isolation to every protected query and operation.
5. Add scoped file-operation authorization.
6. Add WebSocket and terminal session authorization/revocation.
7. Add structured audit events and redaction.
8. Run negative security tests before exposing operational modules.

**Current status: PROPOSED FOR REVIEW — not yet frozen.**
