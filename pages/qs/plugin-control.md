---
title: '[AV]control plugins'
sidebar: home_sidebar
permalink: plugin-control.html
keywords: control, lockguard, lockmeister, xcite, rlv, plugin
toc: true
---

The control family splits two ways in QuickySitter. The **root-prim scripts** (`[QS]root`, `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV`) are **forked** (release 1.25). The **restraint / sensation protocol plugins** (LockGuard, LockMeister, Xcite!) are **genuinely stock AVsitter**, unforked: drop them into a QS prim and they work as in a stock AVsitter prim.

## Forked root-prim scripts (`[QS]root*`)

| Plugin | What it does |
|--------|--------------|
| `[QS]root` | Root-prim touch forwarder: forwards the touch (90005) when the touched prim has no sitA/menu of its own. |
| `[QS]root-control` | "Allow others to control the menu": couples controllers by name. |
| `[QS]root-security` | Sit/menu access gate (ALL / OWNER / GROUP); sends LinkMsg 90202 to sitA. Since 1.25 also a third **Adjust** ACL (OWNER / GROUP / ALL, default OWNER) set from the `[SECURITY]` menu and published to the `qs:sec:adjust` LSD key; `[QS]sitB` / `[QS]adjuster` read it to gate who may enter the adjust workflows (`[HELPER]` / `[QUICKYHUD]` and owner-gated registered `[ADJUST]` entries). Resets to OWNER on `CHANGED_OWNER`. |
| `[QS]root-RLV` | RLV capture/relay; publishes the `qs:alive:rlv` presence flag. |

There is **no** `[QS]root-RLV-extra` in the fork.

## Genuinely stock control plugins

| Plugin | What it does |
|--------|--------------|
| `[AV]LockGuard` | LockGuard restraints protocol. |
| `[AV]LockGuard-object` | LockGuard rezzed objects. |
| `[AV]LockMeister` | LockMeister restraints protocol. |
| `[AV]Xcite!` | Xcite! sensations. |

These (along with `[AV]camera` and `[AV]favs`) are shipped verbatim from upstream. Configuration of each plugin (channels, attachment points, RLV designations) is via the AVpos notecard or per-plugin notecards. See the [upstream AVcontrol documentation](https://avsitter.github.io/avsitter2_control.html).

## Why the split

The restraint/sensation plugins (LockGuard, LockMeister, Xcite!) don't read or write `qs:*` LSD keys and don't gate any sitter menu, so there's no functional benefit to forking them. QS treats them as a stable lower layer the fork cooperates with.

The root-prim scripts were forked because they sit in the QS control path: `[QS]root-RLV` publishes `qs:alive:rlv` so `[QS]sitB` can gate the `Control...` RLV menu via `rlv_present()` (with a `[AV]root-RLV` inventory probe as the stock-AVsitter fallback), and `[QS]root-security` reaches sitA over LinkMsg 90202. Their menu items *do* appear in the adjuster/options menus, which is exactly why presence gating matters.

## Interactions with QS

A few QS-specific behaviors worth noting:

- **`[QS]root-RLV` LinkMsg 90014** (controller + captives broadcast) is consumed by some QS scripts unchanged. RLV-based menu reset on capture still works.
- **`[QS]root-security` LinkMsg 90006** (touch/sit register) flows through QS sitA exactly as in stock; `[QS]root-security` also reaches sitA over LinkMsg 90202 for the access gate.
- **Stock `[AV]select` vs `[QS]select`.** `[AV]select` (stock multi-furniture select) and `[QS]select` (the QS fork) are interchangeable. `[QS]select` advertises presence by writing the `qs:alive:select` LSD flag (re-stamped on `QS_ALIVE_CENSUS` 90079, read on demand); `[QS]sitB`'s `select_present()` reads that flag and falls back to an `[AV]select` inventory probe for stock-AVsitter compat. (The retired `QS_SELECT_HELLO` on the `90088`–`90092` band is gone since 0.9951.)

## See also

- [Upstream AVcontrol documentation](https://avsitter.github.io/avsitter2_control.html).
- [Compatibility Matrix](compatibility-matrix.html): plugin compatibility table.
- [LinkMessage Numbers](linkmessage-numbers.html).
