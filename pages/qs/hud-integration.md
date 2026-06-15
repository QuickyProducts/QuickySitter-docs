---
title: HUD Integration
sidebar: home_sidebar
permalink: hud-integration.html
keywords: hud, quickyhud, hudproxy, hudadmin, 90093, 90266, 90280
toc: true
---

QuickySitter is designed so that **HUD addons attach as seamless adjustment modules over the same LinkMsg surface that the built-in helper uses**. QuickyHUD is the reference HUD; third-party HUDs can speak the same protocol.

This page covers the integration contract — the message numbers, the presence-detection handshake, and the lifecycle. The HUD code itself lives in the separate QuickyHUD project.

## Why a HUD addon at all

Stock AVsitter's `[HELPER]` is a notecard-driven menu accessed by the creator via the prim's blue-dialog menu. It works, but every adjustment requires:

- A round-trip through the dialog menu UI
- Chat-channel typing for unsigned-integer offsets
- Repeated `[SAVE]` / `[CANCEL]` pressing

A wearable HUD with X+/Y+/Z+ buttons, Save/Reset, and a single visible offset display is dramatically faster for fine-tuning poses — especially when adjusting couple poses where both partners' positions matter and the creator wants live feedback.

QuickyHUD attaches non-destructively: on uninstall the prim falls back to the stock `[HELPER]` flow with no leftover state, and no end user ever sees evidence the HUD was there.

## Integration surface

Three components, three message numbers, plus the offset-storage protocol shared with `[QS]offset`.

| Component | Role | Talks via |
|-----------|------|-----------|
| `[QS]hudproxy` | In-prim presence + ADJUSTMODE state owner. Receives `90266`; **sends** `90271` (Re-Sync to sitA) and `90262` (save offset to `[QS]offset`); writes `QPP_CFG:ADJUSTMODE` LSD. | LinkMsg 90093 (presence), 90266 (mode flip), 90262 (save offset), 90271 (Re-Sync). |
| `[QS]hudadmin` | In-prim dynamic-prop attacher. Rezzes the QuickyHUD on sit / on manual "Quicky HUD" button. | LinkMsg 90280 (`QSPROP_ATTACH`). |
| QuickyHUD | The wearable. Talks to the in-prim hudproxy on its own comm channel. | Out-of-band; not part of this protocol. |

## HUDPROXY presence — 90093

QuickyHUD's `[QS]hudproxy` writes the `QPP_CFG:ADJUSTMODE` LSD key unprotected on its `state_entry`. `[QS]sitB` gates QuickyHUD-aware UI on key existence and value:

- sitB appends the `[QUICKYHUD]` button to the Adjust-dialog tail for the owner (gated on `qs:alive:adjuster` present and `qs:hud:unlicensed` not set).
- sitB enriches the main pose menu (`[NEW]`/`[DUMP]`/`[SAVE]`/`[DONE]`) if `value == "On"`.

**Problem.** LSD outlives script removal. If the creator removes hudproxy + hudadmin from the linkset after first install, the LSD key persists with whatever value it last had. sitB keeps showing `[QUICKYHUD]` (clicks no-op because nobody handles 90266) and stays stuck in the qh_on-enriched menu forever if the key happened to be `"On"` at removal time — including a `[DONE]` exit that can't clear the orphaned `"On"` state.

**Fix.** 90093 active-presence probe.

| Num | Direction | `msg` | `id` | Meaning |
|-----|-----------|-------|------|---------|
| 90093 | `[QS]adjuster` → hudproxy | `"PROBE"` | `""` | "Are you still there?" |
| 90093 | hudproxy → `[QS]adjuster` | `"HELLO"` | `<script_name>` | Presence reply. Also broadcast unsolicited from hudproxy's `state_entry`. |

### Adjuster side

```lsl
integer QS_HUDPROXY_HELLO = 90093;
integer hudproxy_present;

state_entry()
{
    hudproxy_present = FALSE;
    llMessageLinked(LINK_SET, QS_HUDPROXY_HELLO, "PROBE", "");
    llSetTimerEvent(1.0);
}

link_message(integer s, integer num, string msg, key id)
{
    if (num == QS_HUDPROXY_HELLO && msg == "HELLO")
    {
        hudproxy_present = TRUE;
        llSetTimerEvent(0.0);
        return;
    }
}

timer()
{
    llSetTimerEvent(0.0);
    if (!hudproxy_present)
        llLinksetDataDelete("QPP_CFG:ADJUSTMODE");
}
```

`changed(CHANGED_INVENTORY)` already calls `llResetScript()` in adjuster, so a script removal triggers a fresh `state_entry` → re-probe automatically. No separate inventory-change probe path needed.

### hudproxy side

```lsl
integer QS_HUDPROXY_HELLO = 90093;

state_entry()
{
    // ... existing init ...
    llMessageLinked(LINK_SET, QS_HUDPROXY_HELLO, "HELLO", llGetScriptName());
}

link_message(integer s, integer num, string str, key id)
{
    if (num == QS_HUDPROXY_HELLO && str == "PROBE")
    {
        llMessageLinked(LINK_SET, QS_HUDPROXY_HELLO, "HELLO", llGetScriptName());
        return;
    }
}
```

