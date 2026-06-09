---
title: Compatibility Matrix
sidebar: home_sidebar
permalink: compatibility-matrix.html
keywords: compatibility, avsitter, stock, plugin, mix
toc: true
---

QuickySitter aims to keep the contract that **plugin scripts** (`[AV]prop`, `[AV]faces`, `[AV]camera`, `[AV]sequence`, `[AV]favs`) and **notecard consumers** see identical to stock AVsitter 2. The fork's structural changes happen below the link-message surface that plugins talk to. QS does fork the root family (`[QS]root`, `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV`), but the stock `[AV]` equivalents still run unchanged in a QS linkset.

## Direction by direction

| Scenario | Status | Notes |
|----------|--------|-------|
| **Stock AVsitter plugin in QuickySitter furniture** | ✅ Works unchanged. | Drop a stock `[AV]prop`, `[AV]faces`, etc. into a QS prim and it works identically to stock. All stock link-message numbers QS receives are handled with the same semantics. |
| **QuickySitter scripts in stock-AVsitter furniture** | ❌ Doesn't work. | sitA/sitB expect `qs:cfg`/`qs:sitter`/`qs:p:*` LSD keys that boot writes during seed; stock furniture has no `[QS]boot`. This is intentional, not a goal of the fork. |
| **Mixed (some [QS], some [AV] scripts in one prim)** | ✅ Works for the QS-script set documented below. | The fork is structured so that creators can adopt one QS script at a time. See "Minimal QS install" below. |
| **AVsitter notecard (AVpos) in QuickySitter furniture** | ✅ Reads stock AVpos directly. | Boot parses the unchanged AVpos format on first run, seeds LSD. No notecard migration needed. |
| **AVsitter notecard with QS-specific directives** | ✅ Forward-compat. | QS does not currently add new AVpos directives; any future additions will be additive (unknown directives ignored by stock AVsitter). |

## Minimal QS install

The smallest installation that gets QS benefits while leaving as much stock as possible:

- `[QS]sitA` + `[QS]sitB` — required. Replaces stock `[AV]sitA` + `[AV]sitB`. This is where the LSD migration happens.
- `[QS]boot` — required. The notecard-to-LSD seeder.
- an `AVpos` notecard — required. boot's self-check ERRORs (red hovertext) if it is missing; without it boot writes no LSD and sitA/sitB never leave their pre-boot state.

Everything else (select, adjuster, prop, faces, sequence) can remain stock. The fork's QSALIVE handshake (90096/90097) lets stock plugins keep working alongside the QS base set; the unsolicited reply from slot-0 sitA reaches stock plugins too — they ignore it via the `num` mismatch.

If you want the full QS feature set:

- `[QS]adjuster` — replaces `[AV]adjuster`. Gives you the on-the-fly `[HELPER] [SAVE]` writing into LSD, the 90263 customs-eviction protocol, and the `[QUICKYHUD]` button.
- `[QS]prop` — replaces `[AV]prop`. Adds the 90280 dynamic-prop attach (used by QuickyHUD).
- `[QS]faces` — replaces `[AV]faces`. Publishes `qs:alive:faces` so menu items gate cleanly.
- `[QS]select` — optional seat-select picker for multi-furniture setups (sitB has a built-in picker otherwise).
- `[QS]offset` — adds the dedicated personal-offset store with LSD persistence.
- `[QS]sequence` — replaces `[AV]sequence`.

## Plugin compatibility table

| Plugin | Stock works in QS? | QS variant? | QS-only features |
|--------|-------------------|-------------|------------------|
| `[AV]prop` | ✅ | `[QS]prop` | `QSPROP_ATTACH` (90280) for dynamic, no-notecard props; lazy `qs:prop:*` LSD store. Publishes `qs:alive:prop`. |
| `[AV]faces` | ✅ | `[QS]faces` | Publishes the `qs:alive:faces` presence flag. |
| `[AV]camera` | ✅ | none planned | Stock camera has no QS-specific code path. |
| `[AV]sequence` | ✅ | `[QS]sequence` | Multi-sitter fix via QSALIVE count. Reads its own `[AV]sequence_settings` notecard. |
| LockGuard / LockMeister / Xcite! | ✅ | none | All stock lock/Xcite controls work unchanged. |
| `[AV]control` family | ✅ | `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV` | QS forks the root family. `[QS]root-RLV` publishes `qs:alive:rlv`. The stock scripts still work as-is. |
| `[AV]favs` | ✅ | none | Stock favs works unchanged. |
| `[AV]helperscript` | ✅ | none (not packaged for QS) | Use the standard import flow. |
| `[AV]select` | ✅ | `[QS]select` (optional) | sitB has a built-in picker. It reads `qs:alive:select`, falling back to an `[AV]select` inventory probe for stock-AVsitter compat. |

## Detection rules for stock plugins that want QS support

A stock plugin can detect QS at runtime and adopt QS-specific code paths without breaking stock-AVsitter compatibility:

```lsl
integer QS_PRESENT = FALSE;

state_entry()
{
    llMessageLinked(LINK_SET, 90096, "", "");  // QSALIVE probe
    llSetTimerEvent(1.0);
}

link_message(integer s, integer num, string msg, key id)
{
    if (num == 90097)
    {
        list d = llParseString2List(msg, ["|"], []);
        QS_PRESENT = (llList2String(d, 0) == "QuickySitter");
        llSetTimerEvent(0.0);
        if (QS_PRESENT)
        {
            // Optional: parse capability CSV for finer gating.
            string caps = llList2String(d, 3);
            if (~llSubStringIndex(caps, "customs90260")) { /* ... */ }
        }
    }
}

timer()
{
    llSetTimerEvent(0.0);
    if (!QS_PRESENT)
    {
        // Stock-AVsitter path.
    }
}
```

This pattern works in any furniture — stock has no 90097 sender, so `QS_PRESENT` stays FALSE and the stock path runs.

## See also

- [QSALIVE Discovery](qsalive-discovery.html) — how plugins detect QS.
- [LinkMessage Numbers](linkmessage-numbers.html) — fork-specific numbers vs stock.
- [Migration from AVsitter](migration.html) — script-by-script swap procedure.
