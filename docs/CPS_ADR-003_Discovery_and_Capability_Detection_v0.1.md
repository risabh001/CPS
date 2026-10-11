# CPS Architecture Decision Record 003
## VPS Discovery and Capability Detection

- **Status:** Proposed for review
- **Decision date:** 2026-10-11
- **Decision owners:** CPS maintainers
- **Related documents:** CPS Project Master Specification v0.1; CPS Project Constitution v0.1; ADR-001 — Technology Stack v0.1; ADR-002 — SSH Onboarding and Agent Architecture
- **Scope:** How CPS discovers a managed Linux VPS, normalizes its profile, determines which capabilities are available, and uses those results to present modules safely.

---

## 1. Context

CPS must support Linux VPS instances that differ in distribution, installed tools, service manager, permissions, container runtime, and available system resources.

A single fixed feature list would incorrectly suggest that every VPS supports every operation. CPS instead needs a structured profile and a capability model that can represent available, unavailable, restricted, unknown, and stale states.

Discovery must be useful without becoming an unrestricted filesystem scan or collecting application secrets. It must also work when the initial SSH user has limited permissions or some standard commands are absent.

## 2. Decision

### 2.1 Discovery is bounded and metadata-oriented

Discovery collects the minimum system metadata needed to describe the host and determine supported CPS operations. It must not recursively read, index, or download the VPS filesystem.

Discovery should use a documented set of bounded probes, with timeouts, output-size limits, and clear error handling. Each probe should declare:
- What it checks
- Why CPS needs the information
- Whether elevated privileges are required
- Its timeout and output limits
- How failure or missing commands are represented
- Whether the result is safe to retain and for how long

The implementation must not assume that all Linux distributions have the same package manager, service manager, command paths, or installed utilities.

### 2.2 Build a normalized VPS profile

CPS converts raw discovery results into a versioned, normalized VPS profile. The profile should distinguish observed facts from inferred or unknown values.

The profile may include the following categories when successfully detected:
- Stable CPS server record ID and agent enrollment state
- Hostname and operating-system identification
- Kernel and architecture
- CPU topology and available CPU metrics
- Memory and swap totals/availability
- Filesystem and mount metadata, capacity, and usage
- Uptime and load averages
- Network interface metadata and counters
- Service-manager availability
- Docker availability and access status
- Relevant runtime/tool availability
- Agent version, protocol compatibility, and last-seen time
- Discovery timestamp, probe status, and profile schema version

The profile must not include passwords, private keys, authentication tokens, environment-variable dumps, arbitrary process command-line secrets, or file contents. Any future field that could expose sensitive data requires a separate review.

Hostnames and operating-system metadata are descriptive attributes, not proof of server identity. The stable CPS server record ID and verified agent identity must be handled separately.

### 2.3 Every capability has an explicit state

Each capability is represented as structured data rather than a simple boolean. The minimum state vocabulary is:

- `available` — the capability was positively detected and the required access checks passed.
- `unavailable` — the capability is not present or cannot be provided by this host/configuration.
- `restricted` — the capability may exist, but the current CPS/OS permissions are insufficient.
- `unknown` — discovery could not determine the state reliably.
- `stale` — a previously observed result is no longer fresh enough to present as current.

A capability record should include:
- Stable capability identifier
- State
- Detection source/probe
- Reason or user-safe diagnostic
- Required permissions, if applicable
- Last checked timestamp
- Relevant version or compatibility information, when applicable

Do not map probe failure automatically to `unavailable`. A timeout, missing permission, unsupported probe, or network error may mean `unknown` or `restricted`.

### 2.4 Separate presence, access, and operational readiness

CPS must not confuse a detected tool with a usable capability.

For example, Docker may be installed while the agent cannot access its socket. In that case, Docker is present, but Docker management is restricted. Similarly, a service manager may exist while the current user lacks permission to inspect or control particular services.

Where practical, detection should distinguish:
1. **Presence:** Is the underlying component installed or exposed?
2. **Access:** Can the current agent identity perform the intended read or write operation?
3. **Readiness:** Are the prerequisites for the specific CPS feature satisfied?

Write-capable actions must have stronger readiness checks than read-only display. A read-only check must not imply that a destructive or privileged operation is authorized.

### 2.5 Discovery is layered and incremental

Use separate discovery stages:

**Stage A — Initial SSH discovery**
- Collect basic OS, kernel, architecture, CPU, memory, uptime, disk/mount, and basic tool metadata.
- Use the privileges available to the supplied SSH account.
- Mark inaccessible or ambiguous data explicitly.
- Do not install packages or change system configuration.

**Stage B — Agent capability detection**
- After explicit agent installation and enrollment, re-check capabilities using the agent's actual runtime environment and permissions.
- Detect filesystem, PTY, monitoring, service-manager, and Docker capabilities individually.
- Keep the results of SSH discovery distinguishable from agent observations.

**Stage C — On-demand re-check**
- Re-check a capability when a user opens a relevant module, before a sensitive operation, after an agent upgrade, or when an error suggests that state has changed.
- Avoid repeatedly running expensive or intrusive probes.
- Use TTLs or freshness rules appropriate to each data type; do not assume one global cache duration is suitable for all observations.

