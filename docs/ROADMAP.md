# Roadmap

## Delivery model

TrueCode is delivered through evidence-based milestones.
Milestones close only when their exit criteria are met.
Dates do not override correctness, conformance, or security requirements.

GitHub-hosted CI is disabled.
Local exact-head verification, independent Hardlook review, protected pull requests, and native platform evidence form the delivery boundary.

## Milestone 0: Documentation and frozen reference

### Deliverables

- Public MIT-licensed repository with `master` as the default branch
- Frozen OpenCode tag, commit, artifact digests, license, and OpenAPI source
- Compatibility, API inventory, architecture, conformance, runtime, storage, security, plugin, and platform documents
- Accepted architecture decisions
- Root agent instructions
- Protected `master` branch
- Adapted change-delivery policy

### Exit criteria

- The documentation consistently names OpenCode `v2.2.7` and commit `fedc6f208aa2bd54c1019227214568083f6380ad`.
- The preserved OpenAPI and license hashes match `PROVENANCE.md`.
- The API inventory reproducibly reports 139 paths, 170 operations, and 497 schemas.
- Intentional exclusions are explicit.
- No Rust runtime implementation has been added.
- The initial documentation has been reviewed before conformance implementation begins.

## Milestone 1: Behavioral inventory and conformance foundation

### Deliverables

- Verified official artifacts in ignored disposable storage
- Process supervisor for frozen reference and candidate binaries
- Disposable namespace builder
- HTTP, SSE, and WebSocket transcript recorder
- Schema-derived route and operation manifest
- Deterministic provider, OAuth, MCP, repository, tool, Git, and PTY fixtures
- Explicit canonicalization rules and tests
- Initial official-client launch and endpoint-selection probes
- Reference behavior reports for service, session, event, error, cancellation, and recovery scenarios

### Exit criteria

- Every one of the 170 operations maps to a planned executable scenario.
- The harness can detect intentional differences in status, JSON, SSE order, errors, cancellation, and side effects.
- No harness process reads normal OpenCode or TrueCode state.
- Reference runs terminate all owned process descendants.
- Unknown behavior is recorded as an inventory question with a reproducible probe.

## Milestone 2: Protocol, service, and storage foundation

### Deliverables

- Stable Rust workspace and pinned toolchain
- Generated protocol DTOs and route manifest tied to the source OpenAPI hash
- Axum and Tower transport foundation
- Authentication and pairing middleware
- Strict configuration and location handling
- `truecode` service CLI and implementation identity
- Fresh SQLite baseline schema, migration runner, repository boundary, and event outbox
- Startup, readiness, status, graceful shutdown, and stale-owner handling
- Local formatting, lint, unit, integration, and conformance entrypoints

### Exit criteria

- Protocol generation is reproducible and generated files are not hand-edited.
- Foundation routes match differential status, header, body, and error cases.
- Fresh-state tests prove no OpenCode or prior TrueCode path is discovered.
- Storage crash tests prove committed transitions and outbox events remain consistent.
- Service lifecycle passes natively on macOS ARM64 and Linux AArch64 for implemented behavior.

## Milestone 3: Durable session and agent runtime

### Deliverables

- Session, message, part, run, and step state machines
- Ordered committed event publication
- Provider-neutral streaming state machine
- Prompt and message APIs
- Permission and form waits
- Cancellation hierarchy
- Interruption and restart recovery
- Compaction and long-session behavior
- Bounded inbox and concurrency handling

### Exit criteria

- Core session and message scenarios match the frozen reference.
- Event ordering, reconnect, cancellation, and recovery scenarios pass.
- No provider, tool, or client disconnect creates an orphaned runtime task.
- Repeated session and compaction soak tests show no unbounded resource growth.

## Milestone 4: Providers, tools, MCP, terminals, and VCS

### Deliverables

- Frozen provider and model matrix
- Provider request translation and streaming normalization
- OAuth state machines and approved credential backend
- Filesystem, editing, search, shell, and bounded process tools
- Git, worktree, and VCS behavior
- MCP transports, authentication, resources, prompts, tools, and lifecycle
- Normal and persistent PTYs
- Configuration, instruction, skill, command, and agent discovery
- Live-provider validation using newly supplied dedicated credentials

### Exit criteria

- Deterministic fixture suites pass for every supported integration behavior.
- Approved live provider and OAuth scenarios pass without using old credentials.
- Process descendants, MCP work, and PTYs cancel and clean up reliably.
- Git and worktree scenarios preserve unrelated user changes.
- Native macOS ARM64 and Linux AArch64 platform-sensitive suites pass.

## Milestone 5: Rust and WASI plugins

### Deliverables

- Versioned WIT interface packages
- Wasmtime component host
- Capability manifest and grant model
- Resource and cancellation controls
- Rust plugin SDK and test host
- Plugin-scoped state
- Explicit OpenCode JavaScript and TypeScript plugin rejection
- Example components used only as test fixtures and documentation

### Exit criteria

- Capability denial and grant tests pass.
- Guest traps, resource exhaustion, cancellation, and shutdown cannot crash or orphan the service.
- Component digest changes invalidate grants as designed.
- Unsupported OpenCode plugins fail explicitly without evaluation.

## Milestone 6: Full conformance and hardening

### Deliverables

- Complete operation coverage report
- Official-client conformance report
- Protocol, parser, and state-machine fuzzing
- Fault-injection report
- Security review and threat-model update
- Native platform reports
- Repeated-session, reconnect, tool, PTY, MCP, plugin, and compaction soak reports
- Memory, readiness, local API latency, and provider-separated performance report
- Complete compatibility deviation register

### Exit criteria

- All required deterministic scenarios pass or have explicitly approved deviations.
- Official clients pass required scenarios unchanged.
- No unresolved release-blocking security or correctness finding remains.
- Performance is reported without numerical gates.
- No unbounded growth, deadlock, corruption, or cancellation leak remains.

## Milestone 7: Candidate release and downstream integration

### Deliverables

- Source-bound macOS ARM64 and Linux AArch64 artifacts with digests
- Installation, configuration, service, troubleshooting, and rollback documentation
- Compatibility statement scoped to OpenCode `v2.2.7`
- TrueCode-native backup and recovery documentation
- TenAxe candidate integration behind an explicit implementation selection

### Exit criteria

- Artifacts come from a clean protected `master` revision.
- Artifact manifests identify source, toolchain, target, and protocol compatibility.
- TrueCode retains a distinct binary and version identity.
- TenAxe's existing tool denials and security boundaries remain in force.
- The official OpenCode runtime remains the downstream default until separate acceptance authorizes a switch.

## Deferred work

The following work is outside the first compatibility release:

- Windows support
- x86_64 support
- OpenCode JavaScript or TypeScript plugin ABI compatibility
- Import of historical OpenCode or TrueCode state
- Reimplementation of official clients
- Automatic tracking of later OpenCode releases
- Remote plugin registries and automatic plugin updates
- GitHub-hosted CI

Deferred items require their own decisions and cannot be added by silently expanding a milestone.
