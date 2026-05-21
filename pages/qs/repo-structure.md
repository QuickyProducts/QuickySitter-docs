---
title: Repo Structure
sidebar: home_sidebar
permalink: repo-structure.html
keywords: repository, structure, layout, contributing
toc: true
---

The QuickySitter source tree is laid out so that fork-specific code (`qs/`) and a pinned snapshot of the AVsitter upstream (`avstock/`) live side by side. Documentation lives in this separate repository.

## Source repositories

| Repo | What's there |
|------|--------------|
| [QuickyProducts/QuickySitter](https://github.com/QuickyProducts/QuickySitter) | LSL scripts, build helper, vendored AVsitter snapshot, in-repo design docs. |
| [QuickyProducts/QuickySitter-docs](https://github.com/QuickyProducts/QuickySitter-docs) | Jekyll documentation site (this site). |
| [QuickyProducts/QuickyHUD](https://github.com/QuickyProducts/QuickyHUD) | HUD addon scripts (hudproxy, hudadmin, the wearable Quicky-Pose-HUD). Sibling project. |

## QuickySitter source tree

```
QuickySitter/
├── qs/                      ← QuickySitter scripts and design docs
│   ├── [QS]boot.lsl
│   ├── [QS]sitA.lsl
│   ├── [QS]sitB.lsl
│   ├── [QS]select.lsl
│   ├── [QS]adjuster.lsl
│   ├── [QS]prop.lsl
│   ├── [QS]faces.lsl
│   ├── [QS]offset.lsl
│   ├── [QS]sequence.lsl
│   ├── [QS]debug.lsl
│   ├── PROTOCOL.md          ← Fork-specific link-message protocol
│   ├── STORAGE.md           ← LSD layout and state model
│   ├── TESTPLAN.md          ← Sync-drift investigation, test scenarios
│   └── TODOLIST.md
│
├── avstock/                 ← Pinned AVsitter snapshot (upstream reference)
│   ├── Plugins/
│   │   ├── AVcamera/
│   │   ├── AVcontrol/
│   │   ├── AVfavs/
│   │   └── AVprop/
│   ├── Utilities/
│   ├── [AV]helperscript.lsl
│   ├── [AV]root-security.lsl
│   ├── avsitter2_link_message_reference.md
│   ├── BUILD_GUIDE.md
│   ├── IMPORT_GUIDE.md
│   └── README.md
│
├── README.md
└── .gitignore
```

## qs/ — fork-specific code

Each `[QS]*.lsl` is a self-contained script. Convention: file header starts with a short comment block (purpose, key inputs/outputs, dependencies), followed by `string version = "X.YYY";` and any other top-level constants. The version line is what plugins read via QSALIVE; bumps follow the [0.001-step convention](version-bump.html).

### Script-by-script summary

| Script | Role |
|--------|------|
| `[QS]boot` | One-shot LSD seeder. Parses `AVpos` notecard on first boot per asset-key, writes `qs:cfg`/`qs:sitter`/`qs:p:*`/`qs:meta:*`. Owns the DUMP cascade. |
| `[QS]sitA` | Main sitter/pose script. One per sitter slot. Handles permissions, animations, sit-targets, the public menu. |
| `[QS]sitB` | Menu and pose-state companion to sitA. One per sitter slot. Reads pose defaults from LSD on demand. |
| `[QS]select` | Sit-time menu router for multi-furniture and multi-sitter setups. |
| `[QS]adjuster` | Creator-tool runtime. `[HELPER] [SAVE]`, sit-target adjust, `[DUMP]` kicker. Optional. |
| `[QS]prop` | Prop spawning. Stock-compatible plus the QSPROP_ATTACH (90280) dynamic protocol. |
| `[QS]faces` | Face / expression animations. Adds QS_FACES_HELLO (90090) presence broadcast. |
| `[QS]offset` | Personal pose offsets. Two-tier store (LSD `QSO:*` + RAM `CUSTOMS`). Single source of truth. |
| `[QS]sequence` | Animation sequences. QSDUMP-aware. |
| `[QS]debug` | Diagnostics. `bDebug` flag in other scripts logs to this script's chat output. |

### In-repo design docs

The three Markdown files in `qs/` are the source of truth for many pages on this docs site:

- [`qs/PROTOCOL.md`](https://github.com/QuickyProducts/QuickySitter/blob/master/qs/PROTOCOL.md) — link-message contracts, fork-specific number ranges, capability tokens.
- [`qs/STORAGE.md`](https://github.com/QuickyProducts/QuickySitter/blob/master/qs/STORAGE.md) — LSD key layout, state per script, reset behavior.
- [`qs/TESTPLAN.md`](https://github.com/QuickyProducts/QuickySitter/blob/master/qs/TESTPLAN.md) — sync-drift test scenarios and design decisions for Re-Sync.

When these documents disagree with the docs site, the in-repo files are canonical. PRs that modify protocol behavior should update PROTOCOL.md/STORAGE.md in the same change.

## avstock/ — vendored AVsitter snapshot

`avstock/` contains a pinned copy of the AVsitter 2 upstream — kept verbatim, never edited, refreshed by running `avstock/build-aux.py` against a specific upstream commit SHA. The SHA is recorded in `avstock/README.md`.

Two purposes:

1. **Diff source for review.** When QS forks a stock script, reviewers can see exactly what changed against the pinned upstream version without checking out a separate repo.
2. **Drop-in fallback.** If a creator wants to mix stock and QS plugins, the stock versions are right here.

`avstock/avsitter2_link_message_reference.md` is the canonical stock link-message reference, vendored from upstream.

## Worktrees and parallel work

Long-running Claude sessions often spawn worktrees under `.claude/worktrees/<name>/` to keep work isolated. Each worktree is a full checkout of the repo at a feature branch. Two conventions matter:

- **Edit paths must live under the active worktree root.** A worktree session shouldn't touch a sibling worktree's files.
- **Coordinate version bumps.** Before bumping a script in your worktree, check sibling worktrees for the same script with an in-flight bump — two parallel `0.916 → 0.917` bumps will both look correct in isolation but collide on merge. See [Version Bump Convention](version-bump.html).

## See also

- [Getting Started](getting-started.html) — installing the scripts in an in-world prim.
- [Contributing](contributing.html) — PR / commit workflow.
- [Version Bump Convention](version-bump.html) — the 0.001-step rule.
