---
title: '[AV]control plugins'
sidebar: home_sidebar
permalink: plugin-control.html
keywords: control, lockguard, lockmeister, xcite, rlv, plugin
toc: true
---

The `[AV]control` family of plugins — LockGuard, LockMeister, Xcite!, RLV — is **stock AVsitter** in QuickySitter linksets, unforked. Drop them into a QS prim and they work as if they were running in a stock AVsitter prim.

## Plugins in this family

| Plugin | What it does |
|--------|--------------|
| `[AV]root-control` | Owner/controller permission gating. The dispatcher for the family. |
| `[AV]root-security` | Touch and sit security; passes events to root-control or root-RLV. |
| `[AV]root-RLV` | Restricted Love Viewer integration. |
| `[AV]root-RLV-extra` | RLV restrict/dress sub-menus. |
| `[AV]LockGuard` | LockGuard restraints protocol. |
| `[AV]LockGuard-object` | LockGuard rezzed objects. |
| `[AV]LockMeister` | LockMeister restraints protocol. |
| `[AV]Xcite!` | Xcite! sensations. |

Configuration of each plugin (channels, attachment points, RLV designations) is via the AVpos notecard or per-plugin notecards. See the [upstream AVcontrol documentation](https://avsitter.github.io/avsitter2_control.html).

## Why these aren't forked

The control plugins:

- Don't read or write `qs:*` LSD keys.
- Don't depend on `[AV]sitA` / `[QS]sitA` script names (presence detection is via the QSALIVE-style 90201/90202 stock protocol).
- Don't show up in adjuster menus that would need QSALIVE gating.

So there's no functional benefit to forking them. QS treats them as a stable lower-layer that the fork's base scripts cooperate with.

## Interactions with QS

A few QS-specific behaviors worth noting:

- **`[AV]root-RLV` LinkMsg 90014** (controller + captives broadcast) is consumed by some QS scripts unchanged. RLV-based menu reset on capture still works.
- **`[AV]root-security` LinkMsg 90006** (touch/sit register) flows through QS sitA exactly as in stock.
- **Stock `[AV]select` vs `[QS]select`.** `[AV]select` (stock multi-furniture select) and `[QS]select` (QS version with QS_SELECT_HELLO 90092 broadcast) are interchangeable. sitB has both probe paths — the QS HELLO bit and the legacy inventory probe — for backward compat.

## See also

- [Upstream AVcontrol documentation](https://avsitter.github.io/avsitter2_control.html).
- [Compatibility Matrix](compatibility-matrix.html) — plugin compatibility table.
- [LinkMessage Numbers](linkmessage-numbers.html).
