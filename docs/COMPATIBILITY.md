# Compatibility contract

## Purpose

This document defines what TrueCode means by full server and functional runtime parity with the frozen OpenCode reference.
It also records intentional compatibility breaks and prevents broad claims based only on route coverage.

## Reference

The sole compatibility target for the first release is OpenCode `v2.2.7` at commit `fedc6f208aa2bd54c1019227214568083f6380ad`.
The target does not move when OpenCode changes.
See [../PROVENANCE.md](../PROVENANCE.md) for provenance and hashes.

## Compatibility dimensions

TrueCode must establish compatibility independently in each dimension below.

### Wire compatibility

- Match route methods and paths.
- Match path, query, header, and body decoding.
- Match successful and unsuccessful status codes.
- Match required response headers and content types.
- Match JSON field names, optionality, nullability, value domains, and unknown-field behavior.
- Match SSE framing, event names, data encoding, reconnect behavior, and terminal conditions.
- Match published WebSocket handshakes, messages, closure behavior, and errors.

### Behavioral compatibility

- Match session, message, permission, form, tool, PTY, MCP, worktree, and service state transitions.
- Match ordering between durable state changes, responses, and emitted events.
- Match cancellation, interruption, retry, reconnect, recovery, and compaction behavior.
- Match duplicate-request, concurrent-request, and invalid-state behavior.
- Match externally visible filesystem, process, Git, and configuration effects.

### Client compatibility

- Exercise official OpenCode clients unchanged against TrueCode where the client permits selecting a server endpoint.
- Treat client success as integration evidence rather than proof that unexercised behavior is compatible.
- Do not patch official clients to hide protocol differences.
- Keep TrueCode's implementation identity visible in its binary, service metadata, logs, and diagnostic surfaces.

### Provider compatibility

- Normalize provider-specific streaming into the same externally visible message and event behavior.
- Match model discovery, model metadata, provider options, credential selection, OAuth lifecycle, and error mapping.
- Use deterministic protocol fixtures for repeatability.
- Use newly provisioned dedicated credentials for approved live-provider validation before release.
- Never read prior provider credentials from OpenCode or TrueCode state.

### Platform compatibility

- Validate macOS on `aarch64-apple-darwin` natively.
- Validate GNU/Linux on `aarch64-unknown-linux-gnu` natively.
- Validate musl Linux separately before publishing a musl artifact.
- Do not infer PTY, process, signal, filesystem, or key-storage compatibility from compilation alone.

### Extension compatibility

- Provide a Rust SDK and a versioned WASI component interface for TrueCode plugins.
- Enforce declared plugin capabilities at the host boundary.
- Report unsupported OpenCode JavaScript and TypeScript plugin configuration clearly.
- Do not evaluate JavaScript as a hidden compatibility path.

## Included functional domains

The first compatibility release includes:

- Server discovery, health, lifecycle, shutdown, and implementation identity
- Authentication, pairing, and credential flows
- Projects, locations, configuration, instructions, skills, commands, and agents
- Sessions, messages, prompts, inbox behavior, generation, branching, revert, sharing, and summaries
- Permissions and forms, including pending requests and replies
- Global events, session events, SSE continuity, and reconnect behavior
- Provider and model discovery, generation, streaming, usage, errors, and OAuth
- Filesystem listing, reading, search, editing, watching, and path handling
- Shell execution, process trees, cancellation, output limits, and environment handling
- PTYs and persistent PTYs
- MCP connections, authentication, resources, prompts, tools, status, and failure handling
- Git status, repositories, branches, worktrees, and VCS metadata
- Plugins through the TrueCode Rust and WASI model
- Fresh SQLite persistence, migrations, crash recovery, and compaction
- Service installation, startup, readiness, status, shutdown, and stale-process recovery

## Intentional compatibility breaks

### Plugin ABI

TrueCode does not load OpenCode JavaScript or TypeScript plugins.
This is a permanent first-class compatibility break, not a temporary missing feature.

### Runtime identity

The executable and service are named `truecode`.
TrueCode may report the pinned OpenCode protocol compatibility version, but it must not claim to be the official OpenCode implementation.

### Existing state

TrueCode starts from a fresh namespace and fresh database.
No OpenCode or previous TrueCode sessions, databases, caches, retained content, or credentials are migrated or inspected.
Migration endpoints present in the frozen API must be behaviorally characterized, but they must not be used to import private historical state during development or acceptance.

### Clients

TrueCode does not reimplement or distribute forks of the OpenCode TUI, web client, or desktop client.
Official clients are external conformance consumers.

### Platforms

Windows and x86_64 are outside the first-release support matrix.
Their exclusion does not permit platform-specific behavior to leak into the supported macOS ARM64 and Linux AArch64 APIs.

## Evidence priority

Compatibility decisions use the following evidence together:

1. The immutable OpenAPI document and generated official client contracts.
2. Observable behavior from the frozen official release in disposable namespaces.
3. OpenCode tests and pinned source behavior.
4. Unchanged official-client behavior.
5. Explicit release documentation.

When evidence conflicts, the conflict must be recorded and resolved through a focused differential test.
A source-code reading alone is not sufficient when the distributed binary behaves differently.

## Accepted deviations

The only accepted deviations at repository kickoff are:

| Area | Deviation | Client impact |
| --- | --- | --- |
| Plugins | OpenCode JavaScript and TypeScript plugin ABIs are unsupported. | Plugin configurations using those ABIs fail with an explicit compatibility error. |
| Identity | The binary, service, and implementation identity are `truecode`. | Clients must connect to the TrueCode endpoint rather than spawning an official binary by name. |
| State | Historical OpenCode and TrueCode state is not imported. | Users begin with an empty runtime namespace. |
| Platforms | Windows and x86_64 artifacts are not produced for the first release. | Clients require a separately reachable supported TrueCode server. |

Any new deviation requires an ADR, a conformance case, an impact statement, and explicit user approval.

## Release acceptance

TrueCode may claim compatibility with the frozen release only when:

- Every frozen OpenAPI operation is accounted for by executable tests.
- Generated wire DTOs and the route manifest match the frozen contract.
- Required deterministic differential scenarios pass.
- Streaming order, cancellation, reconnect, interruption, and recovery suites pass.
- Official-client scenarios pass without client patches.
- Supported provider and OAuth scenarios pass with fixtures and approved fresh live credentials.
- Native macOS ARM64 and Linux AArch64 verification passes.
- Security review, fuzzing, fault injection, and soak testing have no unresolved release-blocking findings.
- Every remaining incompatibility appears in this document and has explicit approval.

Performance measurements are reported but have no numerical release threshold.
Unbounded growth, deadlock, corruption, and failure to cancel remain correctness failures.
