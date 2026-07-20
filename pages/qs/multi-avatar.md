---
title: 3+ Avatars
sidebar: home_sidebar
permalink: multi-avatar.html
keywords: multi-avatar, group, threesomes, more sitters
toc: true
---

QuickySitter supports any number of sitter slots, and the limit is SL's per-prim script-count cap, not anything in QS itself. The setup is a direct extension of the [Couples Setup](couples-setup.html) procedure.

> **Working alone?** QuickySitter Pro's [Animesh Adjust Dummies](quickyhud-animesh.html) fill the empty seats of a group pose while you set it up: one posable dummy per seat, no extra avatars needed.

## Adding more sitters

For N sitter slots:

- `[QS]boot` (still just one instance)
- `[QS]sitA`, `[QS]sitA 2`, …, `[QS]sitA <N>`
- `[QS]sitB`, `[QS]sitB 2`, …, `[QS]sitB <N>` (matching count)

Each slot has its own sit-target offset, its own ANIM lines in the AVpos notecard, and its own row in `qs:cfg:<ch>` / `qs:sitter:<ch>` LSD keys.

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
- **Sit-target clamp.** SL clamps sit-target offsets to ±1.7 m for ground prims. For very long furniture (e.g., banquet table with 8 sitters), you'll need linked child prims with their own sit-targets, not stretched offsets from a single root.

## See also

- [Couples Setup](couples-setup.html): the 2-sitter case.
- [Animation Sequences](animation-sequences.html): multi-step sequences across multiple sitters.
- [Re-Sync Protocol](resync-protocol.html): N-sitter sync.
- [SitTargets](sittargets.html): per-slot offsets.
