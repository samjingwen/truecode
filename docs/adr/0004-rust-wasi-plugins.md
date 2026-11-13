# ADR 0004: Rust-first WASI plugins

- Status: Accepted
- Date: 2026-11-13

## Context

OpenCode plugins use JavaScript and TypeScript runtime assumptions that would reintroduce a large dynamic runtime and broad host authority into TrueCode.
Literal ABI compatibility would conflict with the goal of a Rust-native service and capability-limited extensions.

## Decision

TrueCode supports plugins through a versioned WebAssembly component interface hosted by Wasmtime.
The first-party SDK is Rust-first.

Plugins begin without ambient authority and receive explicit capabilities through narrow host interfaces.
Capability grants bind to component identity and digest.
The host applies memory, time, output, concurrency, and cancellation limits.

OpenCode JavaScript and TypeScript plugin ABIs are unsupported.
TrueCode reports that incompatibility explicitly and does not evaluate those plugins.

## Consequences

Plugin authors must port extensions to the TrueCode SDK and component model.
Official clients can still use compatible server behavior, but OpenCode plugin packages cannot be reused directly.

WASI containment narrows guest authority but does not make granted host functions safe automatically.
The host interface remains security-critical and requires focused review and conformance tests.
