---
title: 3+ Avatars
sidebar: home_sidebar
permalink: multi-avatar.html
keywords: multi-avatar, group, threesomes, more sitters
toc: true
---

QuickySitter supports any number of sitter slots — the limit is SL's per-prim script-count cap, not anything in QS itself. The setup is a direct extension of the [Couples Setup](couples-setup.html) procedure.

## Adding more sitters

For N sitter slots:

- `[QS]boot` (still just one instance)
- `[QS]sitA`, `[QS]sitA 2`, …, `[QS]sitA <N>`
- `[QS]sitB`, `[QS]sitB 2`, …, `[QS]sitB <N>` (matching count)

Each slot has its own sit-target offset, its own ANIM lines in the AVpos notecard, and its own row in `qs:cfg:<ch>` / `qs:sitter:<ch>` LSD keys.

## SYNC across N slots

A SYNC pose with the same `NAME` in N slots animates all N sitters in lockstep. Each slot picks the correct per-slot animation via the ANIM line:

```
SYNC
NAME Group-Hug
ANIM hug_left

SYNC  
NAME Group-Hug
ANIM hug_center

SYNC
NAME Group-Hug
ANIM hug_right
```

(Slot N's section in the AVpos notecard is delimited by the `MENU` and `TOMENU` structure; consult the [upstream multi-sitter docs](https://avsitter.github.io/avsitter2_home.html) for the full layout.)

## Re-Sync with N sitters

LinkMsg 90271 broadcasts to all sitA slots simultaneously via `LINK_SET`. Each slot re-phases its main animation in the same Sim frame; resulting drift is bounded to ~50 ms regardless of how many sitters are seated. See [Re-Sync Protocol](resync-protocol.html).

## Memory considerations

At 1000+ poses across N slots, QuickySitter's LSD-backed `MENU_LIST` becomes a noticeable improvement over stock — sitB stays slim while stock would push past the 64 KB Mono cap. See [LSD Storage](lsd-storage.html).

The other scaling concern is `[QS]offset`'s RAM tier (200-entry LRU cap). With N users × N slots × M poses of personal offsets, persistent LSD storage (`QSO:*`) is the durable place; RAM is the volatile fallback. See [Personal Pose Offsets](personal-pose-offsets.html).

## Menu navigation

With many poses and many sitters, the dialog menu pagination becomes important. The `[NEXT]` button cycles through pages; menu structure (top-level TOMENU buttons → submenus) helps group poses by scene.

See [Submenus](submenus.html) for navigation design.

## Common gotchas

- **`[QS]select`.** Recommended for any multi-sitter setup so the right sitter slot can route the right user's menu. Without it, multi-sitter menu routing falls back to the legacy `[AV]select` if present.
- **Sit-target clamp.** SL clamps sit-target offsets to ±1.7 m for ground prims. For very long furniture (e.g., banquet table with 8 sitters), you'll need linked child prims with their own sit-targets, not stretched offsets from a single root.

## See also

- [Couples Setup](couples-setup.html) — the 2-sitter case.
- [Animation Sequences](animation-sequences.html) — multi-step sequences across multiple sitters.
- [Re-Sync Protocol](resync-protocol.html) — N-sitter sync.
- [SitTargets](sittargets.html) — per-slot offsets.
