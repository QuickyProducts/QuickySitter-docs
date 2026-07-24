---
title: Version Bump Convention
sidebar: home_sidebar
permalink: version-bump.html
keywords: version, bump, convention, semver, lsl
toc: true
---

QuickySitter LSL scripts carry a `string version = "X.YY…"` global near the top of every file. Each script has its **own** version and they drift independently between releases; a shared, uniform number only appears at a release (see below). Read the live value from the file header before bumping — never assume all scripts share a number.

## The default step: +0.0001

A routine change (bug fix that isn't a blocker, refactor, cosmetic edit) bumps the touched script by **+0.0001**:

```
1.0401 → 1.0402
```

A **feature or a blocker fix** rounds the version **up to the next hundredth** instead:

```
1.0402 → 1.05      (feature/blocker, not 1.0403)
1.04   → 1.0501    (a blocker fix lands on top of the rounded 1.05 line)
```

This is a real convention from the history, not a guess: commit `dca64b0` bumped three scripts in one commit to **three different numbers** (`[QS]root-security` 1.0501, `[QS]sitB` 1.0501, `[QS]adjuster` 1.0502) for the Adjust-ACL feature, while `792ce6e` was a routine `+0.0001` (`1.0401 → 1.0402`).

The step has changed over the project's life; for reading old commit history:

| In effect | Default step |
|-----------|--------------|
| from **2026-07-03** (current) | `+0.0001` |
| 2026-06-12 – 2026-07-02 | `+0.001` |
| feature/blocker round-up to the next hundredth has applied since 2026-06-15 |

> Pre-release iteration on an **unreleased** script always uses the plain default step, even for a feature — the round-up rule is for shipped scripts.

## Releases stamp a uniform number

A **release** is the only time all product scripts share a version: the release stamps every script in the set to one new number, chosen above the highest per-fix version currently in the set. The current release is **1.25** (commit `e568aa9` unified the sitter set; the QuickyHUD kit and the QuickySitter engine share the number from this release on, which is why the sitter jumped `1.04 → 1.25` to meet the kit).

Between releases, scripts carry their independent per-fix numbers again. So a bare `1.25` across the whole set means "as shipped in release 1.25"; mixed numbers like `1.0501` / `1.0502` are normal mid-cycle state.

Folding is allowed while a release is **unshipped**: an interim per-fix bump made after the release tag can be folded back into the release number and the tag moved, since nothing was delivered (e.g. `6cb2659` bumped `[QS]prop` to 1.26, then `9fb5fca` folded it back into 1.25).

## Where the version lives

Near the top of each `[QS]*.lsl`:

```lsl
string version = "1.25";
```

It appears in:

- The QSALIVE reply: only slot-0 `[QS]sitA` answers QSALIVE, so this carries the **sitter's** version; consumers can substring-match for capability gating.
- The `Out(level, …)` diagnostic prefix (each line is tagged `[<version>]`). See [Debug Flags](debug-flags.html).
- The `[DUMP]` output header.
- Commit subject lines (below).

## Commit message format

Every script bump is named in the subject line with its own old → new number:

```
[QS]root-security 1.04 -> 1.0501, [QS]sitB 1.04 -> 1.0501, [QS]adjuster 1.04 -> 1.0502: Adjust access ACL
```

Scripts in the same commit that genuinely landed on different numbers list them separately — don't force them to match. The body explains the change.

This convention is enforced socially, not by tooling. Reviewers check that touched scripts bumped.

## Announce the bump

Every edit that touches a version states the file and old → new number up front, e.g.:

```
Update: [QS]sitA 1.04 → 1.0401
```

For several files, a small table at the top of the reply. See [Contributing](contributing.html).

## Coordinated bumps across worktrees

Several agents may work in parallel on Claude worktrees. Before bumping, scan sibling `.claude/worktrees/*` paths for an in-flight bump on the same script — two independent bumps to the same number in different worktrees each look correct in isolation but collide on merge. If a sibling already took `1.0402`, use `1.0403`.

## See also

- [Repo Structure](repo-structure.html): where scripts live in the repo.
- [Contributing](contributing.html): PR / commit-message workflow.
