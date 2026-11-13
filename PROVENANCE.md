# Frozen reference provenance

## Selected release

TrueCode targets the following immutable OpenCode release:

| Field | Value |
| --- | --- |
| Repository | `https://github.com/anomalyco/opencode` |
| Tag | `v2.2.7` |
| Commit | `fedc6f208aa2bd54c1019227214568083f6380ad` |
| Release status | Published, non-draft, non-prerelease |
| Commit time | `2026-11-10T07:52:42Z` |
| Published time | `2026-11-10T10:37:15Z` |
| Project kickoff | `2026-11-13` |
| Source license | MIT |

GitHub's `releases/latest` endpoint still identified `v1.19.5` at kickoff.
That marker was not used because `v2.2.7` was a later published, non-draft, non-prerelease OpenCode product release.
Tags for separately released monorepo packages, such as `process-daemon-*`, were excluded from the product-release selection.

## Exact source evidence

The following source-evidence files are preserved without modification:

| File | SHA-256 |
| --- | --- |
| `provenance/opencode-v2.2.7/openapi.json` | `a2c9b373ac1e5bb0c8658cb4a9dc5db465e750c3c9e92a7234db65621a71bcd8` |
| `provenance/opencode-v2.2.7/LICENSE` | `625f0f619133f89bbbb2abe37369613dfa1885eba1e50d02170deb62bb42cb6b` |

The OpenAPI source path in the OpenCode repository is `packages/protocol/openapi.json`.
Its package version is `2.2.7`.

## Contract inventory

The frozen OpenAPI document declares:

- OpenAPI version `3.1.0`
- 139 paths
- 170 HTTP operations
- 497 component schemas

The document describes itself as an experimental surface for selected instance routes.
It is necessary source evidence, but it is not sufficient proof of runtime parity.
Observed behavior, OpenCode tests, streaming transcripts, error behavior, and unchanged official-client operation are also required.

The earlier `v2.1.4` planning snapshot contained 124 paths, 149 operations, and 293 schemas.
Those numbers are historical context only and are not TrueCode's target contract.

## Reference release artifacts

The following official artifacts are the primary comparison binaries for supported platforms:

| Artifact | SHA-256 |
| --- | --- |
| `opencode-darwin-arm64.zip` | `811c42be9cad489ea8c411ead49e06f691deee4957ba9a0a56a159a7d6c1abdc` |
| `opencode-linux-arm64.tar.gz` | `c1682057f0815cb2abdd6e00305f1146ebaf1c167f0335cdb74ddc80008120e7` |
| `opencode-linux-arm64-musl.tar.gz` | `a03405f94ae02db7acb0870cf32030bab86460dc7faf524780e9cae305526b96` |
| `SHA256SUMS` | `72c73a07124c65662239facaa7da2f3fe474078bbd4cbfa68bf90ae8fdca3b8b` |

Reference artifacts must be downloaded into disposable, ignored test storage and verified before execution.
They must not read a developer's normal OpenCode or TrueCode state.

## Freeze policy

The target does not move when OpenCode publishes another release.
Changing the target requires a new ADR, a new immutable source snapshot, a regenerated inventory, and a new conformance baseline.
Compatibility work for another release must not silently alter claims for `v2.2.7`.

## Attribution

OpenCode is Copyright (c) 2025 opencode and is licensed under the MIT License.
The preserved license text remains authoritative for copied OpenCode material.
TrueCode's own source is separately covered by the repository root `LICENSE`.
