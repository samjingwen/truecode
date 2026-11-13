# TrueCode change delivery

This policy applies to the TrueCode repository and its explicitly assigned implementation worktrees.
Read it together with `AGENTS.md` and the applicable compatibility, architecture, conformance, runtime, storage, security, plugin, platform, and release documentation.
Do not depend on an instruction file in a parent workspace.

## Scope and authority

Own the applicable delivery sequence: dedicated Worktrunk worktree, requirements and implementation, exact-head executable verification and Hardlook review, GitHub pull request, protected merge, and safe local primary-branch synchronization.
Commit intended changes, push the feature branch, open the pull request, and merge only after required local evidence, review, and conversation resolution are complete.

Plan-only, review-only, pull-request-only, human-review, native-runner, release, production, credential, networking, and data-retention limits take precedence over this default sequence.
Continue into packaging, release, downstream integration, or live-provider testing only when the task explicitly authorizes that scope.
Do not treat tool availability or a successful source merge as authorization to operate services, credentials, external systems, or user data.

GitHub-hosted CI is intentionally disabled.
GitHub remains the source-review and protected-merge boundary, not a cloud build service.
The absence of remote checks does not waive executable quality checks, independent review, native platform evidence, branch protections, or unresolved acceptance requirements.

## Inspect and isolate

- Inspect repository status, worktrees, the actual default branch, its upstream, open pull requests, and active task ownership before editing.
- Record the primary checkout path that owns the default branch so final synchronization does not require rediscovery.
- Read `wt switch --help`, inspect effective Worktrunk configuration and hooks, and understand any commands that will run before creating a worktree.
- Fetch the intended base or use the exact base explicitly selected for the task.
- Create a dedicated implementation worktree with Worktrunk rather than changing the primary checkout directly.
- Use the path returned by Worktrunk and verify the actual directory, branch, base, upstream, and clean status before the first edit.
- Continue related work in its existing assigned worktree instead of creating a competing branch.
- Do not move, overwrite, clean, reset, stash, or rebase another task's uncommitted work.
- Keep documentation review separate from runtime implementation unless the task explicitly combines them.

## Preserve source evidence and private state

- Target only the frozen OpenCode release named in `UPSTREAM.md` unless a superseding ADR explicitly changes it.
- Treat `upstream/opencode-v2.2.7/openapi.json` and its preserved license as immutable source evidence.
- Regenerate derived protocol artifacts with their owning tool and source hash rather than editing generated output.
- Never manually edit `CHANGELOG.md` or files marked as generated.
- Never inspect, import, migrate, attach, or modify old OpenCode or TrueCode databases, sessions, retained data, caches, or credentials.
- Never read private environment files, authentication caches, browser state, SSH material, or unrelated repositories to make a test pass.
- Use only fresh owned disposable fixtures within the task's approved filesystem, process, network, credential, and provider limits.
- Preserve unrelated user work, ignored files, worktrees, local artifacts, and private material.

## Implement within accepted decisions

- Read the relevant ADRs and domain documents before changing an accepted behavior or architecture boundary.
- Update or supersede the applicable ADR before implementing a conflicting technical decision.
- Keep the OpenCode `v2.2.7` wire and observed behavior contract distinct from TrueCode's implementation identity.
- Maintain the documented JavaScript and TypeScript plugin ABI break rather than introducing partial hidden compatibility.
- Keep Windows and x86_64 outside the first-release claim unless a new approved decision expands the platform matrix.
- Enforce permissions before side effects and keep host-tool execution separate from WASI plugin containment.
- Keep queues, retries, output, task ownership, subprocesses, PTYs, MCP calls, and plugin work bounded and cancellable.
- Add append-only SQLite migrations after the baseline is established and pair each migration with compatible rollout and recovery behavior.
- Do not use database deletion or fixture reset as a migration or recovery strategy for retained user data.

## Verify the exact change

