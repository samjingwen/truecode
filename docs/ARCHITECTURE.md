# Architecture

## Goals

TrueCode will implement the frozen OpenCode server and functional runtime contract as a native Rust service.
The architecture prioritizes behavioral fidelity, explicit state ownership, bounded resource use, crash recovery, and platform-correct process behavior.

The internal design may differ from OpenCode when the external behavior remains compatible.
Compatibility-sensitive choices must be established through differential tests rather than architectural preference alone.

## Process model

The first release uses one `truecode` service process per configured runtime namespace.
The process owns HTTP listeners, runtime coordination, SQLite access, provider connections, MCP peers, subprocesses, PTYs, and plugin instances.

Long-running resources have an explicit owner and cancellation path.
Detached background tasks are prohibited unless their lifecycle and shutdown behavior are documented and tested.

The service exposes readiness only after configuration, storage, migrations, runtime recovery, and listener setup have succeeded.
Graceful shutdown stops admission, cancels or drains work according to the observed reference contract, closes streams, terminates owned processes, checkpoints durable state, and then exits.

## Planned workspace

| Crate | Responsibility |
| --- | --- |
| `truecode-protocol` | Generated wire DTOs, schema metadata, route manifest, event envelopes, and protocol compatibility version. |
| `truecode-server` | Axum routes, Tower middleware, authentication, pairing, limits, SSE, WebSockets, content negotiation, and graceful shutdown. |
| `truecode-runtime` | Durable session state machines, agent loop, tool orchestration, permission and form waits, cancellation, recovery, compaction, and event ordering. |
| `truecode-storage` | Fresh SQLite schema, migrations, repositories, transactions, recovery queries, and storage diagnostics. |
| `truecode-providers` | Provider clients, model discovery, credential selection, OAuth, request translation, and streaming normalization. |
| `truecode-tools` | Permission-aware filesystem, editing, search, shell, process, Git, and worktree tools. |
| `truecode-terminal` | Normal PTYs, persistent PTYs, process groups, signals, resize, attach, detach, and cleanup. |
| `truecode-mcp` | MCP transports, authentication, lifecycle, resources, prompts, tools, and error normalization. |
| `truecode-plugins-host` | Wasmtime component host, capability enforcement, resource limits, version negotiation, and plugin lifecycle. |
| `truecode-plugins-sdk` | Rust SDK, WIT bindings, manifest types, test host, and plugin authoring support. |
| `truecode-service-cli` | `truecode` binary, daemon lifecycle, configuration locations, status, diagnostics, implementation identity, and local administration. |
| `truecode-conformance` | Reference and candidate launchers, deterministic fixtures, transcript capture, canonicalization, comparison, and reports. |

Crate names describe boundaries rather than requiring one package per concept from the first commit.
Splitting or combining crates requires preserving the dependency rules below.

## Dependency direction

The intended dependency flow is:

```text
service-cli -> server -> runtime -> storage
                    |          -> providers
                    |          -> tools -> terminal
                    |          -> mcp
                    |          -> plugins-host
                    +----------> protocol

plugins-sdk -> protocol-compatible WIT types
conformance -> launched binaries and public protocol only
```

`truecode-protocol` must not depend on server, runtime, or storage implementation crates.
Provider, MCP, tool, terminal, and plugin adapters must not call HTTP handlers directly.
The conformance harness must prefer public process and network boundaries so it does not pass by sharing implementation internals.

## Request flow

A compatibility-sensitive mutation follows this sequence:

1. Tower middleware applies request size, timeout, tracing, authentication, and admission rules.
2. Axum decodes the request through generated protocol types and compatibility validators.
3. The handler sends a typed command to the owning runtime component.
4. The runtime validates the current durable state and permissions.
5. Storage commits the state transition and any outbox event records in one transaction.
6. The runtime publishes committed events in sequence order.
7. The handler produces the contract response.

The exact relationship between response completion and event publication must match the frozen reference behavior for each affected operation.
The durable outbox principle prevents clients from observing events for transitions that were not committed.

## Runtime ownership

Each active session has one logical command serializer.
Concurrent reads may use immutable snapshots where the API permits them.
Mutations that affect ordering pass through the session owner or an equivalent transactional serialization mechanism.

Global resources, including provider credential state, MCP connections, persistent PTYs, and service shutdown, have separate owners.
Ownership must not require one Tokio task per inactive persisted entity.

