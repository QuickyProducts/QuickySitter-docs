---
title: Version Bump Convention
sidebar: home_sidebar
permalink: version-bump.html
keywords: version, bump, convention, semver, lsl
toc: true
---

QuickySitter LSL scripts carry a `string version = "X.YYY…"` global near the top of every file. The whole fork is **version-locked as a set**: every shipped `[QS]*.lsl` currently reports the same number, `0.999`. The number is a shared build counter for the fork, not a per-script semantic version.

## The current step: +0.00001

A routine change bumps the locked version by **0.00001**. The next routine bump from today's baseline is therefore:

```
0.999 → 0.99901
```

The step has shrunk over time as the fork settled and PR cadence rose. For reading older commit history:

| In effect | Step | Example bump |
|-----------|------|--------------|
| from **2026-06-04** (current) | `+0.00001` | `0.999` → `0.99901` |
| 2026-05-24 – 2026-06-03 | `+0.0001` | `0.9989` → `0.999` |
| before 2026-05-24 | `+0.001` | `0.997` → `0.998` |

So a fifth-decimal digit in a `version =` line is a current-era bump; trailing `0.99x`/`0.9xx` numbers in comments or old commits are change-history, not the live value.

Because the set is locked, **every touched script gets the same new number in a PR**, so you don't carry independent per-script counters. Bump them together.

> Don't hardcode `0.999` anywhere as if it were permanent. It's just today's baseline. Read the live value from the file header before bumping.

## Where the version lives

Near the top of each `[QS]*.lsl`, before any other globals:

```lsl
string version = "0.999";
```

It appears in:

- The QSALIVE reply (field 1): only `[QS]sitA` answers QSALIVE, so this carries the sitter's version; consumers can substring-match for capability gating.
- The `Out(level, …)` diagnostic prefix (each line is tagged `[<version>]`). See [Debug Flags](debug-flags.html).
- The `[DUMP]` output header.
- Commit messages: see Commit message format below.

## Commit message format

Every script bump in a commit is named in the subject line:

```
[QS]sitA 0.99901, [QS]boot 0.99901: fix QSALIVE late-arrival race
```

Multiple scripts in one commit are listed comma-separated, all sharing the same new number. The body explains the change. The version part is always quoted in the subject so `git log --oneline` reads naturally.

This convention is enforced socially, not by tooling. Reviewers check that touched scripts bumped.

## Announce the bump

Every edit and PR that touches a version states the file and the old → new number up front, e.g.:

```
Update: [QS]sitA 0.999 → 0.99901, [QS]boot 0.999 → 0.99901
```

This makes review queues easy to skim for "which scripts changed, and to what." See [Contributing](contributing.html).

## Coordinated bumps across worktrees

Several agents may be working in parallel on Claude worktrees. Before bumping, check sibling worktrees for an in-flight bump on the same script. Two independent `0.999 → 0.99901` bumps in different worktrees both look correct in isolation but collide on merge.

The convention: scan sibling `.claude/worktrees/*` paths for the same `[QS]*.lsl` file with a newer `version =` line. If a sibling has already bumped to `0.99901`, use `0.99902` in your worktree.

## See also

- [Repo Structure](repo-structure.html): where scripts live in the repo.
- [Contributing](contributing.html): PR / commit-message workflow.
