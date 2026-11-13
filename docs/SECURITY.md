# Security architecture

## Security objective

TrueCode runs an agent that can communicate with remote systems and perform host operations.
The security design reduces unintended authority, makes side effects reviewable, contains plugins, protects credentials, and provides reliable cancellation.

TrueCode does not claim that implementation in Rust makes agent-selected host commands safe or sandboxed.
Memory safety, permission policy, process isolation, and plugin capabilities are separate concerns.

## Trust boundaries

The following inputs are untrusted:

- HTTP, SSE, and WebSocket clients
- Pairing and authentication material presented by clients
- User prompts and attachments
- Repository files, symlinks, Git metadata, and discovered instructions
- Provider responses and streamed tool calls
- MCP peers and their tools, prompts, resources, and authentication challenges
- Tool and subprocess output
- PTY input and output
- Plugin components and plugin-provided data
- Downloaded release artifacts and dependency packages until verified

The local operator and explicitly granted credentials are trusted only within their documented scope.

## Server boundary

The server binds to loopback by default unless an operator explicitly selects another address.
Non-loopback binding requires authentication and a clear diagnostic of the exposure.

The server applies:

- Request body and header limits
- Timeouts appropriate to non-streaming operations
- Connection and stream admission limits
- Strict content-type handling
- Authentication before protected resource lookup
- Origin and cross-origin policy matching the frozen contract without broad wildcard exposure
- Redacted structured request tracing

Pairing codes and bearer material are short-lived or revocable according to the characterized reference behavior.
Authentication comparisons avoid avoidable timing leakage.

## Authorization and permissions

API authentication, agent tool permissions, plugin capabilities, and operating-system permissions are distinct layers.
Passing one layer never implies approval at another.

Tool permissions are evaluated at the dispatch boundary before side effects begin.
A permission decision binds to the exact tool, validated input, resolved target, session, and scope represented to the user.
Material changes require a new decision.

Simultaneous permission replies resolve once.
Later replies receive the frozen conflict behavior and cannot overwrite the accepted decision.

## Filesystem safety

Filesystem policy uses normalized paths only as one input.
It also accounts for symlinks, hard links where relevant, mount boundaries, case behavior, Unicode normalization, and replacement between check and use.

Operations prefer descriptor-relative APIs and no-follow options where platform support permits them.
Recursive operations remain within the approved root and do not traverse newly introduced links.

Workspace approval does not authorize reading home directories, credential stores, SSH keys, environment files, browser data, or unrelated repositories.
Tests use synthetic secrets to prove denials without reading real private files.

## Process and shell safety

All spawned commands pass through one bounded process runner.
The runner owns:

- Executable and argument construction
- Working directory validation
- Environment allowlist and overrides
- Standard input policy
- Standard output and error budgets
- Wall-clock deadline
- Process-group or session creation
- Cancellation escalation
- Descendant cleanup
- Exit and signal reporting

Shell execution is used only when shell semantics are part of the tool contract.
Structured process invocation does not concatenate untrusted input into a shell command.

Cancellation targets the owned process tree rather than only the immediate child.
PID reuse is accounted for when reconciling persistent resources.

Host tool execution is not a sandbox.
Stronger generated-code isolation requires a separate, explicit execution boundary and cannot be inferred from permission prompts.

## Provider and OAuth security

Credentials are injected through the dedicated credential-store boundary.
They are excluded from logs, traces, errors, event payloads, crash reports, fixtures, and snapshots.

OAuth state and PKCE verifiers are unpredictable, single-use, bound to the initiating flow, and expired after a bounded interval.
Callback listeners bind narrowly and validate all state before exchanging a code.

Refresh operations serialize per credential to avoid token races.
An invalid or revoked token does not cause raw provider responses containing secrets to be surfaced.

Live conformance uses only newly provisioned dedicated test credentials supplied for this project.
Old OpenCode and TrueCode credentials are never inspected.

## MCP security

MCP configuration is untrusted even when discovered inside a repository.
Starting a local MCP process or connecting to a remote endpoint requires the appropriate policy decision.

The MCP boundary enforces:

- Transport allowlists
- Endpoint and redirect validation
- Authentication scoping
- Initialization and capability checks
- Input and result size limits
- Timeouts and cancellation
- Tool schema validation
- Prompt and resource content limits
- Connection and restart limits

MCP-provided text is data and cannot silently become higher-priority system policy.

## Plugin security

Plugins are WASI components hosted by Wasmtime.
They receive no ambient filesystem, network, environment, clock, random, subprocess, credential, MCP, provider, or tool access.

