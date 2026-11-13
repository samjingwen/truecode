# Differential conformance

## Objective

The conformance system determines whether TrueCode matches the frozen OpenCode server and runtime behavior.
It launches the official reference and TrueCode separately, drives the same scenario into each, captures externally visible results, canonicalizes only proven nondeterminism, and reports semantic differences.

The harness must exist before broad runtime implementation so behavior is discovered and locked before code is shaped around assumptions.

## Reference and candidate

The reference is an official OpenCode `v2.2.7` artifact whose digest appears in [../PROVENANCE.md](../PROVENANCE.md).
The candidate is an exact locally built TrueCode commit.

Each process runs with:

- A unique temporary home directory
- Unique config, data, and cache roots
- A unique temporary repository fixture
- Unique sockets, ports, and process namespaces where available
- No access to previous OpenCode or TrueCode runtime state
- No access to normal provider credentials
- A controlled environment variable allowlist

The harness records the artifact digest, candidate commit, operating system, architecture, libc where relevant, fixture version, and scenario seed.

## Harness components

### Process supervisor

The supervisor starts the reference and candidate, captures process trees and logs, waits for readiness, enforces deadlines, and terminates all owned descendants.
Readiness is determined through the public service contract rather than a fixed sleep.

### Scenario driver

The driver sends typed HTTP requests, consumes SSE and WebSocket streams, controls disconnects, injects cancellation, and coordinates deterministic external fixtures.
It maintains separate client state for each implementation so identifiers returned by one are never sent to the other without mapping.

### Deterministic provider

The provider fixture supports scripted:

- Non-streaming and streaming responses
- Text and reasoning deltas
- Tool calls split across arbitrary chunks
- Usage reports
- Rate limits and retry hints
- Authentication errors
- Malformed frames
- Mid-stream disconnects
- Slow responses and cancellation races
- Duplicate and out-of-order provider data

Scripts operate from a shared logical scenario while allowing implementation-specific request identifiers and headers.

### OAuth fixture

The OAuth fixture supports authorization code, PKCE, refresh, expiry, denial, invalid state, token endpoint errors, delayed callbacks, and cancellation.
Tokens are synthetic and scoped to the disposable run.

### MCP fixtures

Fixtures cover stdio and network transports supported by the frozen release.
They expose deterministic tools, prompts, resources, capability changes, authentication challenges, slow operations, invalid payloads, disconnects, and restart behavior.

### Repository and tool fixtures

Repository fixtures contain controlled files, symlinks, ignored files, binary files, long paths, Unicode names, Git history, branches, conflicts, submodules where supported, and worktrees.
Tool fixtures cover standard output, standard error, large output, no output, signals, timeouts, process descendants, shell quoting, environment handling, and cancellation.

### PTY fixtures

PTY scripts expose echo, alternate-screen behavior, resize reporting, signal handling, child processes, attach and detach, output while detached, shell exit, and service restart cases.
PTY tests run natively because terminal and signal semantics cannot be established through schema tests.

### Transcript recorder

The recorder captures:

- Requests after client construction and before transport
- Response status, ordered headers where meaningful, content type, and body bytes
- Decoded JSON with the original body retained
- SSE frame boundaries, fields, payload bytes, and arrival sequence
- WebSocket message type, payload, direction, close code, and order
- Observable filesystem and Git effects
- Child process exit, signal, and cleanup evidence
- Process-tree memory and timing measurements when enabled

## Canonicalization

Canonicalization makes equivalent runs comparable without hiding behavior.
Every rule must name the field, justify its nondeterminism, and have a test proving it does not erase meaningful differences.

Allowed examples include:

- Temporary absolute path prefixes
- Dynamically allocated loopback ports
- Wall-clock timestamps where exact values are not contractual
- Random identifiers mapped by creation order and entity type
- Platform-specific path separators only where the public contract treats them as equivalent

Forbidden canonicalization includes:

- Sorting events, messages, tools, or errors when order is observable
- Removing fields because one implementation omits them
- Replacing distinct error categories with a generic error
- Ignoring status codes or content types
- Dropping cancellation or recovery transitions
- Normalizing malformed output into valid output
- Hiding leaked credentials or unexpected filesystem paths

## Comparison levels

### Structural comparison

Structural comparison validates routes, methods, schemas, required fields, enums, and content types against the frozen OpenAPI document.

### Transcript comparison

Transcript comparison validates ordered requests, responses, streams, errors, and externally visible effects for a scenario.

