# Plugin model

## Decision

TrueCode plugins are Rust-first components using the WebAssembly component model and WASI interfaces hosted by Wasmtime.
TrueCode does not load or emulate OpenCode JavaScript or TypeScript plugin ABIs.

This is an intentional compatibility break recorded in [COMPATIBILITY.md](COMPATIBILITY.md) and [ADR 0004](adr/0004-rust-wasi-plugins.md).

## Goals

The plugin system must provide:

- A stable versioned component interface
- A first-party Rust SDK
- Explicit capability declarations and grants
- Deterministic lifecycle and failure behavior
- Memory, time, output, and concurrency limits
- Test tooling that does not require a full service
- Clear compatibility errors for unsupported plugin formats
- Independent plugin identity based on component bytes

The plugin system is not a route for bypassing tool permissions or host security boundaries.

## Component interface

The public plugin ABI is defined in versioned WIT packages.
WIT definitions are the source of truth for component imports, exports, records, variants, resources, and errors.

The initial interface families are planned to cover:

- Plugin metadata and initialization
- Configuration validation
- Lifecycle notifications
- Agent and command contribution
- Skill and instruction contribution
- Tool definition and invocation
- Event observation with redacted payload types
- Provider contribution where safe and explicitly versioned
- Structured diagnostics

An interface is added only when its lifecycle, capability, compatibility, and failure behavior are defined.
Raw host pointers, arbitrary JSON as the only contract, and unstable Rust ABI types are not exposed.

## Manifest

Each plugin includes a manifest containing:

- Plugin name and semantic version
- Component digest
- Required TrueCode plugin-interface range
- Exported contributions
- Requested capabilities
- Configuration schema
- Resource limit requests
- Human-readable description and provenance

The host validates the manifest before instantiation.
Unknown required fields, unsupported interface versions, and undeclared exports fail clearly.

Capability grants bind to plugin identity, including the component digest.
Replacing component bytes requires grant review rather than inheriting authority by name alone.

## Capability model

Plugins start with no ambient authority.
Potential capabilities include narrow forms of:

- Read access to approved configuration values
- Read access to selected session metadata
- Subscription to selected redacted event classes
- Registration of named tools or commands
- Invocation of permission-mediated host tools
- Read or write access to explicitly granted virtual filesystem roots
- Outbound HTTP through an allowlisted host client
- Access to monotonic time or wall time
- Access to random bytes
- Plugin-scoped persistent key-value state

No capability implies another capability.
Network access does not imply credential access.
Filesystem access does not imply process execution.
Tool registration does not imply automatic approval when the tool is invoked.

Host calls validate capability, current grant, input size, deadline, and cancellation on every invocation.

## WASI exposure

The host does not enable broad WASI inheritance from the service process.
By default, a component receives no host environment, inherited standard streams, host filesystem, network sockets, subprocess access, or service credentials.

Preopened directories are avoided in favor of narrow resource handles where practical.
When a preopen is required, it is rooted to a dedicated plugin directory with explicit read or write mode.

Clocks and randomness are exposed only through declared interfaces because they affect determinism and replay.

## Resource limits

Every plugin instance has configured limits for:

- Linear memory
- Tables and component resources
- Fuel or epoch interruption
- Wall-clock execution time
- Host calls
- Input and output size
- Concurrent invocations
- Persistent state size
- Log and diagnostic volume

Limits have bounded defaults.
A plugin request for a larger limit is not self-authorizing.

Cancellation interrupts guest execution and propagates to cancellable host calls.
A plugin that exceeds a limit receives a typed failure and cannot leave an unowned host operation running.

## Lifecycle

The logical lifecycle is:

1. Discover the manifest and component.
2. Verify path, format, digest, interface compatibility, and policy.
3. Resolve configuration and capability grants.
4. Compile or load a cache entry keyed by runtime and component identity.
5. Instantiate within resource limits.
6. Run initialization with a deadline.
7. Register validated contributions atomically.
8. Serve invocations under cancellation and concurrency controls.
9. Stop admission, cancel active work, run bounded shutdown where supported, and release resources.

A failed initialization publishes no partial contribution set.
A plugin crash fails its invocation and updates plugin status without crashing the service.
Restart and disable behavior must be bounded to prevent crash loops.

## Plugin state

Plugin-scoped persistent state is namespaced by stable plugin identity and interface version.
It is accessed through typed host operations rather than direct SQLite access.

Quotas apply to key count, key size, value size, and total storage.
State migration is initiated explicitly by a compatible plugin version and remains transactional.

Plugins cannot query sessions, credentials, or other plugin state through the storage interface.

## Tool contributions

A plugin-defined tool provides a stable name, description, input schema, output contract, and capability requirements.
Registration rejects collisions according to a documented deterministic precedence rule.

Tool invocation remains inside the normal runtime tool-call and permission lifecycle.
Plugin code cannot mark its own side effects as approved.

Host tools invoked by a plugin receive the plugin identity in audit and permission context.
The plugin cannot expand or rewrite an approved request after approval.

## Provider contributions

Provider plugins have a larger security and compatibility surface than ordinary tools.
The first implementation may defer provider contribution until core provider parity is stable.

If enabled, provider plugins must use typed streaming events, a credential reference interface, redacted diagnostics, explicit network capability, and the same cancellation and usage rules as built-in providers.
They never receive a dump of the service credential store.

## Rust SDK

`truecode-plugins-sdk` will provide:

- Generated WIT bindings
- Safe Rust traits and types over component exports
- Manifest construction and validation
- Error and diagnostic helpers
- Test fixtures and a lightweight test host
- Capability mocks that default to denial
- Component packaging and digest commands
- Compatibility checking against supported interface versions

SDK convenience APIs cannot expose authority absent from the WIT contract and host grant.

## Distribution and provenance

The first release supports local installation from an explicitly selected component and manifest.
Remote registries, automatic updates, and trust catalogs require separate decisions.

Installed metadata records source, digest, install time, interface version, and grant state.
TrueCode never executes a changed component under grants bound to an older digest.

Signing may supplement digest identity in a later release, but a signature does not grant capabilities automatically.

## Unsupported OpenCode plugins

When configuration references an OpenCode JavaScript or TypeScript plugin, TrueCode returns or records an explicit unsupported-plugin compatibility error.
It must not silently ignore the plugin if doing so would make behavior appear complete.

The error identifies the unsupported ABI category and points to the TrueCode Rust and WASI plugin documentation.
It does not execute package managers, resolve JavaScript dependencies, or evaluate plugin code.

## Conformance

Plugin conformance covers:

- Discovery and precedence
- Manifest validation
- Interface negotiation
- Capability denial and grant
- Tool registration and invocation
- Event ordering
- Cancellation and deadline
- Memory and output limits
- Guest trap and host-call failure
- Service shutdown and restart
- State isolation and quota
- Digest change and grant invalidation
- Explicit JavaScript and TypeScript plugin rejection

The accepted plugin ABI break is tested so future changes cannot accidentally introduce partial, unsafe compatibility.
