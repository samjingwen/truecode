# ADR 0006: ARM64 platform scope

- Status: Accepted
- Date: 2026-11-13

## Context

PTYs, signals, process groups, service managers, filesystems, credential stores, and packaging differ across operating systems.
Supporting every OpenCode release target in the first TrueCode release would dilute native behavioral verification.

ARM64 and AArch64 name the same 64-bit ARM architecture, but macOS and Linux artifacts are not interchangeable.

## Decision

The first TrueCode release targets:

- macOS `aarch64-apple-darwin`
- GNU/Linux `aarch64-unknown-linux-gnu`
- musl Linux `aarch64-unknown-linux-musl` only after separate native validation

Windows and x86_64 are excluded from the first release.
Supported platform claims require native conformance evidence.
Cross-compilation alone is insufficient.

## Consequences

The implementation keeps platform behavior behind explicit modules even though the initial CPU architecture is shared.
Users on excluded targets must connect to a supported remote TrueCode server or continue using the official runtime.

Adding a target requires a superseding or extending ADR, native verification capacity, packaging, and full platform-sensitive conformance.