### State comparison

State comparison queries both implementations through public APIs at checkpoints and compares the resulting logical state.
It does not compare private database tables.

### Client comparison

Client comparison runs unchanged official clients against each implementation and compares user-visible behavior and network transcripts.

### Live integration comparison

Live comparison uses newly provisioned dedicated provider and OAuth credentials supplied for this project.
Secrets are injected at runtime, redacted from evidence, and destroyed or revoked according to the test plan.
Fixture tests remain the reproducible primary evidence.

## Scenario families

The required scenario catalog includes:

1. Service startup, readiness, status, shutdown, stale state, and restart.
2. Authentication, pairing, unauthorized requests, malformed credentials, and revocation.
3. Configuration locations, precedence, unknown fields, project discovery, instructions, skills, commands, and agents.
4. Project, location, filesystem, search, and reference operations.
5. Session creation, listing, retrieval, update, deletion, branching, sharing, revert, and summary behavior.
6. Message submission, streaming, tool calls, usage, provider errors, and completion.
7. Concurrent prompts, duplicate requests, interruption, cancellation, reconnect, and recovery.
8. Permission and form creation, listing, reply, rejection, expiry, and disconnect behavior.
9. Compaction, long sessions, context limits, retry, and restart during transition boundaries.
10. Provider discovery, model normalization, options, OAuth, refresh, and credential failures.
11. MCP lifecycle, authentication, tools, prompts, resources, timeout, reconnect, and invalid peers.
12. Shell, filesystem, editing, search, Git, worktree, and VCS side effects.
13. PTY and persistent PTY creation, I/O, resize, attach, detach, exit, cancellation, and restart.
14. Plugin discovery, capability grant, invocation, limits, failure, and unsupported JavaScript plugin errors.
15. Malformed requests, oversized inputs, slow clients, disconnects, and resource saturation.

Every frozen OpenAPI operation must map to at least one scenario.
Operations with meaningful state or failure behavior require multiple scenarios.

## Fault injection

Fault injection covers process kill, service restart, SQLite busy and I/O failures, disk-full simulation where safe, provider disconnect, MCP disconnect, dropped client connections, child process refusal to terminate, clock changes, and partial writes at controlled boundaries.

The harness must prove that owned resources are cleaned up and durable state remains explainable after each injected fault.
Fault injection must never target non-disposable user data or host services.

## Performance observation

The same deterministic workload records:

- Cold process readiness
- Idle process-tree memory
- Peak process-tree memory
- Local API latency
- Stream overhead apart from provider network latency
- Resource behavior over repeated sessions, tools, reconnects, PTYs, and compactions

These measurements are reported without numerical release gates.
Provider network latency is reported separately from local runtime overhead.
Unbounded growth is investigated as a correctness defect.

## Native platform execution

GitHub-hosted CI is disabled.
Release evidence is produced through documented local commands on native macOS ARM64 and native Linux AArch64 hosts.

Each result records the exact commit and host facts.
Cross-compilation proves only that an artifact builds and does not replace native process, PTY, signal, filesystem, or service-lifecycle tests.

## Result classification

Differences are classified as:

- `match`: no meaningful difference after approved canonicalization
- `candidate defect`: TrueCode differs from established reference behavior
- `reference defect`: the official release behaves inconsistently or unsafely and the behavior needs an explicit compatibility decision
- `contract ambiguity`: available evidence does not establish one expected behavior
- `approved deviation`: the difference is documented and explicitly accepted
- `harness defect`: the fixture, capture, or canonicalization is invalid

Only `match` and approved deviations count toward release acceptance.

## Evidence retention

Checked-in evidence contains schemas, fixture definitions, canonicalization rules, concise expected transcripts, and reports without secrets or machine-local paths.
Large raw traces remain in ignored disposable storage unless a sanitized failure fixture is deliberately promoted.

Conformance reports identify all skipped scenarios and unavailable platforms.
A partial run must never be labeled as a complete pass.

## Release gate

Before a compatibility release:

- All 170 operations must have executable coverage.
- Required deterministic scenarios must pass on the supported native platforms.
- Streaming and recovery ordering must have no unresolved differences.
- Official-client scenarios must pass without patches.
- Approved live provider and OAuth validation must be complete.
- Security, fuzzing, fault-injection, and soak findings must be resolved or explicitly accepted.
- `docs/COMPATIBILITY.md` must list every remaining deviation.
