# TrueCode agent instructions

These instructions apply to the repository root and every directory below it.

## Product identity

- TrueCode is an independent Rust reimplementation of the OpenCode server and runtime.
- Use `truecode` for the binary, service, version, and fresh runtime namespace.
- Do not present TrueCode as the official OpenCode binary or as an OpenCode-maintained project.
- Describe the compatibility goal as full server and functional runtime parity, not literal total parity.

## Current milestone

- The repository is in a documentation-first and conformance-first milestone.
- Do not add runtime implementation code until the documentation baseline has been reviewed and the conformance harness work has been authorized.
- Keep implementation changes separate from documentation review unless the user explicitly expands the task.
- Treat unresolved behavior as an inventory or conformance question rather than silently choosing an implementation.

## Frozen upstream reference

- Target OpenCode `v2.2.7` at commit `fedc6f208aa2bd54c1019227214568083f6380ad`.
- Do not chase the OpenCode development branch or silently update the target release.
- Treat `upstream/opencode-v2.2.7/openapi.json` as immutable generated source evidence.
- Regenerate derived protocol code and manifests with their owning tools rather than editing generated output.
- Preserve the upstream MIT license and attribution.

## Compatibility scope

- Cover the complete pinned server API and functional runtime behavior, including the agent loop, providers, tools, storage, OAuth, MCP, PTYs, worktrees, VCS, configuration, lifecycle, and streaming behavior.
- Use official clients unchanged as conformance consumers where practical.
- Do not rewrite or fork the OpenCode TUI, web client, or desktop client.
- Do not implement JavaScript or TypeScript plugin ABI compatibility.
- Build plugins as Rust-first, capability-limited WASI components.
- Keep Windows and x86_64 outside the first release scope.
- Support macOS ARM64 and Linux AArch64 with separate native artifacts and verification.

## Runtime state and data safety

- Start from fresh TrueCode runtime state.
- Never inspect, import, migrate, or modify old OpenCode databases, sessions, retained data, or provider credentials.
- Never inspect or reuse previous TrueCode runtime state or credentials.
- Use newly provided dedicated credentials only for approved live provider and OAuth conformance tests.
- Run tests in owned disposable config, data, cache, home, repository, and credential namespaces.
- Never point destructive or mutation tests at a developer's normal home directory or working repositories.

## Architecture requirements

- Keep wire DTOs and the route manifest in a protocol crate generated from the frozen OpenAPI document.
- Use Axum and Tower for HTTP serving, middleware, limits, streaming, and graceful shutdown unless a later ADR replaces them.
- Model sessions and agent execution as explicit durable state machines.
- Persist externally visible state transitions transactionally before publishing their events.
- Use fresh SQLite storage with repository-owned append-only migrations after the initial baseline.
- Keep provider-specific protocols behind a normalized streaming interface.
- Use bounded channels, bounded process output, explicit cancellation, and owned task lifetimes.
- Isolate PTY, process-group, and signal behavior behind platform modules with native tests.
- Prefer the system Git executable for behavior where matching Git semantics matters.
- Keep the plugin host and plugin SDK separate from the core runtime.

## Protocol and conformance standards

- Build the differential conformance harness before broad runtime implementation.
- Drive identical deterministic scenarios against the frozen upstream binary and TrueCode.
- Compare status codes, headers, response bodies, SSE frames, event order, errors, cancellation, reconnects, and recovery.
- Canonicalize only proven nondeterministic values such as temporary paths, ports, timestamps, and generated identifiers.
- Never canonicalize event order, missing fields, error categories, or state transitions to hide incompatibility.
- Treat route presence without matching behavior as incomplete.
- Record every accepted deviation in `docs/COMPATIBILITY.md`.

## Security requirements

- Treat provider responses, MCP peers, plugins, repository files, tool output, and client input as untrusted.
- Enforce permissions at the tool-dispatch boundary before side effects begin.
- Do not claim that Rust makes host tool execution sandboxed.
- Keep process permissions, generated-code isolation, and WASI plugin capabilities as separate security boundaries.
- Keep secrets out of Git, logs, diagnostics, snapshots, and rendered command output.
- Apply explicit time, memory, output, concurrency, and cancellation limits to plugins and subprocesses.
- Reject path traversal and account for symlink races when enforcing filesystem boundaries.

## Engineering standards

- Use stable Rust and pin the toolchain before implementation begins.
- Keep configuration strict and fail on unknown fields unless upstream compatibility requires otherwise.
- Keep production queues and caches bounded and observable.
- Avoid `unsafe` Rust unless a platform boundary requires it.
- Isolate approved `unsafe` code, document its invariants with `SAFETY` comments, and cover the boundary with focused tests.
- Add append-only database migrations after the baseline schema is accepted.
- Reproduce bugs through the end-user API or client path before fixing them.
- Test real process, PTY, filesystem, and Git behavior rather than replacing all boundaries with mocks.
- Never manually edit `CHANGELOG.md` or files marked as generated.
- Preserve unrelated user work and private local material.

## Verification standards

- GitHub-hosted CI is intentionally disabled.
- Provide fast documented local quality entrypoints for formatting, static analysis, unit tests, integration tests, and conformance suites.
- Run relevant checks against the exact commit proposed for merge.
- Record commands, results, platform coverage, and limitations in the pull request.
- A missing native platform result must be reported and must not be represented as passing coverage.
- Measure memory, readiness, API latency, and soak behavior, but do not treat numerical performance thresholds as release gates.
- Treat unbounded resource growth, deadlock, corruption, and loss of cancellation as correctness defects.

## Documentation standards

- Keep each complete sentence on its own physical line in long Markdown files.
- Update the relevant ADR when changing an accepted architectural decision.
- Keep commands, paths, target triples, version pins, and compatibility claims consistent across documents.
- Mark hypotheses and unresolved behavior explicitly.
- Link claims about the frozen contract to checked-in source evidence or reproducible inventory commands.