LSL suppresses self-delivery of `llMessageLinked` to the same script, so adjuster's own `"PROBE"` doesn't loop back into its 90093 handler. The `msg == "HELLO"` discriminator is defensive — if a future QuickyHUD script also writes to 90093, only HELLO messages set the flag.

## ADJUSTMODE flip — 90266

| Num   | Direction                | `msg`               | `id` | Meaning |
|-------|--------------------------|---------------------|------|---------|
| 90266 | `[QS]adjuster` → hudproxy | `"On"` / `"Off"`    | `llGetOwner()` (unused) | "Flip QuickyHUD ADJUSTMODE remotely." |

Sent from the `[HELPER]` choice dialog's "Quicky HUD" button (→ `"On"`), from `[STOP HELP]` (→ `"Off"`, routed back through `[HELPER]`), and from `end_helper_mode` auto-Off (→ `"Off"`, only when adjuster's local `helper_method == 1`). hudproxy mirrors the same `sAdjustmode` + LSD write its own settings menu performs; no confirmation dialog (the user already confirmed by clicking `[HELPER]`).

## Dynamic prop attach — `QSPROP_ATTACH` 90280

`[QS]prop` is a minimally-invasive fork of stock `[AV]prop` with one new link-message: a way to register and rez a prop **dynamically** without writing it into the `AVpos` notecard. Used by `[QS]hudadmin` to attach the QuickyHUD on sit / on the manual "Quicky HUD" button.

| Num | Direction | `msg` | `id` | Meaning |
|-----|-----------|-------|------|---------|
| 90280 | any in-prim source → `[QS]prop` | `<object>\|<type>\|<point>\|<sitter>\|<post_rez_say>` | sitter UUID | "Register this dynamic prop for the given sitter slot and rez it now. If a prior 90280 with the same `(sitter, object)` exists, update `point` + `post_rez_say` and re-rez." |

**Payload fields** (pipe-delimited, parse with `llParseString2List`):

| Field | Content |
|-------|---------|
| 0 | Object name in this prim's inventory. Must be an `INVENTORY_OBJECT`. |
| 1 | Stock `[AV]prop` type: `0` = ground prop (COPY-OK NEXT), `1` = attachment prop (COPY-TRANSFER NEXT), `2` = attachment prop personal, `3` = special. The HUD case is type `1`. |
| 2 | Attachment-point name (case-insensitive substring match into `ATTACH_POINTS` table). Empty string falls through to point `0` = "avatar center". For HUDs use e.g. `"HUD center"`. |
| 3 | Sitter slot index (0-based). Must be `< llGetListLength(SITTERS)`; out-of-range messages are silently dropped. |
| 4 | **Optional post-rez say.** Verbatim string `[QS]prop` will `llSay` on its `comm_channel` once the rezzed prop reports `REZ` back via the same channel. Empty = no extra message. hudadmin uses it to push `"*QUICKYTEXTURE*\|<uuid>"` to a freshly-rezzed QuickyHUD. |

The dynamic-prop entry is **stored** in the same `prop_triggers` / `prop_types` / `prop_objects` parallel lists that stock loads from `AVpos`. The trigger string is `<sitter>|<object>`, the prop group is `<sitter>|QSDYN`. Dedup is by trigger: re-issuing 90280 for the same `(sitter, object)` pair replaces the mutable fields and re-rezzes via the existing `rez_prop(idx)` path — no growth in the registry.

No new linkmsg is needed for cleanup. Stock `[AV]prop`'s 90065 (stand-up) handler already calls `remove_props_by_sitter(msg, FALSE)`, which wipes all non-type-3 entries matching the standing sitter — including dynamic ones.

## Re-Sync trigger — 90271

The HUD also owns Re-Sync policy. `[QS]sitA` exposes a single trigger; the HUD decides when to fire it. See [Re-Sync Protocol](resync-protocol.html) for the full design.

## Lifecycle summary

1. **Install.** Creator drops hudproxy + hudadmin into the prim. hudproxy's `state_entry` broadcasts an unsolicited HELLO (90093). Adjuster latches `hudproxy_present = TRUE`. sitA / sitB enable QuickyHUD menu items.
2. **Sit.** hudadmin sends 90280 to attach the wearable Pose-HUD. The user interacts with the HUD; X+/Y+/Z+ clicks go straight to `[QS]offset` via 90262.
3. **Stand-up.** Stock 90065 sweeps dynamic props. ADJUSTMODE stays as last set.
4. **Uninstall.** Creator removes hudproxy + hudadmin. Adjuster's next reset (via `CHANGED_INVENTORY`) probes 90093, gets no reply, deletes `QPP_CFG:ADJUSTMODE`. sitA / sitB stop offering QuickyHUD menu items.

The fact that ADJUSTMODE is unprotected (no `LSD_PASS`) and adjuster can delete it is a key design decision — it makes uninstall fully reversible without coordinated removal scripts.

## See also

- [QSALIVE Discovery](qsalive-discovery.html) — sibling presence protocol for plugin gating.
- [Re-Sync Protocol](resync-protocol.html) — the SYNC trigger HUD policy owns.
- [Personal Pose Offsets](personal-pose-offsets.html) — 90262 / 90264 lifecycle from the storage side.
