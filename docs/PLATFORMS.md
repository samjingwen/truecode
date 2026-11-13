# Platform support

## First-release matrix

TrueCode's first compatibility release targets 64-bit ARM on macOS and Linux.
ARM64 and AArch64 refer to the same processor architecture, but each operating system and libc requires a separate artifact and native verification.

| Platform | Rust target | Support intent |
| --- | --- | --- |
| macOS ARM64 | `aarch64-apple-darwin` | Required first-release target |
| Linux AArch64 with glibc | `aarch64-unknown-linux-gnu` | Required first-release target |
| Linux AArch64 with musl | `aarch64-unknown-linux-musl` | Release only after separate native validation |

Windows and all x86_64 targets are outside the first-release scope.
Cross-compiling an unsupported target does not create a support claim.

## Native verification requirement

GitHub-hosted CI is disabled.
Release verification runs through documented local commands on native target systems.

Each native report records:

- TrueCode commit and build profile
- Rust toolchain and target triple
- Operating-system and kernel version
- CPU architecture
- libc and dynamic-linker details on Linux
- Git and default shell versions
- SQLite runtime version
- Conformance fixture and upstream artifact digests
- Tests run, results, skips, and limitations

macOS results do not establish Linux behavior.
GNU/Linux results do not establish musl behavior.

## Platform-sensitive areas

### Process lifecycle

The process runner must validate process groups, sessions, signals, descendant cleanup, exit status, executable lookup, environment handling, and cancellation on each target.
It must not assume that killing an immediate child terminates its descendants.

### PTYs

PTY allocation, controlling-terminal behavior, resize, signal delivery, shell startup, output encoding, detach, and teardown require native tests.
Persistent PTY identity must resist PID reuse after service restart.

### Filesystems

Tests cover case sensitivity where available, Unicode paths, symlinks, permissions, executable bits, atomic rename, file watching, long paths, temporary files, and repository boundary checks.
Code cannot assume macOS's common default filesystem behavior is shared by Linux.

### Service lifecycle

macOS service integration and Linux service integration use platform-specific adapters behind one service CLI contract.
Installation, startup, readiness, status, logs, shutdown, restart, stale ownership, and uninstall require native tests.

The exact macOS launchd and Linux systemd packaging behavior requires dedicated ADRs after the frozen upstream lifecycle inventory is complete.

### Credential storage

macOS Keychain and Linux credential facilities differ substantially, especially on headless Linux.
The credential backend requires a separate accepted decision before implementation.
No platform silently falls back to plaintext secret storage.

### Paths and locations

TrueCode uses its own namespace and never falls back to OpenCode state directories.
Configuration, data, cache, runtime, sockets, logs, and credentials have platform-aware defaults plus explicit test overrides.

Exact location precedence must be characterized against the frozen API and documented before implementation.
Conformance always supplies disposable roots so it cannot discover user state.

### Networking

Loopback binding, dual-stack behavior, ephemeral ports, socket reuse, proxy environment, DNS, TLS roots, and shutdown differ by platform and distribution.
Network conformance records these host facts and avoids assuming one Linux distribution represents all deployments.

### Git and shells

The service records and tests the Git and shell versions used for platform conformance.
Shell tools must preserve the frozen behavior without assuming Bash is the login shell on macOS or every Linux system.

## Artifact policy

Artifacts are named with the TrueCode version, target triple, and source commit association.
They report TrueCode's implementation version separately from OpenCode `v2.2.7` protocol compatibility.

Release artifacts are built from a clean merged `master` revision.
Each artifact receives a SHA-256 digest and a manifest containing source commit, toolchain, target, and build command.

TrueCode does not reuse OpenCode artifact names and does not install an `opencode` executable.

## Unsupported platform behavior

Unsupported targets may be used by developers at their own risk, but they do not receive compatibility claims or release artifacts.
Platform-specific fixes must preserve supported-target behavior and include native evidence for every affected supported target.

Adding a target requires:

1. An ADR defining the support tier.
2. A native verification host.
3. Process, PTY, filesystem, service, SQLite, provider, MCP, and plugin conformance.
4. Packaging and artifact provenance.
5. Updates to compatibility and release documentation.
