# Security policy

## Project status

TrueCode is pre-release and does not yet provide a production runtime.
No version is currently supported for production security updates.

## Reporting a vulnerability

Use GitHub's private security advisory reporting for this repository when it is available.
Do not open a public issue for a vulnerability that could expose credentials, execute commands, cross filesystem boundaries, bypass permissions, corrupt state, or compromise plugin isolation.

Include:

- The affected commit or version
- The operating system and architecture
- A concise impact description
- Reproduction steps using disposable data
- Relevant logs with secrets removed
- Any known mitigations

Do not include real provider tokens, OAuth credentials, private repository contents, or historical OpenCode or TrueCode data.

## Scope

Security-sensitive areas include:

- Server authentication and pairing
- Provider and OAuth credentials
- Filesystem and shell tools
- Process trees and PTYs
- Git and worktree operations
- MCP transports and authentication
- SQLite durability and state isolation
- WASI plugin capabilities and host functions
- Configuration and instruction discovery
- Service installation and lifecycle

See [docs/SECURITY.md](docs/SECURITY.md) for the architecture, trust boundaries, and explicit non-claims.
