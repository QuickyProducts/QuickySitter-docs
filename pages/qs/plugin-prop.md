---
title: '[QS]prop'
sidebar: home_sidebar
permalink: plugin-prop.html
keywords: prop, plugin, attachment, dynamic, QSPROP_ATTACH
toc: true
---

`[QS]prop` is a minimally-invasive fork of stock `[AV]prop` (AVsitter 2 / 2.2p04) that adds one new link-message, `QSPROP_ATTACH` (90280), to register and rez a prop dynamically without writing it into the AVpos notecard. Used by `[QS]hudadmin` to attach the wearable QuickyHUD on sit.

Everything else matches stock semantics exactly. Drop a stock `[AV]prop` into a QS prim and it works; drop `[QS]prop` into a stock-AVsitter prim and stock paths work, because the QS-specific 90280 handler is dormant when no one sends to it.

## Notecard syntax (unchanged from stock)

Props are declared with `PROP` / `PROP1` / `PROP2` / `PROP3` directives in `AVpos`, one per line with `|`-separated arguments:

```
PROP Read|paper|G1|<0.55, 0.0, 0.14>|<-134.0, 75.9, 44.8>
PROP1 Dine|knife|G1|<0.39, -0.31, 0.17>|<-0.02, 9.9, 90.1>|Right Hand
```

Arguments:

| Field | Content |
|-------|---------|
| 0 | `<trigger>`: pose name that triggers this prop. |
| 1 | `<object>`: inventory object name to rez. |
| 2 | `<group>`: prop group label (e.g. `G1`). Used to de-rez sibling props of the same group when the trigger changes. |
| 3 | `<pos>`: position vector `<x, y, z>`. |
| 4 | `<rot>`: rotation Euler `<x, y, z>` in degrees. |
| 5 | `<attach_point>` *(optional)*: attachment point name for `PROP1` / `PROP2` / `PROP3`. Empty for ground props. |

The directive name controls the prop-type semantic in `[QS]prop`:

| Directive | Internal type | Meaning |
|-----------|---------------|---------|
| `PROP`  | `0` | Ground prop (rezzed at the prim's position; no attachment). |
| `PROP1` | `1` | Attachment prop (auto-attaches to the sitter's `<attach_point>`). |
| `PROP2` | `2` | Attachment prop, personal (COPY-TRANSFER NEXT). |
| `PROP3` | `3` | Special: persists across pose changes. |

Full directive reference in the [upstream AVprop page](https://avsitter.github.io/avsitter2_prop.html).

## QS-specific addition: 90280 (QSPROP_ATTACH)

Dynamic prop attach **without** a notecard entry:

| Field | Content |
|-------|---------|
| 0 | Object name in this prim's inventory. Must be `INVENTORY_OBJECT`. |
| 1 | Stock type (0/1/2/3). |
| 2 | Attachment-point name (case-insensitive substring match). Empty falls through to point 0. |
| 3 | Sitter slot index (0-based). |
| 4 | Optional post-rez `llSay` message (for forwarding HUD-specific data like texture keys). |

Sender:

```lsl
llMessageLinked(LINK_SET, 90280, "MyHUD|1|HUD center|0|*QUICKYTEXTURE*|" + (string)tex_uuid, sitter_uuid);
```

Idempotent: re-issuing 90280 for the same `(sitter, object)` pair replaces the mutable fields (`point`, `post_rez_say`) and re-rezzes, with no growth in the prop registry.

Full protocol details in [HUD Integration § QSPROP_ATTACH](hud-integration.html#dynamic-prop-attach-qsprop_attach-90280).

## QS additions: presence + DUMP

`[QS]prop` also:

1. **Announces itself for QSDUMP** on 90095, which joins the DUMP cascade in `[QS]boot` so `[DUMP]` includes prop entries. See [Boot Sequence § QSDUMP](boot-sequence.html#qsdump-plugin-announce-for-the-dump-cascade).
2. **Publishes the `qs:alive:prop` LSD flag**, written early in `state_entry`, re-stamped on `QS_ALIVE_CENSUS` (90079), read on demand at menu-build, so `[QS]sitB` / `[QS]adjuster` can gate the `[PROP]` menu item without an inventory probe.

This is the script-name-independent presence pattern shared with `[QS]faces`, `[QS]adjuster`, `[QS]select` and `[QS]root-RLV`, all of which write their own `qs:alive:<name>` flag. (The earlier HELLO broadcasts on the `90088`–`90092` band were retired in 0.9951; the only live HELLO today is hudproxy's `90093`.) See [QSALIVE Discovery](qsalive-discovery.html).

## Stock-diff summary

Total changes from stock `[AV]prop` 2.2p04:

1. **Lazy-load prop DB in LSD (`qs:prop:*`).** The biggest real change: instead of holding the full prop table in RAM at all times, `[QS]prop` keeps the prop definitions in `qs:prop:*` LSD keys and reads them on demand, so heap stays flat regardless of how many `PROP` lines the notecard carries.
2. **Presence via the `qs:alive:prop` LSD flag, not script-name inventory probes.** Stock's `string main_script = "[AV]sitA";` and its `llGetInventoryType(main_script)` checks are gone. `[QS]prop` writes `qs:alive:prop` in `state_entry` and re-stamps it on `QS_ALIVE_CENSUS` (90079); menu gating reads it on demand. (QSALIVE 90096/90097 is sitter **count**/version/caps only and is answered by slot-0 sitA, not by plugins.)
3. **New global** `list prop_post_rez_say;` for the optional post-rez forward.
4. **One line in `dataserver` event** to keep `prop_post_rez_say` aligned with `prop_triggers`.
5. **One line in 90171/90173 handler** for the same alignment.
6. **Three lines in `listen()`'s REZ branch** to forward the post-rez say.
7. **New `link_message` handler block** for 90280 (≈40 lines).
8. **Version string + header comment block.**

Everything else verbatim from stock.

## Cleanup

No new linkmsg needed for prop removal. Stock `[AV]prop`'s 90065 (stand-up) handler already calls `remove_props_by_sitter(msg, FALSE)`, which wipes all non-type-3 entries matching the standing sitter, including dynamic ones from 90280.

## See also

- [HUD Integration](hud-integration.html): full QSPROP_ATTACH protocol with the QuickyHUD use case.
- [Upstream AVprop documentation](https://avsitter.github.io/avsitter2_prop.html): notecard syntax and behaviors.
- [QSALIVE Discovery](qsalive-discovery.html): sitter count/version discovery and the `qs:alive:*` presence model.
- [Boot Sequence](boot-sequence.html): QSDUMP cascade integration.
