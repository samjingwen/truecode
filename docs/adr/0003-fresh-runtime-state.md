# ADR 0003: Fresh runtime state

- Status: Accepted
- Date: 2026-11-13

## Context

Historical OpenCode and TrueCode data may contain private sessions, prompts, repository content, provider credentials, OAuth tokens, stale process metadata, and schema assumptions from prior experiments.
Migrating that material is unnecessary for proving a new runtime and would expand security and compatibility risk.

## Decision

TrueCode starts with a fresh runtime namespace and a native fresh SQLite schema.

Development, conformance, and release work must not inspect, import, attach, migrate, or modify old OpenCode or TrueCode databases, sessions, retained data, caches, or credentials.
Tests use owned disposable config, data, cache, home, repository, and credential roots.

Live provider testing uses only newly provisioned dedicated credentials supplied for this project.

## Consequences

Users begin with no historical sessions or credentials.
The physical database schema does not need compatibility with OpenCode.

Public migration operations in the frozen API still require behavioral characterization, but private historical data must not be used to exercise them.
Future TrueCode-native schema upgrades remain supported through append-only migrations after the baseline.