- Use the project's documented dependency preparation and quality entrypoints.
- Until executable entrypoints exist, validate documentation, source hashes, contract counts, links, formatting, and repository invariants with available local checks.
- Once Rust exists, retain strict `rustfmt`, compilation, Clippy, unit, integration, and relevant conformance checks.
- Run behavioral tests through public APIs, clients, processes, PTYs, Git repositories, and other real boundaries where the change affects them.
- Reproduce reported bugs through the user-facing API or unchanged official-client path before fixing them.
- Prove enforcement with passing conforming fixtures and representative rejected or failing fixtures.
- Preserve failing evidence, fix causes rather than weakening checks, and rerun checks invalidated by each fix.
- Run checks against the exact commit proposed for merge or rerun them after the final commit.
- Record each command, result, target platform, fixture or artifact digest where relevant, skipped coverage, and limitation.
- Do not represent cross-compilation or one operating system's results as native coverage for another target.

## Review

- Review the complete default-branch-to-head diff, including committed, staged, unstaged, and untracked files.
- Use the installed Hardlook skill for an independent full-diff review.
- Record review coverage, findings, fixes, residual risks, and unavailable evidence.
- Resolve every substantiated blocking finding and rerun checks invalidated by its fix.
- A partial review, unavailable tool, failed check, undocumented skip, or passing rerun without a substantiated fix is not clean acceptance.
- Keep release, runtime, provider, credential, and downstream integration holds in force even when source checks pass.

## Commit and publish

- Review status and stage only the task's intended files.
- Use commit messages that describe the change without adding agent attribution or co-author trailers.
- Verify the committed tree and rerun exact-head checks when the commit changes relevant bytes.
- Push the feature branch without force.
- Prefer `gh-axi` for GitHub operations, with `gh` only when the required operation is unavailable in `gh-axi`.
- Open a pull request into the actual protected default branch.
- Include decisions, exact-head local evidence, Hardlook results, limitations, compatibility impact, migration notes, platform evidence, and any separately authorized release implications.

## Merge through protection

- Satisfy actual branch protection, review, and conversation-resolution requirements without bypassing them.
- Do not invent hosted checks for this no-CI project.
- Confirm that the pull request head still equals the locally verified commit before merge.
- Merge through GitHub and verify the pull request state, resulting merge method, and exact merged commit.
- Do not push directly to the protected default branch after repository bootstrap.

## Synchronize the primary checkout

- Fetch the remote after merge and identify its current default-branch tip.
- Inspect the primary checkout's dirty, untracked, and ignored paths plus active ownership before synchronization.
- Pull the default branch with fast-forward-only semantics when safe.
- Never stash, reset, clean, force refs, discard files, or rebase local divergence to make synchronization succeed.
- Preserve user-owned untracked and ignored files and report any collision or ownership blocker.
- Verify that local default-branch `HEAD` equals the freshly fetched remote tip and contains the merged revision.
- If synchronization is blocked, remain in the implementation worktree and report the blocker without changing or moving the user's files.
- After safe synchronization, return the current OpenCode session to the recorded primary checkout with `opencode.session_move` when the session is elsewhere.
- Do not use `wt switch`, `cd`, shell integration, or a new OpenCode process as a substitute for moving the current OpenCode session.
- Call `opencode.session_move` alone at a safe tool boundary because the directory change applies at the next boundary.
- Verify directory, branch, `HEAD`, upstream tip, merge ancestry, and status in a separate tool call after the move.
- If the session already uses the primary checkout, skip the move and perform the same final verification.
- Do not remove the implementation worktree automatically after returning the session.

## Release and external effects

- Perform a release only with explicit release authority and from a clean merged revision.
- Bind artifacts to source commit, Rust toolchain, target triple, build command, and cryptographic digest.
- Require native evidence before claiming macOS ARM64, GNU/Linux AArch64, or musl Linux support.
- Use only newly supplied dedicated provider and OAuth credentials within an approved live-validation plan.
- Do not switch TenAxe or another downstream consumer from the official OpenCode runtime without separate acceptance and explicit authorization.
- Do not claim that binary rollback reverses SQLite migrations, external provider actions, tool side effects, Git changes, or issued credentials.

## Final report

Report:

- Pull request URL and final state
- Feature-head commit and merged commit
- Exact local checks and their results
- Hardlook coverage and findings
- Native platform and live-integration evidence actually obtained
- Primary checkout path, branch, `HEAD`, upstream equality, and status
- Preserved worktrees and files
- Any blocker, skip, limitation, accepted deviation, migration, release, or external effect

Describe only evidence that was actually produced.