Discovery must be idempotent: repeating a probe must not change the managed host's configuration.

### 2.6 Dynamic modules use capability gates

The user interface should use capability results to determine which modules can be shown as available, limited, or unsupported.

Examples:
- Show Docker management as unavailable when Docker is absent.
- Show Docker management as restricted when Docker exists but the agent lacks access.
- Show service management only when a supported service manager and required permissions are detected.
- Show agent-dependent features as unavailable or pending when the agent is not enrolled or is disconnected.
- Show an explanatory reason and a safe next step instead of silently hiding every unsupported feature.

A visible module does not grant authorization. UI capability gates are for usability; the backend must independently authorize every operation and verify that the relevant capability is still valid.

### 2.7 Use capability identifiers, not UI labels, as contracts

Capabilities must have stable machine-readable identifiers, such as `system.metrics.read`, `filesystem.read`, `filesystem.write`, `terminal.pty`, `services.read`, `services.manage`, `docker.read`, and `docker.manage`.

These examples are illustrative; the final capability registry and mapping to the RBAC permission model must be reviewed before implementation. Display labels may change without changing capability IDs. A capability indicates technical support/readiness, while RBAC decides whether a particular user may use it.

### 2.8 Profile freshness and change detection

CPS records when a profile and each capability were last checked. The UI must distinguish current observations from stale observations.

When a material change is detected—such as Docker being installed or removed, the agent version changing, or permissions changing—CPS updates the relevant capability states and module availability. A change in observed capability must not silently broaden user permissions.

Historical values may be retained for diagnostics where useful, but retention limits and sensitive-data handling must be documented. The profile must not be treated as a real-time source for metrics that require a separate monitoring stream.

## 3. Error handling and diagnostics

- Missing utilities should produce a structured probe result, not crash the entire discovery run.
- Permission failures should be represented as restricted when evidence supports that conclusion.
- Timeouts and malformed output should be reported as unknown or failed probes with bounded diagnostic text.
- Partial discovery should still produce a usable profile.
- Raw probe output must not be exposed to ordinary users if it could contain secrets or internal system details.
- Diagnostics must redact credentials, tokens, private-key data, and other sensitive values.
- A failed probe must not silently erase a previously valid profile; mark relevant fields stale or unknown according to the freshness policy.

## 4. Security and privacy requirements

Discovery and capability detection must be tested for:
- Command injection through hostname, paths, usernames, or other user-controlled fields
- Unbounded execution, output, CPU, memory, and disk use
- Collection of environment variables, secrets, or file contents
- Incorrect assumptions about root/sudo permissions
- Misclassification of a restricted capability as available
- Stale capability state authorizing a sensitive action
- Cross-server profile or capability data leakage
- Malformed probe output and unsupported operating systems
- Log and diagnostic secret leakage
- Capability/UI mismatch with backend authorization

Capability detection is not an authorization system. It must never override RBAC, server scope, or resource/path scope.

## 5. Consequences

### Benefits
- CPS can adapt the UI and supported operations to the actual host.
- Partial permissions and uncertain results are represented honestly.
- Discovery remains bounded and less invasive.
- The same profile model can be used by the UI, API, agent, diagnostics, and future plugins.

### Costs and trade-offs
- A structured capability registry and schema require ongoing maintenance.
- Probe results may become stale and need revalidation.
- Different Linux distributions and restricted environments require broader testing.
- UI module states and backend authorization must be maintained as separate concerns.

## 6. Explicit non-decisions

This ADR does not yet freeze:
- The complete normalized profile schema or database representation
- The exact command/probe list for every Linux distribution
- The exact capability ID registry and its final relationship to RBAC permissions
- TTL values for individual profile fields and capabilities
- Long-term retention periods for historical profile observations
- Support guarantees for non-Linux hosts or unusual containerized environments
- The plugin API for third-party capability providers

These must be resolved in implementation design and testing. Do not treat illustrative fields or capability IDs in this draft as a final schema.

## 7. Acceptance criteria

Before this ADR is marked accepted:
- [ ] Review it against the Master Specification, Constitution, ADR-001, and ADR-002.
- [ ] Define the versioned normalized profile schema.
- [ ] Define the initial discovery probe list and per-probe limits.
- [ ] Define capability state transitions and freshness rules.
- [ ] Map capabilities to RBAC permissions without conflating the two.
- [ ] Define which profile fields are retained and their retention limits.
- [ ] Test partial discovery on hosts with missing tools and restricted permissions.
- [ ] Test that failed or stale probes cannot authorize sensitive operations.
- [ ] Approve the ADR in the repository.

## 8. Implementation sequence

1. Define the profile schema and versioning strategy.
2. Implement bounded SSH probes and structured probe results.
3. Normalize results into the server profile.
4. Implement the capability registry and state model.
5. Add agent-based capability re-checks after ADR-002 enrollment.
6. Add UI module gating with explanatory states.
7. Add on-demand revalidation before sensitive operations.
8. Test across multiple Linux distributions and permission configurations.

**Current status: PROPOSED FOR REVIEW — not yet frozen.**
