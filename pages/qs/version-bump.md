---
title: Version Bump Convention
sidebar: home_sidebar
permalink: version-bump.html
keywords: version, bump, convention, semver, lsl
toc: true
---

QuickySitter LSL scripts carry a `string version = "X.YYY"` global near the top of every file. Versions bump in steps of **0.001**, not 0.01.

## Why 0.001

Each script in the fork evolves independently. A typical PR touches two or three scripts; bumping by 0.01 once per change would burn through a "minor version" on every commit. The 0.001 step lets us treat the version number as a fine-grained build counter without losing the larger-scale semantic-version intuition for big changes.

Examples:

- `0.281` → `0.282`: routine refactor, bug fix, small feature.
- `0.299` → `0.300`: minor "milestone" — accumulated changes since 0.290.
- `0.900` → `0.901`: significant feature (e.g., QSDUMP migration).
- `0.999` → `1.000`: major release / stability landmark.

## Where the version lives

Near the top of each `[QS]*.lsl`, before any other globals:

```lsl
string version = "0.915";
```

It appears in:

- The QSALIVE reply (field 1) — plugins can substring-match for capability gating on specific versions.
- The `state_entry` `llOwnerSay` log (when `bDebug` is true).
- The `[DUMP]` output header.
- Commit messages — see Commit message format below.

## Commit message format

Every script bump in a commit is named in the subject line:

```
[QS]sitA 0.916, [QS]boot 0.916: fix QSALIVE late-arrival race
```

Multiple scripts in one commit are listed comma-separated. The body explains the change. The version part is always quoted in the subject so `git log --oneline` reads naturally.

This convention is enforced socially, not by tooling. Reviewers check that touched scripts bumped.

## Coordinated bumps across worktrees

Several agents may be working in parallel on Claude worktrees. Before bumping a script version, check sibling worktrees for an in-flight bump on the same script — two independent `0.916 → 0.917` bumps in different worktrees will both look correct in isolation but produce a collision on merge.

The convention: scan sibling `.claude/worktrees/*` paths for the same `[QS]*.lsl` file with a newer `version =` line. If a sibling has bumped to `0.917`, use `0.918` in your worktree.

## See also

- [Repo Structure](repo-structure.html) — where scripts live in the repo.
- [Contributing](contributing.html) — PR / commit-message workflow.
