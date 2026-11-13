# TrueCode

TrueCode is a planned Rust reimplementation of the OpenCode server and agent runtime.
It is intended to provide the same published server API and functional runtime behavior with a distinct implementation identity, lower resource use, and tighter control over process and service behavior.

## Status

TrueCode is in its documentation and conformance-foundation milestone.
No Rust runtime implementation exists yet.
The protocol inventory, behavioral test strategy, security boundaries, and architectural decisions must be reviewed before implementation begins.

## Frozen compatibility target

The compatibility target is OpenCode `v2.2.7` at commit `fedc6f208aa2bd54c1019227214568083f6380ad`.
That reference was the newest published, non-draft, non-prerelease OpenCode product release at project kickoff on November 13, 2026.

The frozen OpenAPI document contains:

- 139 paths
- 170 HTTP operations
- 497 component schemas

See [UPSTREAM.md](UPSTREAM.md) for provenance, hashes, and the freeze policy.

## Scope

The intended parity surface includes:

- Published HTTP, SSE, and WebSocket behavior
- Sessions, messages, inbox, permissions, and forms
- Agent execution, tools, cancellation, interruption, recovery, and compaction
- Provider streaming, model normalization, model discovery, and OAuth
- Fresh SQLite persistence and service lifecycle
- Configuration, instruction, skill, and command discovery
- MCP clients, servers, transports, resources, prompts, tools, and authentication
- Filesystem, editing, search, shell, process, and Git tools
- Normal and persistent PTYs
- Git worktrees and VCS behavior
- Pairing and server authentication
- Rust-first plugins hosted as capability-limited WASI components

## Explicit exclusions

- TrueCode will not load OpenCode JavaScript or TypeScript plugins.
- TrueCode will not migrate or inspect old OpenCode or TrueCode state, sessions, databases, or credentials.
- TrueCode will not reimplement the OpenCode TUI, web client, or desktop client.
- The first release will not target Windows or x86_64.
- The first release targets macOS ARM64 and Linux AArch64.

Official OpenCode clients may be used unchanged as conformance consumers.
The project describes its goal as full server and functional runtime parity, not literal total parity.

## Design principles

1. Freeze one upstream release and do not chase its development branch.
2. Build a differential conformance harness before building the runtime.
3. Treat observable behavior as the contract, not route stubs alone.
4. Persist state transitions before publishing externally visible events.
5. Bound queues, streams, subprocesses, and resource ownership.
6. Keep tool permissions and generated-code isolation separate from plugin sandboxing.
7. Use fresh disposable namespaces for every conformance run.
8. Keep the official OpenCode runtime as the default in downstream integrations until TrueCode passes acceptance.

## Repository guide

- [UPSTREAM.md](UPSTREAM.md) records the frozen upstream reference.
- [docs/COMPATIBILITY.md](docs/COMPATIBILITY.md) defines parity and compatibility breaks.
- [docs/API_INVENTORY.md](docs/API_INVENTORY.md) inventories the frozen HTTP contract.
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) defines the planned Rust workspace and dependency boundaries.
- [docs/CONFORMANCE.md](docs/CONFORMANCE.md) defines differential testing and acceptance evidence.
- [docs/RUNTIME.md](docs/RUNTIME.md) defines runtime state and event semantics.
- [docs/STORAGE.md](docs/STORAGE.md) defines the fresh SQLite design constraints.
- [docs/SECURITY.md](docs/SECURITY.md) defines trust boundaries and non-claims.
- [docs/PLUGINS.md](docs/PLUGINS.md) defines the Rust and WASI plugin model.
- [docs/PLATFORMS.md](docs/PLATFORMS.md) defines the supported release matrix.
- [docs/ROADMAP.md](docs/ROADMAP.md) defines delivery milestones and exit criteria.
- [docs/adr](docs/adr) contains accepted architecture decisions.

## Verification and delivery

GitHub-hosted CI is intentionally disabled.
Changes must be verified locally against the exact proposed commit, independently reviewed, and merged through the protected `master` branch.
Native release verification is required on macOS ARM64 and Linux AArch64 before making platform claims.

## License

TrueCode is licensed under the [MIT License](LICENSE).
The frozen OpenCode materials retain their original MIT notice under `upstream/opencode-v2.2.7/`.
