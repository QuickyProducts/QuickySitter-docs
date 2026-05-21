---
title: QSALIVE Discovery (LinkMsg 90096 / 90097)
sidebar: home_sidebar
permalink: qsalive-discovery.html
keywords: qsalive, 90096, 90097, plugin discovery, presence
toc: true
---

Stock AVsitter plugins detect "is sitA in this prim?" and "how many sitter slots?" with `llGetInventoryType("[AV]sitA")` and a `while (llGetInventoryType("[AV]sitA " + (string)i) == INVENTORY_SCRIPT)` loop. QuickySitter's main script is `[QS]sitA`, so plugins that probe only the stock name see `INVENTORY_NONE` and bail even though sitA is sitting right next to them.

QSALIVE is the replacement: a presence handshake that **does not depend on script names**. Plugins ask the question, sitA answers it.

## The handshake

| Num    | Direction              | `msg`                                    | `id` | Meaning |
|--------|------------------------|------------------------------------------|------|---------|
| 90096  | plugin → `[QS]sitA`    | `""`                                     | `""` | "Anyone here? Identify yourself." |
| 90097  | `[QS]sitA` → plugin    | `<product>\|<ver>\|<sitters>\|<caps>`    | `""` | Presence reply. Also broadcast unsolicited from slot 0's `state_entry` once boot finishes. |

## Reply payload

Pipe-delimited. **Use `llParseString2List`, not `llParseStringKeepNulls`** — empty trailing fields would otherwise produce stale entries that confuse capability matching.

| Field | Content |
|-------|---------|
| 0     | Product token. `QuickySitter` for this fork. Future forks (or upstream) may set their own. |
| 1     | Version string. Mirrors the global `version` in `[QS]sitA.lsl`. |
| 2     | Sitter-slot count, identical to `get_number_of_scripts()`. Plugins can use this directly instead of running the legacy inventory loop. |
| 3     | Capability CSV. Substring-match for individual features. Initial set: `customs90260`, `dump90098`, `offsetlsd_v1`. |

### Capability tokens

| Token | Meaning |
|-------|---------|
| `customs90260` | Personal-offset cache is available; plugin may request a push via 90261. See [Personal Pose Offsets](personal-pose-offsets.html). |
| `dump90098` | DUMP cascade is owned by `[QS]boot`; plugin may register via QSDUMP (90094/90095). |
| `offsetlsd_v1` | `[QS]offset` ≥ 0.04 supports persistent LSD storage at `QSO:<short>:<slot>:<pose>`. Gates migrations from older volatile-only releases. |

## Who answers, when, and on which link

- Only the **slot-0** `[QS]sitA` answers 90096 (`if (SCRIPT_CHANNEL == 0)`), so a multi-sitter prim sends exactly one 90097 per probe — plugins don't have to deduplicate.
- Both probe and reply use `LINK_SET` so plugins in child prims see them.
- On boot, slot 0 emits one unsolicited 90097 at the end of `state_entry` (after `boot_done = TRUE`). Plugins that came up before sitA missed any earlier replies; this lets them latch onto QS without sending a probe. Plugins that come up *after* sitA get their answer via the normal probe path.

## Adoption pattern for plugin authors

```lsl
integer QS_ALIVE   = FALSE;
integer QS_SITTERS = 0;

probe_qs()
{
    llMessageLinked(LINK_SET, 90096, "", "");
    llSetTimerEvent(1.0); // fallback after 1 s
}

default
{
    state_entry()
    {
        probe_qs();
    }

    link_message(integer sender, integer num, string msg, key id)
    {
        if (num == 90097)
        {
            // llParseString2List, NOT KeepNulls — empty trailing fields
            // must be dropped, not kept.
            list d = llParseString2List(msg, ["|"], []);
            QS_ALIVE   = (llList2String(d, 0) == "QuickySitter");
            QS_SITTERS = (integer)llList2String(d, 2);
            llSetTimerEvent(0.0);
            // ... wire up plugin state knowing sitA is here ...
        }
    }

    timer()
    {
        llSetTimerEvent(0.0);
        if (!QS_ALIVE)
        {
            // Fallback: stock inventory probe. Try the QS name first
            // (cheap), then the AV name for backward compat with stock
            // furniture.
            if (llGetInventoryType("[QS]sitA") == INVENTORY_SCRIPT
             || llGetInventoryType("[AV]sitA") == INVENTORY_SCRIPT)
            {
                // ... legacy slot-count loop here ...
            }
        }
    }
}
```

`changed(CHANGED_INVENTORY)` is a good place to re-run `probe_qs()` if the plugin needs to react to sitter-count changes. Slot 0 also re-emits 90097 on its own reset (state_entry runs again), so the plugin can rely on either trigger.

## Sibling presence protocols

QSALIVE inspired a small family of presence broadcasts inside the fork, all unsolicited HELLOs that gate menu items or DUMP routing without inventory probes:

| Num   | Sender | Purpose |
|-------|--------|---------|
| 90089 | `[QS]prop` | Gates the `[PROP]` button in adjuster's menu. |
| 90090 | `[QS]faces` | Gates the `[FACES]` / `[EXPRESSION]` menu items in sitA and adjuster. |
| 90091 | `[QS]adjuster` | Gates the `[HELPER]` menu item in sitA. |
| 90092 | `[QS]select` | Gates select-driven menu routing in sitB. |
| 90093 | `[QS]hudproxy` | Bidirectional probe with adjuster — see [HUD Integration](hud-integration.html). |
| 90094 / 90095 | `[QS]boot` ↔ DUMP plugins | QSDUMP — plugin announce for the DUMP cascade. |

All of them are name-independent: a fork could rename `[QS]prop` to `[FOO]prop` and the `[PROP]` button still appears, because gating reads the HELLO bit set by `link_message`, not `llGetInventoryType`.

## See also

- [LinkMessage Numbers](linkmessage-numbers.html) — complete fork link-message map.
- [Boot Sequence](boot-sequence.html) — how boot uses QSALIVE for the self-check.
- [HUD Integration](hud-integration.html) — 90093, the QSALIVE-shaped probe for hudproxy.
