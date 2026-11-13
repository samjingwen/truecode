# ADR 0001: Freeze OpenCode v2.2.7

- Status: Accepted
- Date: 2026-11-13

## Context

OpenCode changes frequently across routes, schemas, runtime semantics, providers, and clients.
A reimplementation cannot establish parity against a moving development branch.

At kickoff, OpenCode `v2.2.7` was the newest published, non-draft, non-prerelease OpenCode product release.
It was published after the release that GitHub's `releases/latest` endpoint still marked as latest.

## Decision

TrueCode targets OpenCode tag `v2.2.7` at commit `fedc6f208aa2bd54c1019227214568083f6380ad`.

The immutable OpenAPI source and OpenCode MIT license are checked into `provenance/opencode-v2.2.7/` with hashes recorded in `provenance/README.md`.
Implementation and conformance work must use this tag, commit, and official artifact digests.

Later OpenCode releases do not change this target automatically.

## Consequences

TrueCode can make reproducible compatibility claims against one defined contract.
New OpenCode behavior remains out of scope until a separate upgrade effort is approved.

Changing the target requires a superseding ADR, a new source snapshot, a regenerated inventory, updated official artifacts, and a complete conformance reassessment.
