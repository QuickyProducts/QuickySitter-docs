---
title: '[QS]prop'
sidebar: home_sidebar
permalink: plugin-prop.html
keywords: prop, plugin, attachment, dynamic, QSPROP_ATTACH, objectadjust, scale, worn fit
toc: true
---

`[QS]prop` is a minimally-invasive fork of stock `[AV]prop` (AVsitter 2 / 2.2p04) with two QS additions: a new link-message, `QSPROP_ATTACH` (90280), to register and rez a prop dynamically without writing it into the AVpos notecard (used by `[QS]hudadmin` to attach the wearable QuickyHUD on sit), and since 1.25 [prop scale & worn fit](#prop-scale-and-worn-fit-qsobjectadjust) together with the `[QS]objectadjust` companion script.

Everything else matches stock semantics exactly in the QS-to-stock direction: drop `[QS]prop` into a stock-AVsitter prim and stock paths work, because the QS-specific handlers are dormant when no one sends to them. The reverse is degraded: a stock `[AV]prop` in a QS prim still counts sitters with its script-name walk (`[AV]sitA N`), which finds nothing in the QS suite, so it only ever rezzes props for slot 0.

## Notecard syntax

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
| 5 | `<attach_point>` *(optional)*: attachment point name for `PROP1` / `PROP2` only. Empty for ground and world props (`PROP` / `PROP3`). |
| 6 | `<scale>` *(optional, QS 1.25+)*: uniform scale factor relative to the object's inventory size. Empty or `1` = unchanged. |
| 7 | `<wornpos>` *(optional, QS 1.25+)*: worn-fit position vector, local to the attach point. |
| 8 | `<wornrot>` *(optional, QS 1.25+)*: worn-fit rotation Euler in degrees, local to the attach point. |

Fields 6 to 8 are the persisted output of the [prop scale & worn fit](#prop-scale-and-worn-fit-qsobjectadjust) feature; `[DUMP]` writes them only as far as they are set. Stock `[AV]prop` ignores the extra fields.

The directive name controls the prop-type semantic in `[QS]prop`:

| Directive | Internal type | Meaning |
|-----------|---------------|---------|
| `PROP`  | `0` | Ground prop (rezzed at the prim's position; no attachment). |
| `PROP1` | `1` | Attachment prop, auto-attaches to the sitter's `<attach_point>` via the `ATTACHTO` handshake on sit. |
| `PROP2` | `2` | Attachment prop, attaches only when the sitter touches (clicks) the rezzed prop. |
| `PROP3` | `3` | World prop (no attachment) that persists across pose changes. |

`PROP1` and `PROP2` are both attachments and both require the rezzed object (and its contents) to be **COPY + TRANSFER**; the difference is only *how* they attach (auto-handshake vs. touch), so the COPY-TRANSFER requirement is not what distinguishes `PROP2`.

Full directive reference in the [upstream AVprop page](https://avsitter.github.io/avsitter2_prop.html).

## Prop scale and worn fit ([QS]objectadjust)

Since 1.25 a prop can carry a persisted size and, for attachment props, a worn fit (position and rotation on the body). The prop-side half is the **`[QS]objectadjust`** companion script, named after the stock `[AV]object` it ships beside: drop it into the prop's **root prim**, next to the untouched `[AV]object`. The furniture-side half is built into `[QS]prop`; no configuration is needed on either side.

What it enables:

- **Resize in the editor.** Stretch the rezzed prop with the normal viewer editor, then run `[SAVE]` (ADJUSTMODE or `[HELPER]`): the size is persisted and every future rez comes out at the saved size. No more take-back-and-replace loop. Scaling is uniform; a per-axis stretch is flattened to the X-axis ratio.
- **Fit attachment props on the body.** Wear the prop via its pose, adjust position and rotation in the editor, `[SAVE]`: the fit is stored relative to the attach point and re-applied on every future attach.
- **Touch fine-tuning for owners.** Touching a world-rezzed prop (types `PROP`/`PROP3`) as furniture owner opens a size menu: presets of ±1/5/10 % and `[RESTORE]` back to inventory size. Menu edits are per-rez unless persisted with `[SAVE]`. The menu exists for **world-rezzed props only**: an attached prop cannot be touched into the menu (clicks on worn attachments go to the wearer, not the script), so size and fit of attachment props are done in the viewer editor while worn, then saved.

The saved values live in the prop's database row, so they are per prop line (per sitter and trigger), and `[DUMP]` emits them as the optional notecard fields 6 to 8 above. The factor is always relative to the prop's inventory size, so `[RESTORE]` and factor `1` mean "as the object is in the furniture inventory".

Compatibility follows the stock promise in both directions: a prop **without** `[QS]objectadjust` simply rezzes unscaled and ignores the extra wire commands, and under stock `[AV]prop` the companion never receives them and stays passive. The wire commands (`QSSCALE`, `QSWORN`, `QSSAVESCALE`, `QSSAVEWORN`) are `llSay` on the prop `comm_channel` (20 m range, like the stock prop handshake); only the `[SAVE]`-time `PROPSEARCH` broadcast is `llRegionSay`. They are specified in [`qs/PROTOCOL.md` § Prop scale](https://github.com/QuickyProducts/QuickySitter/blob/master/qs/PROTOCOL.md) in the QuickySitter repository.

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
2. **Publishes the `qs:alive:prop` LSD flag**, written early in `state_entry`, re-stamped on `QS_ALIVE_CENSUS` (90079), read on demand. Only `[QS]adjuster` (to gate its `[PROP]` menu item) and `[QS]boot`'s self-check (to warn on a missing prop plugin) read this flag; `[QS]sitB` never reads it. No inventory probe is involved.

This is the script-name-independent presence pattern shared with `[QS]faces`, `[QS]adjuster`, `[QS]select` and `[QS]root-RLV`, all of which write their own `qs:alive:<name>` flag. (The earlier HELLO broadcasts on the `90088`–`90092` band were retired in 0.9951. The HELLO messages still in use are hudproxy's `90093`, the `QSDUMP_HELLO` `90095` that `[QS]prop` and `[QS]faces` send, and `[QS]sitB`'s `QS_SITB_HELLO` `90078`.) See [QSALIVE Discovery](qsalive-discovery.html).

## Stock-diff summary

Total changes from stock `[AV]prop` 2.2p04:

1. **Lazy-load prop DB in LSD (`qs:prop:*`).** The biggest real change: instead of holding the full prop table in RAM at all times, `[QS]prop` keeps the prop definitions in `qs:prop:*` LSD keys and reads them on demand, so heap stays flat regardless of how many `PROP` lines the notecard carries.
2. **Presence via the `qs:alive:prop` LSD flag, not script-name inventory probes.** Stock's `string main_script = "[AV]sitA";` and its `llGetInventoryType(main_script)` checks are gone. `[QS]prop` writes `qs:alive:prop` in `state_entry` and re-stamps it on `QS_ALIVE_CENSUS` (90079); menu gating reads it on demand. (QSALIVE 90096/90097 is sitter **count**/version/caps only and is answered by slot-0 sitA, not by plugins.)
3. **Post-rez say for 90280.** The optional post-rez `llSay` payload is stored as field 7 (`prs`) of the `qs:prop:<i>` LSD row and forwarded in the REZ handshake, so a dynamic 90280 attach can hand HUD-specific data (texture keys, etc.) to the freshly-rezzed object. (Pre-lazy-load builds carried this in a parallel `prop_post_rez_say` RAM list; that global is gone now that the whole prop DB lives in `qs:prop:*`.)
4. **New `link_message` handler block** for 90280 (≈40 lines).
5. **Prop scale & worn fit (1.25).** The `qs:prop:<i>` row grows to 11 fields (scale, wornpos, wornrot), the REZ handshake additionally sends `QSSCALE`/`QSWORN`, and the `[SAVE]`-triggered `PROPSEARCH` accepts `QSSAVESCALE`/`QSSAVEWORN` replies from `[QS]objectadjust`. Older 8/9-field rows stay readable.
6. **Version string + header comment block.**

Everything else verbatim from stock.

## Cleanup

No new linkmsg needed for prop removal. Stock `[AV]prop`'s 90065 (stand-up) handler already calls `remove_props_by_sitter(msg, FALSE)`, which wipes all non-type-3 entries matching the standing sitter, including dynamic ones from 90280.

## See also

- [HUD Integration](hud-integration.html): full QSPROP_ATTACH protocol with the QuickyHUD use case.
- [Upstream AVprop documentation](https://avsitter.github.io/avsitter2_prop.html): notecard syntax and behaviors.
- [QSALIVE Discovery](qsalive-discovery.html): sitter count/version discovery and the `qs:alive:*` presence model.
- [Boot Sequence](boot-sequence.html): QSDUMP cascade integration.
