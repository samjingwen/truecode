# Storage

## Scope

TrueCode uses a fresh SQLite database for durable server and runtime state.
The physical schema is native to TrueCode and is not an OpenCode database clone.

No old OpenCode or TrueCode database may be opened, inspected, imported, attached, or migrated as part of this project.
Conformance compares public logical behavior, not private table layouts.

## Storage principles

- Persist externally visible transitions before publishing their events.
- Keep transactions short and free of provider, network, plugin, process, and PTY waits.
- Enforce invariants with schema constraints in addition to application checks.
- Use append-only migrations after the baseline schema is accepted.
- Make recovery deterministic from committed state.
- Keep secrets out of ordinary diagnostic queries and snapshots.
- Bound database work admission and surface overload.
- Treat SQLite busy, corruption, disk-full, and I/O errors as explicit runtime conditions.

## Database ownership

The service owns one database per TrueCode runtime namespace.
Tests create a database only inside their disposable namespace.

The planned Rust boundary uses `rusqlite` or an equivalently direct SQLite binding behind a bounded blocking executor.
The exact crate and pooling strategy require a dependency ADR before implementation.

SQLite writes are serialized by SQLite and coordinated through a bounded TrueCode write path.
Read concurrency is permitted only when it does not violate snapshot and ordering expectations.
HTTP handlers never retain a transaction while waiting for runtime work.

## Connection configuration

The baseline configuration will enable:

- Foreign key enforcement on every connection
- Write-ahead logging where supported by the target filesystem
- An explicit busy timeout
- Bounded connection count
- Transaction modes selected by operation semantics
- Integrity and migration checks during startup

Durability settings must be documented and validated under crash tests before release.
The service must detect filesystems or modes that cannot safely support the configured journal behavior.

## Conceptual data model

The initial schema is expected to represent the following durable concepts:

| Concept | Durable responsibility |
| --- | --- |
| Runtime metadata | Schema version, implementation version, protocol target, namespace identity, and clean-shutdown state. |
| Projects and locations | Stable project identity, roots, VCS association, and discovery metadata required by the API. |
| Sessions | Session metadata, parentage, sharing, summaries, lifecycle state, and ordering information. |
| Messages and parts | Ordered user, assistant, tool, reasoning, attachment, usage, and error content required by the protocol. |
| Runs and steps | Durable execution, retry, cancellation, interruption, and recovery boundaries. |
| Pending interactions | Permissions and forms with ownership, state, reply, and terminal outcome. |
| Tool calls | Validated input, approval, start, completion, bounded output metadata, and error state. |
| Compactions | Source range, result, activation, cancellation, and failure state. |
| Event outbox | Ordered committed events awaiting or eligible for live publication and replay. |
| MCP metadata | Configured peers, negotiated capabilities, authentication state references, and status required across restart. |
| Persistent PTYs | Durable protocol identity and verifiable live-resource metadata without assuming process survival. |
| Plugin metadata | Installed component identity, interface version, digest, declared capabilities, grants, and status. |
| Service metadata | Listener identity, startup generation, lifecycle state, and stale-owner detection data. |

This table is conceptual and does not pre-approve table names or denormalization choices.
The baseline schema requires a separate reviewed design before migration files are created.

## Identifiers and ordering

Identifiers exposed through the API must match the frozen format and validation behavior.
Internal surrogate keys may differ but cannot leak through wire DTOs.

Ordered child entities use explicit stable sequence values.
Creation timestamps are not used as the sole ordering mechanism because equal or adjusted clocks cannot establish total order.

Durable events use an explicit monotonic sequence in each required ordering scope.
Sequence allocation occurs in the same transaction as the represented state change.

## Transaction boundaries

A mutation transaction may include:

- Validation against current durable state
- Entity inserts and updates
- Invariant checks
- Event outbox inserts
- Idempotency or ownership records

A mutation transaction must not include:

- Provider requests or stream reads
- MCP calls
- Plugin execution
- Subprocess or PTY waits
- Filesystem traversal outside narrowly required atomic metadata work
- Sleep or retry backoff

External work uses durable intent and completion transitions.
Late completion is rejected when its run or step no longer owns the operation.

## Migrations

The initial baseline creates an empty TrueCode database.
Every later schema change uses a new immutable migration.

Migrations must be:

- Transactional where SQLite permits
- Ordered and uniquely identified
- Safe to rerun only through the migration system's applied-version tracking
- Covered by empty-database and prior-version upgrade tests
- Paired with application compatibility and recovery notes

Applied migrations are never edited.
Development convenience is not a reason to rewrite migration history after the baseline is shared.

Database downgrade is not assumed.
Binary rollback does not imply schema rollback, and release documentation must state compatible rollback boundaries.

## Credentials and secrets

Provider and OAuth secrets are accessed through a dedicated credential-store interface.
They are not stored in ordinary session, event, or configuration tables and never appear in conformance snapshots.

The final credential backend for macOS and headless Linux remains an explicit pre-implementation decision.
It must address native key stores, unavailable desktop secret services, file permissions, noninteractive service startup, revocation, backup behavior, and recovery.
No implementation may silently fall back to plaintext secret storage.

Credential record identifiers may be durable so runtime entities can refer to secrets without embedding them.

## Event outbox retention

Outbox records remain until their replay and retention obligations are satisfied.
Live subscriber success alone is not proof that an event can be deleted.

Retention and compaction policy must preserve reconnect behavior established by conformance tests.
Pruning runs in bounded batches and cannot block foreground writes for an unbounded interval.

## Large content

The database does not accumulate unbounded process, PTY, provider, or tool output.
The frozen contract determines truncation and retained content behavior.

If large content is stored outside SQLite, the database owns its digest, length, path-independent identity, lifecycle, and deletion state.
External content is written atomically before a transaction makes it reachable.
Cleanup detects both unreachable files and missing referenced files without deleting unknown user files.

## Backup and recovery

Fresh-state policy does not remove the need for TrueCode-native backup once users create new state.
Backup uses SQLite's supported online backup or snapshot mechanisms rather than copying a live database and journal independently.

Startup recovery checks:

- Migration state
- Database integrity at an appropriate cost level
- Last clean shutdown marker
- Nonterminal runs and steps
- Pending cancellation
- Stale service ownership
- Persistent PTY identity
- Reachable external content

Automatic destructive repair is prohibited.
Corruption produces a clear diagnostic and preserves evidence for an explicitly authorized recovery procedure.

## Testing

Storage tests cover:

- Fresh database creation
- Every migration path
- Foreign key and uniqueness invariants
- Concurrent readers and serialized writers
- Busy timeout and lock contention
- Cancellation before and after commit
- Process termination at selected transition boundaries
- Disk-full and I/O failure simulation in disposable storage
- Outbox ordering and replay
- Pruning and compaction bounds
- Corruption detection without destructive repair

Tests must never discover or open databases outside their disposable root.
