# Runtime semantics

## Purpose

The TrueCode runtime coordinates durable sessions, provider streams, tools, permissions, forms, MCP calls, PTYs, compaction, and client events.
Its internal model is explicit so cancellation, restart, and concurrent requests produce explainable outcomes.

The state names in this document are implementation concepts.
External state, event, and error shapes must match the frozen contract even when their names differ.

## Core entities

### Session

A session is the durable conversation and execution boundary exposed through the server API.
It owns ordered messages, run history, summaries, compaction state, sharing state, and session-level metadata.

### Message and part

Messages and their parts are durable protocol entities.
Provider deltas may be accumulated internally, but externally visible parts and update events must follow the reference ordering and partial-state behavior.

### Run

A run is one attempt to advance a session from a user request through provider and tool work to a terminal result.
A run has one durable identity even when provider retries or multiple tool rounds occur within it.

### Step

A step represents one provider turn, tool batch, compaction action, or other restartable unit of a run.
Steps make external side effects and recovery decisions explicit.

### Pending interaction

Permissions and forms are durable pending interactions between the runtime and clients.
They outlive one HTTP request and can be resolved, rejected, cancelled, expired, or recovered after restart according to the frozen behavior.

### Runtime resource

Processes, PTYs, MCP connections, provider streams, and plugin instances are live resources with durable references only where the public contract requires them.
Persisted metadata must never be treated as proof that a host resource still exists or is owned by TrueCode.

## Logical run states

The planned run state machine uses the following logical states:

| State | Meaning |
| --- | --- |
| `queued` | The request is durably accepted but has not started provider or tool work. |
| `starting` | Configuration, credentials, model, instructions, and context are being resolved. |
| `generating` | A provider request is active or being retried. |
| `awaiting_permission` | One or more requested side effects require a client decision. |
| `awaiting_form` | The agent is waiting for structured client input. |
| `executing_tools` | Approved tool work is active. |
| `compacting` | Context is being summarized or reduced. |
| `cancelling` | Cancellation is durable and owned resources are being stopped. |
| `completed` | The run reached its successful terminal state. |
| `cancelled` | The run reached its cancellation terminal state. |
| `failed` | The run reached a terminal error state. |
| `interrupted` | The service stopped before a nonterminal transition could complete and recovery policy must decide the next action. |

Transitions that appear externally are persisted before their events are published.
The conformance suite determines which intermediate states are exposed and which remain private implementation details.

## Command serialization

Each active session has one logical command serializer.
Commands include prompts, aborts, replies to permissions or forms, branch operations, revert operations, compaction requests, and session deletion.

The serializer prevents two mutations from independently assuming the same current state.
It does not require all external work to run while holding a mutex or database transaction.
Long work records durable intent, releases storage locks, and later applies a completion transition if the run still owns that work.

Idempotency behavior is characterized per endpoint.
Where the upstream API provides no idempotency key, TrueCode must still prevent duplicated internal completion after reconnect or retry.

## Durable transition and event ordering

State mutation and event production use a transactional outbox or equivalent atomic design.
Each durable event receives a monotonic sequence within its required ordering scope.

The intended sequence is:

1. Validate the command against current durable state.
2. Write entity changes and event records in one SQLite transaction.
3. Commit the transaction.
4. Publish committed events to live subscribers in sequence order.
5. Advance subscriber cursors or acknowledgements where the protocol provides them.

The reference contract decides whether ordering is global, per session, per connection, or a combination.
TrueCode must not impose a weaker order than clients observe from the frozen release.

Live delivery may coalesce internal provider deltas only if response and event transcripts remain compatible.
No event may be published for a rolled-back transition.

## Provider loop

The provider loop performs:

1. Resolve session, agent, model, provider, credentials, instructions, skills, and current configuration snapshot.
2. Build context from durable messages and accepted compaction state.
3. Start a provider attempt with a child cancellation scope.
4. Normalize provider events without losing provider-specific information needed by the protocol.
5. Persist externally visible message and part transitions in order.
6. Dispatch tool requests only after their call data is complete and validated.
7. Continue provider rounds until the reference terminal condition is reached.
8. Persist usage, finish reason, errors, and terminal state.

Retry decisions account for provider error class, retry hints, request progress, cancellation, and duplicate-side-effect risk.
The runtime does not retry a tool side effect merely because a provider connection failed afterward.

## Tool loop