## Asynchronous execution

Tokio is the planned asynchronous runtime.
Every queue must have a documented capacity, overload behavior, cancellation behavior, and metric.
Unbounded channels are not permitted in production paths.

CPU-heavy parsing, compression, image work, and blocking host APIs must not run on asynchronous executor threads without an explicit budget.
SQLite access will use an explicit blocking boundary and bounded work admission.

## HTTP and streaming

Axum and Tower are the planned server stack.
Streaming responses must avoid proxy and middleware buffering that changes frame timing.
Slow clients must encounter a defined backpressure or disconnect policy rather than causing unbounded event retention.

SSE connections use durable event sequence information where the reference contract exposes replay or cursor behavior.
Ephemeral events that cannot be replayed must have explicitly tested reconnect semantics.

WebSocket tasks must coordinate reader, writer, heartbeat, shutdown, and peer closure without orphaning tasks.

## Storage boundary

SQLite is the source of truth for durable runtime state.
The first schema is native to TrueCode and does not attempt to match OpenCode's physical schema.
Repositories expose domain operations rather than raw SQL to HTTP handlers.

Write transactions remain short and do not include provider calls, process waits, MCP requests, or plugin execution.
Durable intent may be committed before external work begins, followed by explicit completion or failure transitions.

## Provider boundary

Provider adapters translate between provider protocols and a normalized internal stream of message-start, content, reasoning, tool-call, usage, completion, and error events.
Normalization must preserve enough raw information to reproduce the frozen external behavior.
Provider-specific extensions stay typed and cannot leak unchecked JSON through unrelated runtime layers.

OAuth flows are state machines with explicit expiry, refresh, cancellation, and storage behavior.
Credential material is never included in conformance snapshots.

## Tool boundary

Tool dispatch performs permission resolution before creating side effects.
The bounded process runner owns process groups, standard streams, output budgets, deadlines, cancellation, and exit status translation.
Filesystem tools resolve workspace boundaries and account for symlink replacement between validation and use.

Git behavior uses the installed Git executable where library behavior would diverge from user-visible Git semantics.
Git commands still pass through the bounded process runner and environment policy.

## PTY boundary

PTY creation, resize, signal, attach, detach, persistence, and teardown are isolated behind platform interfaces.
macOS and Linux implementations may differ internally while preserving the protocol contract.

Persistent PTYs have a durable identity and a live process identity.
Recovery must distinguish an attachable surviving process from stale persisted metadata.
The service must not silently attach to unrelated host processes after restart.

## MCP boundary

MCP peers are untrusted external systems.
Each connection has transport, authentication, initialization, capability, timeout, cancellation, and shutdown state.
MCP resource, prompt, and tool results pass through size and schema limits before entering session state.

## Plugin boundary

Plugins run as WASI components under Wasmtime.
The host grants named capabilities and does not ambiently expose the host filesystem, environment, network, clocks, randomness, or subprocess execution.
Any granted host function remains subject to TrueCode permissions and resource limits.

The plugin component interface is independent from OpenCode's JavaScript and TypeScript ABI.

## Configuration and locations

Configuration is layered deterministically from service arguments, approved environment overrides, user configuration, project configuration, and discovered instruction or skill files according to characterized reference precedence.
Unknown fields fail by default unless compatibility evidence requires tolerant handling at a specific layer.

Tests always override config, data, cache, home, repository, socket, and credential locations into a disposable namespace.
Production and test paths never fall back to OpenCode directories.

## Observability

Structured logs include request, session, run, tool, provider, MCP, PTY, and plugin correlation identifiers where available.
Logs exclude prompts, file contents, credentials, authorization headers, OAuth tokens, and raw provider bodies by default.

Metrics must cover queue saturation, active streams, subprocesses, PTYs, MCP connections, plugin instances, SQLite latency, event lag, cancellations, and shutdown duration.
Diagnostics expose TrueCode's own implementation version and the frozen protocol target separately.

## Dependency acceptance

Major direct dependencies require an ADR or an update to an existing architecture decision before adoption.
Selection must consider maintenance, security posture, platform support, cancellation semantics, memory behavior, and the ability to match the frozen contract.
Development cost is secondary to correctness, simplicity, robustness, and long-term maintainability.
