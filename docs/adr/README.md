# Architecture decision records

Architecture decision records capture accepted choices that constrain TrueCode implementation and compatibility claims.

| ADR | Decision | Status |
| --- | --- | --- |
| [0001](0001-freeze-opencode-v2.2.7.md) | Freeze OpenCode `v2.2.7` as the compatibility reference. | Accepted |
| [0002](0002-rust-runtime-architecture.md) | Use a bounded, state-machine-oriented Rust service architecture. | Accepted |
| [0003](0003-fresh-runtime-state.md) | Start with fresh TrueCode state and prohibit historical-state migration or inspection. | Accepted |
| [0004](0004-rust-wasi-plugins.md) | Support Rust-first WASI component plugins and exclude OpenCode JavaScript and TypeScript plugin ABIs. | Accepted |
| [0005](0005-local-verification-delivery.md) | Use local exact-head verification and protected pull requests without GitHub-hosted CI. | Accepted |
| [0006](0006-arm64-platform-scope.md) | Target macOS ARM64 and Linux AArch64 for the first release. | Accepted |

Changing an accepted decision requires a new superseding ADR rather than rewriting its history.