Tool calls are validated against the selected tool schema and current runtime policy.
The permission decision occurs before starting the side effect.

For each tool call, the runtime records:

- Stable call identity
- Tool name and validated input
- Permission state and decision evidence
- Start and completion state
- Bounded output or a reference to retained output
- Error or exit information
- Cancellation and cleanup outcome

Independent read-only tools may run concurrently only when the frozen behavior permits it and deterministic result ordering is preserved.
Mutating tools use explicit conflict rules for filesystem paths, Git repositories, PTYs, and session state.

## Permissions and forms

Pending permissions and forms are created durably before clients are notified.
Replies are accepted once according to the characterized upstream conflict behavior.

The runtime must define behavior for:

- Multiple connected clients observing the same request
- Simultaneous replies
- Client disconnect before reply
- Session abort while waiting
- Service restart while waiting
- Expiry where the frozen behavior defines it
- Deletion of the owning session

Permission approval authorizes only the represented operation and scope.
It is not a general sandbox escape or a transferable capability for unrelated calls.

## Cancellation hierarchy

Cancellation is hierarchical:

```text
service
  session run
    provider attempt
    tool call
      subprocess tree
    MCP request
    plugin call
    compaction operation
```

Cancelling a parent requests cancellation of every owned child.
Children receive a bounded graceful-stop interval followed by platform-correct forced termination where applicable.

Cancellation has a durable state transition so restart does not resume work that was already cancelled.
The API response, final message state, emitted events, subprocess exit, and provider connection closure must match differential evidence.

## Client disconnect

An HTTP, SSE, or WebSocket disconnect does not automatically imply run cancellation.
The frozen behavior for each initiating route determines whether work continues, is detached, or is cancelled.

Live stream ownership is separate from run ownership.
Reconnect behavior must not duplicate durable message parts or terminal events.

## Interruption and restart recovery

At startup, recovery scans nonterminal durable work and classifies it by last committed transition.
Recovery never assumes that a prior in-memory task, provider stream, or ordinary subprocess survived.

Possible recovery actions are:

- Mark the work interrupted and expose the reference error behavior.
- Resume from a documented restartable boundary.
- Reconcile an externally durable operation using its stable identity.
- Reattach only to a verifiably owned persistent PTY or service resource.
- Complete cancellation that was committed before shutdown.

Provider generation cannot generally resume from an arbitrary stream offset.
Retries after interruption must avoid duplicating committed tool side effects and must follow observed upstream behavior.

Recovery decisions are deterministic from durable state plus verified live-resource identity.
Every decision emits auditable diagnostics without exposing secrets.

## Compaction

Compaction is a durable session operation rather than an in-place deletion of history.
The runtime records the source range, model and configuration identity where exposed, summary result, and activation transition.

A failed or cancelled compaction leaves the previously active context valid.
Compaction must not race with a provider turn in a way that creates a context assembled from incompatible snapshots.

Long-session conformance covers repeated compaction, restart during compaction, cancellation, provider failure, and reconnect event order.

## Inbox and queued work

If the frozen runtime accepts work while a session is active, queued items are durable and ordered.
The conformance inventory determines admission limits, replacement behavior, cancellation behavior, and whether queued items are exposed as inbox entities or session messages.

Production admission is bounded even if the public API does not expose the bound directly.
Overload must produce a stable response rather than unbounded memory growth.

## Error model

Internal errors retain a typed category, source, retryability, public mapping, and redaction policy.
Arbitrary strings are not used as the only error representation across subsystem boundaries.

Public errors match frozen status codes and response schemas.
Sensitive provider bodies, tokens, local paths, environment values, and process command details are redacted unless the contract explicitly and safely exposes them.

Panics are process defects, not routine control flow.
Task failures are observed and converted into owned runtime failure transitions where recovery is possible.

## Shutdown

Shutdown proceeds through explicit phases:

1. Stop accepting new external work.
2. Notify long-lived clients according to protocol behavior.
3. Commit cancellation or interruption intent for active durable work.
4. Cancel provider, MCP, plugin, tool, and PTY tasks.
5. Terminate owned subprocess trees and close transports.
6. Flush committed events and storage work within the shutdown deadline.
7. Release listeners, locks, and runtime ownership records.
8. Exit with a status that distinguishes clean shutdown from forced failure where the service interface exposes it.

Shutdown behavior is tested with active streams, pending permissions, slow tools, persistent PTYs, busy storage, and repeated signals.
