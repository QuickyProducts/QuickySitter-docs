---
title: SitTargets
sidebar: home_sidebar
permalink: sittargets.html
keywords: sit target, sittarget, set, dflt, default position
toc: true
---

A **sit-target** is the position and rotation the SL physics engine places an avatar at when they right-click → Sit. QuickySitter handles sit-targets identically to stock AVsitter 2, and this page covers the QS-relevant aspects. For the conceptual tutorial, see the [upstream AVsitter SitTargets page](https://avsitter.github.io/avsitter2_sittargets.html).

## The model

Each sitter slot (`[QS]sitA`, `[QS]sitA 2`, …) has its own sit-target. The sit-target is a `<position, rotation>` pair set via `llSitTarget` (or `llSetLinkSitTarget` on the linked prim).

When a pose plays, it adds an additional `POS` / `ROT` offset on top of the sit-target. The total avatar position is:

```
avatar = prim_position + sit_target_offset + pose_POS
```

The sit-target is **per-slot**, the pose POS is **per-pose**.

## SET sets

The `SET <n>` directive declares how many sit-target *sets* the furniture has. It is a plain channel-level directive, and there is no `SETUP` section wrapping it. Each set is a different physical seating arrangement (e.g., chair facing left, chair facing right, couples on a sofa, solo on a bench). The user picks the set via the `[SET]` button in the menu.

`DFLT <n>` sets which set is active by default (1-based).

For each set there's a per-slot sit-target offset. The notecard syntax for declaring multiple sets is in the upstream docs.

## Adjusting sit-targets in-world

With `[QS]adjuster` installed, the `[ADJUST]` → `[HELPER]` → `[SITTARGET]` menu path enters sit-target adjustment mode:

- Helper-bar arrows move the **sit-target itself** (not the pose offset).
- Click `[SAVE]` to commit.
- The new sit-target persists across resets because the prim's link properties survive `llResetScript` (unless the prim is also taken/rerezzed, which preserves the saved sit-target via inventory).

## QS-specific: sit-target sync via 90150

When slot-0's `[QS]sitA` resets (typically because boot re-seeded LSD), it broadcasts `90150` so all other sitA slots in the prim re-place their own sit-targets. Without this, a notecard re-save could leave sit-target offsets inconsistent across slots until a manual reset.

| Num | Direction | `msg` | `id` | Meaning |
|-----|-----------|-------|------|---------|
| 90150 | `[QS]sitA` slot 0 → all other `[QS]sitA` | `""` | `""` | Re-apply your sit-target now. |

This is a stock-AVsitter number used identically by QS.

## Clamp behavior

SL clamps sit-target offsets relative to the prim:

- **Ground prims (rezzed in-world):** ±1.7 m on any axis. Larger offsets are silently truncated.
- **Attached prims (worn HUD / attachment):** different clamps depending on attachment point.

The clamp is hard: there's no way around it from LSL except by linking additional prims at the position you want and setting the sit-target on those.

## Adjusting at the QS-extension level

Personal pose offsets ([Personal Pose Offsets](personal-pose-offsets.html)) sit **on top of** the sit-target + pose offset. They're per-user, stored in `QSO:*` LSD keys, applied at pose play time. The sit-target itself isn't touched; only `CURRENT_POSITION`/`CURRENT_ROTATION` shifts.

## See also

- [AVpos Reference](avpos-reference.html): `SET` and `DFLT` directives.
- [Adjustment Workflow](adjustment-workflow.html): `[HELPER]` → `[SITTARGET]` mode.
- [Personal Pose Offsets](personal-pose-offsets.html): per-user offsets layered on top.
- [Known Limits](known-limits.html): sit-target clamp details.
