---
title: 3+ Avatars
sidebar: home_sidebar
permalink: multi-avatar.html
keywords: multi-avatar, group, threesomes, more sitters
toc: true
---

QuickySitter's core engine imposes no sitter-slot limit of its own, so the seat count you can wire in the AVpos notecard is bounded mainly by SL's per-prim script-count cap. The *product family* does add caps, though: `[QS]hudproxy` hard-caps concurrent HUD-driven sitters at **6**, and the `[QS]faces`, `[QS]sequence` and `[QS]select` plugins are sized for a small per-furniture cap (around 7 pre-handshake). So on a piece using the HUD or those plugins, plan around those limits rather than SL's script cap. The setup is a direct extension of the [Couples Setup](couples-setup.html) procedure.

> **Working alone?** The Creator Edition's [Animesh Adjust Dummies](quickysitter-pro-animesh.html) fill the empty seats of a group pose while you set it up: one posable dummy per seat, no extra avatars needed.

## Adding more sitters

For N sitter slots:

- `[QS]boot` (still just one instance)
- `[QS]sitA`, `[QS]sitA 1`, `[QS]sitA 2`, …, `[QS]sitA <N-1>`
- `[QS]sitB`, `[QS]sitB 1`, `[QS]sitB 2`, …, `[QS]sitB <N-1>` (matching count)

The suffixes start at ` 1` (the unnumbered script is slot 0) and must be **contiguous**: sitA counts its siblings by probing `[QS]sitA 1`, `[QS]sitA 2`, … until the first gap, so a missing number silently truncates the slot count.

Each slot has its own sit-target offset, its own `POSE`/`SYNC` lines in the AVpos notecard, and its own row in `qs:cfg:<ch>` / `qs:sitter:<ch>` LSD keys. (`ANIM` is a separate directive, the `[QS]faces` face-animation command, not a per-slot pose line.)

## SYNC across N slots

A SYNC pose with the same `<menu_name>` in N slots animates all N sitters in lockstep. Each slot picks its own per-slot animation file:

```
SITTER 0
SYNC Group-Hug|hug_left

SITTER 1
SYNC Group-Hug|hug_center

SITTER 2
SYNC Group-Hug|hug_right
```

`SITTER <n>` opens the section for sitter slot `<n>`; all subsequent POSE/SYNC/BUTTON lines belong to that slot until the next `SITTER`. See [AVpos Reference](avpos-reference.html) for the full grammar, and the [upstream multi-sitter docs](https://avsitter.github.io/avsitter2_home.html) for tutorial-style examples.

## Re-Sync with N sitters

LinkMsg 90271 broadcasts to all sitA slots simultaneously via `LINK_SET`. Each slot re-phases its main animation in the same Sim frame; resulting drift is bounded to ~50 ms regardless of how many sitters are seated. See [Re-Sync Protocol](resync-protocol.html).

## Memory considerations

At 1000+ poses across N slots, QuickySitter's LSD-backed menu storage becomes a noticeable improvement over stock, because sitB stays slim while stock would push past the 64 KB Mono cap. The old in-RAM `MENU_LIST` was retired (0.9954); menu state is now built from paged LSD reads of `qs:p:<ch>:<i>`, windowed by the `qs:nm` sidecar. See [LSD Storage](lsd-storage.html).

The other scaling concern is `[QS]offset`'s RAM tier (200-entry LRU cap). With N users × N slots × M poses of personal offsets, persistent LSD storage (`QSO:*`) is the durable place; RAM is the volatile fallback. See [Personal Pose Offsets](personal-pose-offsets.html).

## Menu navigation

With many poses and many sitters, the dialog menu pagination becomes important. The `[<<]` / `[>>]` buttons page through entries when a section overflows; menu structure (top-level TOMENU buttons → submenus) helps group poses by scene.

See [Submenus](submenus.html) for navigation design.

## Common gotchas

- **`[QS]select`.** Optional and presence-gated, since sitB has a built-in seat picker, so multi-sitter menu routing works without it. When `[QS]select` is present (sitB reads the `qs:alive:select` flag, with a `[AV]select` inventory probe as fallback), it provides the dedicated seat-select picker.
- **Seat prims.** SL's sit-target clamp is ±300 m per axis, so offsets are not the constraint on long furniture. Still, give each seat its own prim and pin it via the prim description (`<set>-<slot>`): right-click Sit docks on the clicked prim, so separate seat prims are what make "sit where you click" work on a banquet table with 8 sitters. See [SitTargets](sittargets.html).

## See also

- [Couples Setup](couples-setup.html): the 2-sitter case.
- [Animation Sequences](animation-sequences.html): multi-step sequences across multiple sitters.
- [Re-Sync Protocol](resync-protocol.html): N-sitter sync.
- [SitTargets](sittargets.html): per-slot offsets.
