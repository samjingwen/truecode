# ADR 0002: Rust runtime architecture

- Status: Accepted
- Date: 2026-11-13

## Context

TrueCode needs full server and functional runtime parity while improving control over memory, process ownership, event ordering, cancellation, and service lifecycle.
The runtime coordinates HTTP streams, provider streams, subprocesses, PTYs, MCP peers, plugins, and durable SQLite state.

Implicit shared state and unowned background tasks would make recovery and conformance difficult.

## Decision

TrueCode will be a stable-Rust workspace organized around protocol, server, runtime, storage, providers, tools, terminal, MCP, plugins, service CLI, and conformance boundaries.

Axum and Tower are the planned HTTP stack.
Tokio is the planned asynchronous runtime.
SQLite is accessed through a bounded blocking boundary.

Sessions and runs are explicit durable state machines.
Externally visible transitions and outbox events commit atomically before live publication.
Queues, output buffers, retries, subprocesses, and task lifetimes are bounded and owned.

Major dependency selection remains subject to focused dependency decisions before implementation.

## Consequences

The architecture favors explicit ownership and recoverability over direct translation of the reference TypeScript structure.
The internal implementation may differ while wire and behavioral results remain compatible.

Some platform code may require carefully isolated `unsafe` Rust.
Concurrency optimizations must preserve durable ordering and cannot bypass the session state machine.
