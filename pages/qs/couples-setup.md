---
title: Couples Setup
sidebar: home_sidebar
permalink: couples-setup.html
keywords: couples, two sitters, SYNC, sit target
toc: true
---

A couples furniture has two sitter slots and at least one SYNC pose pair so the two avatars animate in lockstep. QuickySitter handles couples setups identically to stock AVsitter 2. This page summarises the per-slot considerations and the QS-specific gotchas. For the tutorial walkthrough, see the [upstream AVsitter Couples page](https://avsitter.github.io/avsitter2_home.html).

> **Working alone?** The Creator Edition's [Animesh Adjust Dummies](quickysitter-pro-animesh.html) stand in for the missing partner while you set up and adjust SYNC poses, so no second avatar is needed.

## Script set

For a 2-sitter prim:

Mandatory:

- `[QS]boot` (one instance)
- `[QS]sitA` + `[QS]sitA 1` (two instances)
- `[QS]sitB` + `[QS]sitB 1` (matching count)
- an `AVpos` notecard

Optional (all presence-gated; add only what you need):

- `[QS]adjuster` (for `[HELPER]`)
- `[QS]select` (dedicated seat-select picker; sitB already has a built-in picker, so this is not required)
- `[QS]offset`, `[QS]prop`, `[QS]faces`, `[QS]sequence`

The script name suffix matches stock: space + number starting at 1 for the second instance (`[QS]sitA`, `[QS]sitA 1`, `[QS]sitA 2`, ...). The numbering must be contiguous; a gap (e.g. `[QS]sitA` + `[QS]sitA 2` without `[QS]sitA 1`) breaks the slot count.

## Sit-targets

Each slot has its own sit-target, derived from the slot's default pose. The `SET` directive is an ID used to pin seats to specific prims via prim descriptions (`<set>-<slot>`), not a count of arrangements. `SET` is a plain global directive, and there is no `SETUP` section.

See [SitTargets](sittargets.html) for the model and the pinning syntax.

## SYNC poses

For lockstep timing, both partners' poses must be declared with `SYNC` (not `POSE`) in the AVpos notecard. Each slot uses the same `<menu_name>` but a different animation file:

```
SITTER 0
SYNC Cuddle|cuddle_left
{Cuddle}<0.05, 0.0, 0.0><0.0, 0.0, 0.0>

SITTER 1
SYNC Cuddle|cuddle_right
{Cuddle}<-0.05, 0.0, 0.0><0.0, 0.0, 180.0>
```

When the menu selects "Cuddle," both sitA scripts play their respective `cuddle_left` / `cuddle_right` animations at their per-slot offsets.

The `{<name>}<pos><rot>` lines are normally written by `[HELPER] [SAVE]` / `[DUMP]` after you adjust positions in-world, so you don't usually type them by hand. See [AVpos Reference](avpos-reference.html) for the full grammar.

QS stores SYNC poses without the `P:` prefix in `qs:p:<ch>:<i>` LSD keys. `POSE` (solo) entries get the `P:` prefix to distinguish them. See [LSD Keys](lsd-keys.html).

## Re-Sync between viewers

The main couples-specific issue: SYNC poses drift between viewers over time, especially after camera operations. QuickySitter's solution is LinkMsg 90271, sent by a HUD or any in-prim script to re-phase all sitters in the same Sim frame. See [Re-Sync Protocol](resync-protocol.html).

For QuickyHUD users, the HUD has a SYNC button that fires 90271 on demand. For non-HUD setups, the trigger can be sent from a custom script:

```lsl
llMessageLinked(LINK_SET, 90271, "", "");
```

## Personal offsets per slot

A user might want their seated position shifted forward when sitting on slot 0 (e.g., the left side of the couch) but not when sitting on slot 1. `[QS]offset` keys are per-(user, slot, pose), so saves on slot 0 don't affect slot 1.

The all-poses fallback `M#T!` is also per-slot: a user can have a different "global" offset for each side of the couch.

See [Personal Pose Offsets](personal-pose-offsets.html).

## Common gotchas

- **Mismatched sitA/sitB counts.** `[QS]sitA` instances should match `[QS]sitB` instances. Boot's self-check hard-fails (ERROR) only when sitA or sitB is missing entirely. It does not raise a dedicated warning for a count mismatch, so double-check the instance counts yourself.
- **Same pose name in both slots.** Required: both slots have a pose named `Cuddle` so the menu can pick it as a single entry that triggers both. Different pose names in each slot don't pair up.
- **Animation drift after region restart.** Region restart preserves LSD, but viewer-side animation phase resets to `t=0`. A re-sync after sit-down restores phase coherence.
- **SWAP behavior.** The `SWAP` directive controls whether the menu allows users to swap positions (slot 0 ↔ slot 1). Stock semantics unchanged in QS.

## See also

- [3+ Avatars](multi-avatar.html): for setups with more than two sitters.
- [Re-Sync Protocol](resync-protocol.html): the SYNC drift fix.
- [Personal Pose Offsets](personal-pose-offsets.html): per-slot offset model.
- [AVpos Reference](avpos-reference.html): `SYNC` vs `POSE`.
- [Upstream AVsitter Couples](https://avsitter.github.io/avsitter2_home.html).
