---
title: QSALIVE Discovery (LinkMsg 90096 / 90097)
sidebar: home_sidebar
permalink: qsalive-discovery.html
keywords: qsalive, 90096, 90097, plugin discovery, presence
toc: true
---

Stock AVsitter plugins detect "is sitA in this prim?" and "how many sitter slots?" with `llGetInventoryType("[AV]sitA")` and a `while (llGetInventoryType("[AV]sitA " + (string)i) == INVENTORY_SCRIPT)` loop. QuickySitter's main script is `[QS]sitA`, so plugins that probe only the stock name see `INVENTORY_NONE` and bail even though sitA is sitting right next to them.

QSALIVE is the replacement: a **count / version / capabilities** query that **does not depend on script names**. Plugins ask the question, sitA answers it.

> **Not a presence handshake.** QSALIVE tells a plugin *how many sitter slots exist and what sitA supports*. It does not report which other plugins are loaded.

## The query

| Num    | Direction              | `msg`                                    | `id` | Meaning |
|--------|------------------------|------------------------------------------|------|---------|
| 90096  | plugin → `[QS]sitA`    | `""`                                     | `""` | "How many sitters, what version, what caps?" |
| 90097  | `[QS]sitA` → plugin    | `<product>\|<ver>\|<sitters>\|<caps>`    | `""` | Count/version/caps reply. Also broadcast unsolicited by slot 0 after every LSD (re)load: fresh boot, own reset, notecard re-seed. |

## Reply payload

Pipe-delimited. **Use `llParseString2List`, not `llParseStringKeepNulls`**: empty trailing fields would otherwise produce stale entries that confuse capability matching.

| Field | What it is | What your plugin does with it |
|-------|------------|-------------------------------|
| 0     | Product token: `QuickySitter` for this fork; other forks (or upstream) may set their own. | Identity check. Compare against the token you support and treat anything else as "QS not present". |
| 1     | Version string: mirrors the global `version` in `[QS]sitA.lsl`. | Diagnostics and support output only. Don't gate features on version comparisons, because that's what the capability tokens are for. |
| 2     | Sitter-slot count: the same number `get_number_of_scripts()` returns. | Use it directly instead of the legacy `[AV]sitA N` inventory loop. For most plugins this is the field that matters. |
| 3     | Capability CSV. | Feature discovery. Substring-match the tokens you need; ignore tokens you don't know. |

**Payload contract:** the field order is fixed, new fields are only ever appended, and the capability CSV only ever grows. Parse leniently: don't assume exactly four fields, and never treat an unknown capability token as an error.

### Capability tokens

Most current tokens are first-party plumbing: they exist so QS's own optional scripts and the QuickyHUD can detect features without script-name probes. Third-party plugins rarely need more than `dump90098`.

| Token | Meaning for a plugin author |
|-------|-----------------------------|
| `customs90260` | The personal-offset cache is live. First-party plumbing between `[QS]offset` and sitA. See [Personal Pose Offsets](personal-pose-offsets.html) if you need to interoperate. |
| `dump90098` | `[QS]boot` owns the DUMP cascade. A plugin that stores its own AVpos-relevant settings can join the dump via QSDUMP (90094/90095). See [Boot Sequence](boot-sequence.html). |
| `offsetlsd_v1` | `[QS]offset` persists offsets in LSD (instead of RAM only). A migration gate for offset-aware tooling; irrelevant to most plugins. |

## Who answers, when, and on which link

- Only the **slot-0** `[QS]sitA` answers 90096 (`if (SCRIPT_CHANNEL == 0)`), so a multi-sitter prim sends exactly one 90097 per probe, so plugins don't have to deduplicate.
- Both probe and reply use `LINK_SET` so plugins in child prims see them.
- Slot 0 emits one unsolicited 90097 at the end of every `qs_load_from_lsd()`, reached from `state_entry` when the linkset is already seeded (own reset), and from boot's `QS_BOOT_RELOAD` broadcast on a fresh boot and on every notecard re-seed. Plugins that came up before sitA missed any earlier replies; this lets them latch onto QS without sending a probe. Plugins that come up *after* sitA get their answer via the normal probe path.

## Adoption pattern for plugin authors

```lsl
integer QS_ALIVE   = FALSE;
integer QS_SITTERS = 0;

probe_qs()
{
    llMessageLinked(LINK_SET, 90096, "", "");
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
            // llParseString2List, NOT KeepNulls: empty trailing fields
            // must be dropped, not kept.
            list d = llParseString2List(msg, ["|"], []);
            QS_ALIVE   = (llList2String(d, 0) == "QuickySitter");
            QS_SITTERS = (integer)llList2String(d, 2);
            // ... wire up plugin state knowing sitA is here ...
        }
    }
}
```

The pattern is deliberately QS-native: no 90097 reply simply means "no QuickySitter here", and the plugin stays dormant. Whether to *also* support stock AVsitter (e.g. by treating a missing reply after a short timer as the cue to fall back to the legacy `[AV]sitA` inventory probe) is the plugin author's own product decision, out of scope for this page.

`changed(CHANGED_INVENTORY)` is a good place to re-run `probe_qs()` if the plugin needs to react to sitter-count changes. Slot 0 also re-emits 90097 on its own reset and after every notecard re-seed (each ends in a fresh LSD load), so the plugin can rely on either trigger.

## Discovery vs. Integration

QSALIVE answers a *discovery* question: "is QuickySitter even here, and what does it support?" That's stateless: every probe is fresh, sitA has no list of who's asked, and the reply payload is read-only metadata.

Some plugins need a second step beyond discovery: they want to put a button into the furniture's menu. That's *integration*, and it's stateful: sitB has to remember which plugins registered, with which label, dispatched to which channel. QSPLUG_REGISTER (90212) is the integration channel; see [Options Menu Plugins](options-menu-plugins.html) for the full spec.

The two protocols complement each other:

| | **QSALIVE** | **QSPLUG_REGISTER** |
|---|---|---|
| **Layer** | Discovery | Integration |
| **Statefulness** | stateless probe + reply | stateful registry (sitB-side) |
| **Direction** | bidirectional | unidirectional (plugin → sitB) |
| **Use case** | "Should I activate at all?" | "Add my button to the menu." |

A plugin with UI typically uses **both**:

1. **QSALIVE** at startup to confirm QuickySitter is present (see boilerplate above).
2. **QSPLUG_REGISTER** to claim its `[OPTIONS]` menu slot.

The most important cross-wiring: **listen to 90097 and trigger your QSPLUG_REGISTER re-announce on it**. Every event that can empty sitB's button registry ends in a 90097 broadcast: a full pack reset reloads sitA (unsolicited 90097), and after a sitB-only reset sitB probes 90096 itself, so sitA's reply reaches every plugin. Re-announcing is idempotent (sitB dedupes by script name), so one line in your `link_message` handler keeps the registry consistent for free.

A plugin **without UI** (logger, analytics, state mirror) only needs QSALIVE.

## See also

- [Options Menu Plugins](options-menu-plugins.html): the integration counterpart, QSPLUG_REGISTER.
- [LinkMessage Numbers](linkmessage-numbers.html): complete fork link-message map.
- [Boot Sequence](boot-sequence.html): how boot uses QSALIVE for the self-check.
- [HUD Integration](hud-integration.html): 90093, the QSALIVE-shaped probe for hudproxy.
