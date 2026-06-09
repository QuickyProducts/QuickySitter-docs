---
title: QSALIVE Discovery (LinkMsg 90096 / 90097)
sidebar: home_sidebar
permalink: qsalive-discovery.html
keywords: qsalive, 90096, 90097, plugin discovery, presence
toc: true
---

Stock AVsitter plugins detect "is sitA in this prim?" and "how many sitter slots?" with `llGetInventoryType("[AV]sitA")` and a `while (llGetInventoryType("[AV]sitA " + (string)i) == INVENTORY_SCRIPT)` loop. QuickySitter's main script is `[QS]sitA`, so plugins that probe only the stock name see `INVENTORY_NONE` and bail even though sitA is sitting right next to them.

QSALIVE is the replacement: a **count / version / capabilities** query that **does not depend on script names**. Plugins ask the question, sitA answers it.

> **Not a presence handshake.** QSALIVE tells a plugin *how many sitter slots exist and what sitA supports* — it does not report which sibling plugins are loaded. Plugin presence is carried by the `qs:alive:*` LSD flags instead (see [Sibling presence](#sibling-presence) below).

## The query

| Num    | Direction              | `msg`                                    | `id` | Meaning |
|--------|------------------------|------------------------------------------|------|---------|
| 90096  | plugin → `[QS]sitA`    | `""`                                     | `""` | "How many sitters, what version, what caps?" |
| 90097  | `[QS]sitA` → plugin    | `<product>\|<ver>\|<sitters>\|<caps>`    | `""` | Count/version/caps reply. Also broadcast unsolicited from slot 0's `state_entry` once boot finishes. |

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
| `offsetlsd_v1` | `[QS]offset` supports persistent LSD storage at `QSO:<short>:<slot>:<pose>`. Gates migrations from older volatile-only releases. |

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

## Sibling presence

"Which sibling plugins are loaded?" is a **separate** question from QSALIVE, answered by `qs:alive:*` LSD flags rather than any link-message reply.

Each optional plugin writes a flag in its `state_entry`:

| Flag | Plugin | Gates |
|------|--------|-------|
| `qs:alive:prop` | `[QS]prop` | the `[PROP]` button in adjuster's menu. |
| `qs:alive:faces` | `[QS]faces` | the `[FACES]` / `[EXPRESSION]` menu items in sitA and adjuster. |
| `qs:alive:adjuster` | `[QS]adjuster` | the `[HELPER]` menu item in sitB. |
| `qs:alive:select` | `[QS]select` | select-driven menu routing in sitB (sitB also keeps an `[AV]select` inventory fallback for stock-AVsitter compat). |
| `qs:alive:rlv` | `[QS]root-RLV` | the RLV `Control…` gate in sitB. |
| `qs:offset:alive` | `[QS]offset` | personal-offset storage (note the **inverted** flag name). |

Menu builders read these flags **on demand** at menu-build time and never cache them. Removal is handled by `QS_ALIVE_CENSUS` (90079): `[QS]boot` wipes every `qs:alive:*` flag and broadcasts the census; surviving plugins re-stamp their flag, so a removed plugin simply fails to re-appear.

This mechanism is name-independent: a fork could rename `[QS]prop` to `[FOO]prop` and the `[PROP]` button still appears, because gating reads the `qs:alive:prop` flag the plugin wrote, not `llGetInventoryType`.

> **Retired (0.9951).** The per-plugin HELLO broadcasts `90088`–`90092` (`QS_OFFSET/PROP/FACES/ADJUSTER/SELECT_HELLO`) were the *old* presence mechanism and were replaced by these flags. Those numbers are reserved, not reused. The only remaining live HELLO is hudproxy's `90093`.

A couple of related link-messages are still presence-adjacent but are **not** plugin-alive flags:

| Num   | Sender | Purpose |
|-------|--------|---------|
| 90093 | `[QS]hudproxy` | Bidirectional probe with adjuster — see [HUD Integration](hud-integration.html). |
| 90094 / 90095 | `[QS]boot` ↔ DUMP plugins | QSDUMP — plugin announce for the DUMP cascade. |
| 90212 | plugin → `[QS]sitB` | QSPLUG_REGISTER — stateful registration of a plug-and-play `[OPTIONS]` menu button. See [Options Menu Plugins](options-menu-plugins.html). |

## Discovery vs. Integration

QSALIVE answers a *discovery* question: "is QuickySitter even here, and what does it support?" That's stateless — every probe is fresh, sitA has no list of who's asked, and the reply payload is read-only metadata.

Some plugins need a second step beyond discovery: they want to put a button into the furniture's menu. That's *integration*, and it's stateful — sitB has to remember which plugins registered, with which label, dispatched to which channel. QSPLUG_REGISTER (90212) is the integration channel; see [Options Menu Plugins](options-menu-plugins.html) for the full spec.

The two protocols complement each other:

| | **QSALIVE** | **QSPLUG_REGISTER** |
|---|---|---|
| **Layer** | Discovery | Integration |
| **Statefulness** | stateless probe + reply | stateful registry (sitB-side) |
| **Direction** | bidirectional | unidirectional (plugin → sitB) |
| **Use case** | "Should I activate at all?" | "Add my button to the menu." |

A plugin with UI typically uses **both**:

1. **QSALIVE** at startup to confirm QuickySitter is present (falls back to legacy AVsitter inventory probe if not — see boilerplate above).
2. **QSPLUG_REGISTER** to claim its `[OPTIONS]` menu slot.

The most important cross-wiring: **listen to 90097 and trigger your QSPLUG_REGISTER re-announce on it**. sitA's unsolicited 90097 broadcast is the cheapest possible "host just rebooted" signal — sitB likely went through its own `QS_BOOT_RELOAD` cascade and dropped your entry. One line in your `link_message` handler keeps the registry consistent for free.

A plugin **without UI** (logger, analytics, state mirror) only needs QSALIVE.

## See also

- [Options Menu Plugins](options-menu-plugins.html) — the integration counterpart, QSPLUG_REGISTER.
- [LinkMessage Numbers](linkmessage-numbers.html) — complete fork link-message map.
- [Boot Sequence](boot-sequence.html) — how boot uses QSALIVE for the self-check.
- [HUD Integration](hud-integration.html) — 90093, the QSALIVE-shaped probe for hudproxy.
