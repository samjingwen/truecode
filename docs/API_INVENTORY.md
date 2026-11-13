# Frozen API inventory

## Source

This inventory is derived from `provenance/opencode-v2.2.7/openapi.json`.
The immutable source came from `packages/protocol/openapi.json` in OpenCode `v2.2.7` at commit `fedc6f208aa2bd54c1019227214568083f6380ad`.

The source hash is `a2c9b373ac1e5bb0c8658cb4a9dc5db465e750c3c9e92a7234db65621a71bcd8`.

## Summary

| Property | Count |
| --- | ---: |
| Paths | 139 |
| Operations | 170 |
| Component schemas | 497 |
| Operations missing `operationId` | 0 |
| Duplicate operation IDs | 0 |

The document uses OpenAPI `3.1.0`.
It declares no top-level server entries and no OpenAPI security schemes.
Authentication and pairing behavior therefore require source and observed-behavior inventory beyond schema generation.

## Operations by method

| Method | Count |
| --- | ---: |
| `GET` | 78 |
| `POST` | 62 |
| `DELETE` | 18 |
| `PATCH` | 7 |
| `PUT` | 5 |

## Operations by tag

| Tag | Operations |
| --- | ---: |
| `agent` | 2 |
| `command` | 1 |
| `config` | 3 |
| `credential` | 5 |
| `debug` | 3 |
| `event` | 1 |
| `experimental` | 15 |
| `filesystem` | 6 |
| `form` | 1 |
| `generate` | 4 |
| `integration` | 12 |
| `location` | 2 |
| `mcp` | 8 |
| `migration` | 1 |
| `model` | 2 |
| `permission` | 8 |
| `persistentPty` | 11 |
| `plugin` | 4 |
| `project` | 2 |
| `provider` | 2 |
| `pty` | 7 |
| `reference` | 1 |
| `rpc` | 1 |
| `server` | 3 |
| `session` | 44 |
| `shell` | 5 |
| `skill` | 1 |
| `usage` | 1 |
| `vcs` | 6 |
| `voice` | 2 |
| `websearch` | 2 |
| `worktree` | 4 |

The tag counts total 170 operations.
Tags are an inventory aid and do not define implementation crate boundaries.

## Reproducible count

Run the following command from the repository root:

```sh
jq '{
  openapi,
  paths: (.paths | length),
  operations: ([
    .paths[]
    | to_entries[]
    | select(
        .key == "get"
        or .key == "put"
        or .key == "post"
        or .key == "delete"
        or .key == "options"
        or .key == "head"
        or .key == "patch"
        or .key == "trace"
      )
  ] | length),
  schemas: (.components.schemas | length)
}' provenance/opencode-v2.2.7/openapi.json
```

## Derived artifacts

Implementation work will generate the following artifacts from the immutable source:

- A method and path route manifest
- Request and response DTOs
- Schema validators for conformance fixtures
- An operation coverage manifest
- Client request generators for deterministic cases
- A machine-readable map from operations to conformance scenarios

Derived artifacts must identify the source hash.
Generated files must be reproducible and must not be manually edited.

## Inventory limits

The OpenAPI description calls the document an experimental surface for selected instance routes.
The inventory does not by itself specify:

- Authentication and pairing middleware behavior
- Header defaults and headers omitted from schemas
- SSE timing, keepalive, replay, and disconnect semantics
- WebSocket message timing and closure behavior
- Concurrent request ordering
- Process, signal, PTY, and worktree side effects
- Configuration discovery and precedence
- Provider and OAuth behavior outside structural response schemas
- Crash recovery and persisted transition boundaries
- Service installation and stale-process handling
- Plugin execution semantics

Each gap must be covered by observed-behavior inventory and differential scenarios.

## Historical comparison

The earlier OpenCode `v2.1.4` planning snapshot contained 124 paths, 149 operations, and 293 schemas.
TrueCode does not use those numbers as implementation or acceptance targets.
The growth to 139 paths, 170 operations, and 497 schemas is one reason the release target is frozen before implementation.
