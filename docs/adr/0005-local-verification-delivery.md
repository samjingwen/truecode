# ADR 0005: Local verification and protected delivery

- Status: Accepted
- Date: 2026-11-13

## Context

GitHub-hosted CI is too slow for the intended development loop.
TrueCode still requires executable verification, independent review, protected merging, and native platform evidence.

Platform-sensitive PTY, process, filesystem, service, and credential behavior cannot be proven by a generic hosted build alone.

## Decision

TrueCode does not use GitHub-hosted CI.
GitHub remains the source-review and protected-merge boundary.

Every change uses a dedicated Worktrunk worktree after repository bootstrap.
Relevant local checks run against the exact proposed commit.
The full base-to-head diff receives Hardlook review.
Pull requests record commands, results, commit identity, platform coverage, limitations, and release implications.

The protected `master` branch requires pull requests and resolved conversations, enforces protection for administrators, and blocks force pushes and deletion.
It initially requires no remote status checks and no human approval count.

Release claims require separately recorded native macOS ARM64 and Linux AArch64 verification.

## Consequences

Pull requests do not receive automatic cloud checks.
Reviewers must evaluate exact-head local evidence and cannot infer that absent evidence passed.

The project must provide fast, stable local quality entrypoints.
A later decision may add non-GitHub automation, but it cannot weaken protected delivery or native verification.
