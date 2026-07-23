---
title: SitTargets
sidebar: home_sidebar
permalink: sittargets.html
keywords: sit target, sittarget, set, dflt, default position
toc: true
---

A **sit-target** is the position and rotation the SL physics engine places an avatar at when they right-click → Sit. QuickySitter handles sit-targets identically to stock AVsitter 2, and this page covers the QS-relevant aspects. For the conceptual tutorial, see the [upstream AVsitter SitTargets page](https://avsitter.github.io/avsitter2_sittargets.html).

## The model

Each sitter slot (`[QS]sitA`, `[QS]sitA 1`, ...) gets its own sit-target, placed with `llLinkSitTarget` on the prim assigned to that slot.

The sit-target only controls the initial docking when the avatar sits down. Immediately afterwards the pose engine positions the avatar at the current pose's `{Pose}` coordinates (via `PRIM_POS_LOCAL` on the seated avatar). The pose position **replaces** the sit-target placement; it is not an offset added on top of it. Pose coordinates are always relative to the prim that holds the `[QS]` scripts, no matter which prim carries the sit-target.

QS derives each slot's sit-target from the slot's current default pose (lowered slightly so the avatar docks close to its final spot). Seats follow the poses automatically; there is no separate sit-target data to configure.

## The SET directive

`SET <n>` tags this installation's seat assignments with an ID. It is **not** a count of seating arrangements, and there is no menu to switch sets at runtime.

- Without a `SET` line (internal default -1), seats are **auto-assigned**: avatars are matched to free slots in link order, gender-aware for couples. The physical prim is decoupled from the logical slot.
- With `SET <n>`, pin each seat to a prim by putting `<n>-<slot>` in that prim's description (e.g. `0-1` = slot 1 of set 0). `-1` in a description excludes the prim from receiving a sit-target. Sitting on a pinned prim yields exactly that slot and its menu.

Upstream AVsitter uses distinct SET numbers to keep several independent script installations in one linkset from claiming each other's prims. QuickySitter supports one installation per linkset (single AVpos notecard, shared LinksetData store), so in QS the directive's practical use is pinning seats to prims: use a single `SET 0`.

## The DFLT directive

`DFLT` is unrelated to sets. It is a 0/1 flag that controls whether a seat reverts to its first pose when the last sitter stands up: `1` (the default) reverts; `0` keeps the last chosen `POSE` as the new default.

## Moving and showing sit-targets in-world

There is no separate sit-target editor. Sit-targets are derived from pose positions and recomputed at boot and whenever pose data changes. To move a seat, adjust the slot's poses (`[HELPER]` bar or the QuickyHUD adjust mode) and save; the derived sit-target follows the new default pose. To move a seat to a different prim, change the prim descriptions (see above) and reset.

With `[QS]adjuster` installed, the owner chat command `/5 targets` labels each assigned prim with floating text showing its `SET-SLOT` pair (link message 90298 to the sitA scripts).

## Re-placement on linkset changes (90150)

When the linkset's prim count changes (link/unlink), slot 0's `[QS]sitA` clears all sit-targets (prims marked `-1` are left alone) and broadcasts `90150`; every sitA slot then re-runs its assignment and re-places its own sit-target.

| Num | Direction | `msg` | `id` | Meaning |
|-----|-----------|-------|------|---------|
| 90150 | `[QS]sitA` slot 0 → all `[QS]sitA` | `""` | `""` | Re-assign and re-place your sit-target now. |

This is a stock-AVsitter number used identically by QS.

## Clamp behavior

SL clamps a sit-target offset to **±300 m per axis**; out-of-range values are rounded to the limit (see [llSitTarget on the SL wiki](https://wiki.secondlife.com/wiki/LlSitTarget)). This is practically irrelevant for furniture: seated avatars are positioned by the pose engine via `PRIM_POS_LOCAL`, which is not subject to the sit-target clamp.

## Adjusting at the QS-extension level

Personal pose offsets ([Personal Pose Offsets](personal-pose-offsets.html)) sit **on top of** the pose position. They're per-user, stored in `QSO:*` LSD keys, applied at pose play time. The sit-target itself isn't touched; only `CURRENT_POSITION`/`CURRENT_ROTATION` shifts.

## See also

- [AVpos Reference](avpos-reference.html): `SET` and `DFLT` directives.
- [Adjustment Workflow](adjustment-workflow.html): `[HELPER]` pose adjustment.
- [Personal Pose Offsets](personal-pose-offsets.html): per-user offsets layered on top.
- [Known Limits](known-limits.html): storage, HTTP and clamp limits.