Capabilities are declared in a manifest, approved by policy, and materialized as narrow host interfaces.
The host enforces memory, table, instance, fuel or epoch, wall-time, output, and concurrency limits.

Capabilities are checked on every host call.
A plugin cannot delegate a capability it does not hold.
Plugin identity includes a content digest so grants do not silently transfer to changed bytes.

WASI containment does not protect against flaws in granted host functions.
Host functions remain part of the trusted computing base and require focused review and tests.

## PTY security

PTYs can run interactive shells and therefore carry the authority of their owning service account.
Creation requires the same permission scrutiny as shell execution.

Attach tokens or identifiers must not permit cross-session attachment.
Persistent PTY recovery verifies process ownership and a non-reusable instance identity rather than trusting a PID alone.

Output is bounded for retention even when live delivery continues.
Detach does not abandon ownership or cleanup responsibility.

## Git and worktree security

Git repositories are untrusted input.
Commands disable or control hooks, external filters, pagers, credential helpers, SSH commands, protocol helpers, and configuration includes where they could execute code outside the intended contract.

Operations that intentionally invoke user-configured Git behavior require explicit documentation and conformance evidence.
Worktree paths are validated against approved roots before creation or removal.

TrueCode never force-cleans, resets, checks out over, or removes unrelated user work to recover from a Git error.

## Database security

SQLite files and external content use restrictive permissions.
Database paths cannot resolve into OpenCode state or a prior TrueCode namespace during tests or first-run setup.

SQL uses bound parameters for values.
Dynamic identifiers are selected from fixed application definitions rather than untrusted strings.

Database diagnostics redact message content and credentials by default.
Automatic destructive repair is prohibited.

## Network egress

Provider, OAuth, MCP, plugin-host, and update-related egress are separate policy categories.
Redirects are bounded and revalidated.
Proxy configuration is explicit and secret-bearing proxy URLs are redacted.

Plugins have no network egress unless a specific host capability mediates it.
Host tools may create network traffic through their own behavior, which is one reason tool permission is not equivalent to a sandbox.

## Resource exhaustion

Every externally influenced collection, queue, stream, cache, output buffer, retry loop, and concurrency pool has a bound or eviction policy.
Limits cover:

- Request and attachment size
- Concurrent clients and streams
- Session commands and inbox entries
- Provider retries and buffered deltas
- Tool calls and subprocesses
- Captured process and PTY output
- MCP peers and in-flight calls
- Plugin instances and calls
- SQLite work queue
- Event replay windows

Overload returns stable errors or disconnect behavior rather than unbounded allocation.

## Logging and diagnostics

Logs use structured fields and stable error categories.
The default policy excludes prompt bodies, file contents, message content, tool output, provider payloads, authorization headers, cookies, tokens, environment values, and command input that may contain secrets.

Debug modes cannot silently disable redaction.
Diagnostic bundles enumerate included material before creation and require explicit operator action.

## Supply chain

Direct dependencies are pinned through Cargo's lockfile for released applications.
New major dependencies require architecture review covering maintenance, vulnerabilities, transitive surface, build scripts, native code, and platform support.

Upstream OpenCode artifacts are verified against recorded SHA-256 digests.
Plugin component identity is digest-based.
Release artifacts are built from a clean merged revision and associated with their source commit.

## Unsafe Rust

`unsafe` is avoided except where an operating-system, FFI, or performance boundary requires it.
Approved `unsafe` code is isolated in small modules, states each invariant in a `SAFETY` comment, and has tests focused on invalid inputs, cancellation, and lifecycle edges.

No crate receives a blanket exemption from review because its purpose is platform integration.

## Security testing

Security verification includes:

- Protocol and parser fuzzing
- Property tests for state machines and canonicalization
- Path traversal and symlink-race tests
- Authentication and pairing abuse cases
- OAuth state, redirect, expiry, and refresh races
- MCP malformed and oversized payloads
- Subprocess descendant and cancellation tests
- PTY cross-session and stale-PID tests
- Plugin capability and resource-exhaustion tests
- SQLite fault injection and invariant checks
- Secret-scanning of logs, fixtures, reports, and repository content

## Security non-claims

TrueCode does not initially claim:

- General sandboxing of shell or filesystem tools
- Safe execution of arbitrary generated code
- Protection from a compromised operating-system account
- JavaScript or TypeScript plugin isolation
- Compatibility with unreviewed OpenCode releases
- Security validation on unsupported operating systems or architectures

These limits must remain visible in user and operator documentation.
