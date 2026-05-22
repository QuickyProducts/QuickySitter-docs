---
title: '[QS]prop'
sidebar: home_sidebar
permalink: plugin-prop.html
keywords: prop, plugin, attachment, dynamic, QSPROP_ATTACH
toc: true
---

`[QS]prop` is a minimally-invasive fork of stock `[AV]prop` (AVsitter 2 / 2.2p04) that adds one new link-message — `QSPROP_ATTACH` (90280) — to register and rez a prop dynamically without writing it into the AVpos notecard. Used by `[QS]hudadmin` to attach the wearable Quicky-Pose-HUD on sit.

Everything else matches stock semantics exactly. Drop a stock `[AV]prop` into a QS prim and it works; drop `[QS]prop` into a stock-AVsitter prim and stock paths work — the QS-specific 90280 handler is dormant when no one sends to it.

## Notecard syntax (unchanged from stock)

Props are declared with `PROP` / `PROP1` / `PROP2` / `PROP3` directives in `AVpos`, one per line with `|`-separated arguments:

```
PROP Read|paper|G1|<0.55, 0.0, 0.14>|<-134.0, 75.9, 44.8>
PROP1 Dine|knife|G1|<0.39, -0.31, 0.17>|<-0.02, 9.9, 90.1>|Right Hand
```

Arguments:

| Field | Content |
|-------|---------|
| 0 | `<trigger>` — pose name that triggers this prop. |
| 1 | `<object>` — inventory object name to rez. |
| 2 | `<group>` — prop group label (e.g. `G1`). Used to de-rez sibling props of the same group when the trigger changes. |
| 3 | `<pos>` — position vector `<x, y, z>`. |
| 4 | `<rot>` — rotation Euler `<x, y, z>` in degrees. |
| 5 | `<attach_point>` *(optional)* — attachment point name for `PROP1` / `PROP2` / `PROP3`. Empty for ground props. |

The directive name controls the prop-type semantic in `[QS]prop`:

| Directive | Internal type | Meaning |
|-----------|---------------|---------|
| `PROP`  | `0` | Ground prop (rezzed at the prim's position; no attachment). |
| `PROP1` | `1` | Attachment prop (auto-attaches to the sitter's `<attach_point>`). |
| `PROP2` | `2` | Attachment prop, personal (COPY-TRANSFER NEXT). |
| `PROP3` | `3` | Special — persists across pose changes. |

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

Idempotent: re-issuing 90280 for the same `(sitter, object)` pair replaces the mutable fields (`point`, `post_rez_say`) and re-rezzes — no growth in the prop registry.

Full protocol details in [HUD Integration § QSPROP_ATTACH](hud-integration.html#dynamic-prop-attach--qsprop_attach-90280).

## QS additions: presence broadcasts

`[QS]prop` (≥ 0.020) also:

1. **Announces itself for QSDUMP** on 90095 — joins the DUMP cascade in `[QS]boot` so `[DUMP]` includes prop entries. See [Boot Sequence § QSDUMP](boot-sequence.html#qsdump--plugin-announce-for-the-dump-cascade).
2. **Broadcasts `QS_PROP_HELLO` (90089)** on `state_entry` / `on_rez` / QSALIVE-reply, so `[QS]adjuster` can gate the `[PROP]` menu item without an inventory probe. id = announcer's script name.

This is the script-name-independent presence pattern shared with `[QS]faces` (90090), `[QS]adjuster` (90091), `[QS]select` (90092), and hudproxy (90093). See [QSALIVE Discovery](qsalive-discovery.html).

## Stock-diff summary

Total changes from stock `[AV]prop` 2.2p04:

1. **Sitter presence via QSALIVE, not script-name inventory probes.** Stock's `string main_script = "[AV]sitA";` and its `llGetInventoryType(main_script)` checks are gone. Replaced by `qs_alive` + `qs_sitter_count_cached`, populated by a 90096 probe in `state_entry` / `on_rez` / `changed(CHANGED_INVENTORY)`.
2. **New global** `list prop_post_rez_say;` for the optional post-rez forward.
3. **One line in `dataserver` event** to keep `prop_post_rez_say` aligned with `prop_triggers`.
4. **One line in 90171/90173 handler** for the same alignment.
5. **Three lines in `listen()`'s REZ branch** to forward the post-rez say.
6. **New `link_message` handler block** for 90280 (≈40 lines) and QSALIVE reply (≈15 lines).
7. **Version string + header comment block.**

Everything else verbatim from stock.

## Cleanup

No new linkmsg needed for prop removal. Stock `[AV]prop`'s 90065 (stand-up) handler already calls `remove_props_by_sitter(msg, FALSE)`, which wipes all non-type-3 entries matching the standing sitter — including dynamic ones from 90280.

## See also

- [HUD Integration](hud-integration.html) — full QSPROP_ATTACH protocol with the QuickyHUD use case.
- [Upstream AVprop documentation](https://avsitter.github.io/avsitter2_prop.html) — notecard syntax and behaviors.
- [QSALIVE Discovery](qsalive-discovery.html) — the presence handshake.
- [Boot Sequence](boot-sequence.html) — QSDUMP cascade integration.
