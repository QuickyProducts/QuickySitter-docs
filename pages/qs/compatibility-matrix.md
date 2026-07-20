---
title: Compatibility Matrix
sidebar: home_sidebar
permalink: compatibility-matrix.html
keywords: compatibility, avsitter, stock, plugin, mix
toc: true
---

QuickySitter keeps the **link-message contract** that plugin scripts and **notecard consumers** see identical to stock AVsitter 2. The fork's structural changes happen below that surface. What the fork does **not** keep is script-**name**-based discovery: the engine scripts are named `[QS]sitA` / `[QS]sitB`, so a stock plugin that probes `llGetInventoryType("[AV]sitA")` for presence or walks `[AV]sitA N` names for the sitter count mis-detects the engine, and QS menu entries are gated on `qs:alive:*` flags that only `[QS]` plugins write. Purely protocol-driven plugins work unchanged. QS does fork the root family (`[QS]root`, `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV`), but the stock `[AV]` equivalents still run unchanged in a QS linkset.

## Direction by direction

| Scenario | Status | Notes |
|----------|--------|-------|
| **Stock AVsitter plugin in QuickySitter furniture** | ⚠️ Protocol-compatible, but name probes are not. | All stock link-message numbers QS receives are handled with the same semantics, so purely protocol-driven plugins (camera, control family, favs, …) work unchanged. Plugins that find the engine via `[AV]sitA` script names or need QS menu gating (faces, select, adjuster, sequence, prop) degrade. See the [plugin table](#plugin-compatibility-table). |
| **QuickySitter scripts in stock-AVsitter furniture** | ❌ Doesn't work. | sitA/sitB expect `qs:cfg`/`qs:sitter`/`qs:p:*` LSD keys that boot writes during seed; stock furniture has no `[QS]boot`. This is intentional, not a goal of the fork. |
| **Mixed (some [QS], some [AV] scripts in one prim)** | ✅ Works for the QS-script set documented below. | The fork is structured so that creators can adopt one QS script at a time. See "Minimal QS install" below. |
| **AVsitter notecard (AVpos) in QuickySitter furniture** | ✅ Reads stock AVpos directly. | Boot parses the unchanged AVpos format on first run, seeds LSD. No notecard migration needed. |
| **AVsitter notecard with QS-specific directives** | ✅ Forward-compat. | QS does not currently add new AVpos directives; any future additions will be additive (unknown directives ignored by stock AVsitter). |

## Minimal QS install

The smallest installation that gets QS benefits while leaving as much stock as possible:

- `[QS]sitA` + `[QS]sitB`: required. Replaces stock `[AV]sitA` + `[AV]sitB`. This is where the LSD migration happens.
- `[QS]boot`: required. The notecard-to-LSD seeder.
- an `AVpos` notecard: required. boot's self-check ERRORs (red hovertext) if it is missing; without it boot writes no LSD and sitA/sitB never leave their pre-boot state.

Camera, favs, texture, helperscript and the lock/adult plugins (LockMeister, LockGuard, Xcite!) can remain stock: they are purely protocol-driven (QS's unsolicited QSALIVE broadcasts reach them too; they ignore the unknown `num`). Select, adjuster, prop, faces, sequence and `[AV]root-RLV` are a different story: each derives engine presence and/or the sitter count from `[AV]sitA` script names that don't exist in a QS linkset, so they degrade to single-sitter behavior at best, and their QS menu entries stay hidden, because the gates read `qs:alive:*` flags only the `[QS]` variants write. That is exactly why these have `[QS]` forks; plan to swap them along with the base set. (`[QS]root-control` and `[QS]root-security` carry no sitter-name probe themselves, but the control suite addresses its members by name, so run them as a `[QS]` set rather than mixing with stock.)

> **Escape hatch: stock names.** QS scripts never locate each other by hardcoded name (each derives its sibling names from its own name at runtime), so a creator who must keep a legacy name-probing plugin can rename the base pair back to the stock names (`[QS]sitA` → `[AV]sitA`, numbered copies too, and `[QS]sitB` → `[AV]sitB`), and legacy presence probes and `[AV]sitA N` count walks find their targets again. This does **not** restore the flag-gated QS menu entries; those still need the `[QS]` plugin forks.

If you want the full QS feature set:

- `[QS]adjuster`: replaces `[AV]adjuster`. Gives you the on-the-fly `[HELPER] [SAVE]` writing into LSD, the 90263 customs-eviction protocol, and the `[QUICKYHUD]` button.
- `[QS]prop`: replaces `[AV]prop`. Adds the 90280 dynamic-prop attach (used by QuickyHUD).
- `[QS]faces`: replaces `[AV]faces`. Publishes `qs:alive:faces` so menu items gate cleanly.
- `[QS]select`: optional seat-select picker for multi-furniture setups (sitB has a built-in picker otherwise).
- `[QS]offset`: adds the dedicated personal-offset store with LSD persistence.
- `[QS]sequence`: replaces `[AV]sequence`.

## Plugin compatibility table

| Plugin | Stock works in QS? | QS variant? | QS-only features |
|--------|-------------------|-------------|------------------|
| `[AV]prop` | ⚠️ degraded | `[QS]prop` | Stock prop derives presence/sitter mapping from `[AV]sitA` names: pose-driven rezzing works for slot 0 / single-sitter at best, and boot's self-check WARNs "prop plugin missing" because `qs:alive:prop` is never written. `[QS]prop` adds `QSPROP_ATTACH` (90280) and the lazy `qs:prop:*` LSD store. |
| `[AV]faces` | ⚠️ degraded | `[QS]faces` | Stock faces counts sitters by walking `[AV]sitA N` names → faces play for sitter 0 at best, and the flag-gated `[FACES]` / `[FACE]` menu entries never appear. |
| `[AV]adjuster` | ⚠️ degraded | `[QS]adjuster` | The `[HELPER]` menu entry is gated on `qs:alive:adjuster`, which stock never writes, so the helper flow is unreachable from QS menus; stock's sitter-count walk also comes up empty. `[QS]adjuster` adds the LSD `[SAVE]`, the 90263 eviction protocol and `[QUICKYHUD]`. |
| `[AV]camera` | ✅ | none planned | Stock camera's only name-bound code path is dead code; all working paths are protocol-based. |
| `[AV]sequence` | ⚠️ single-sitter only | `[QS]sequence` | Stock counts sitters via `[AV]sitA N` names → slots ≥ 1 lose sequences. `[QS]sequence` takes the count from QSALIVE; reads its own `[AV]sequence_settings` notecard. |
| LockGuard / LockMeister / Xcite! | ✅ | none | All stock lock/Xcite controls work unchanged. |
| `[AV]root-RLV` | ⚠️ multi-sitter degraded | `[QS]root-RLV` | Stock RLV gates its multi-sitter capture and seat-relocation on a `[AV]sitA 1` name probe that fails here, so those misfire on multi-sitter pieces (basic RLV restraints still work). `[QS]root-RLV` uses the role count instead and publishes `qs:alive:rlv`. (sitB's `Control…` button has an `[AV]root-RLV` inventory fallback, so it still shows.) |
| `[AV]root-control` / `[AV]root-security` | ✅ | `[QS]root-control` / `[QS]root-security` | Logic works stock; the QS forks just retarget the suite's inter-script name couplings. Run all `[QS]` or all `[AV]`, and don't mix. |
| `[AV]favs` | ✅ | none | Stock favs works unchanged. |
| `[AV]helperscript` | ✅ | none (not packaged for QS) | Use the standard import flow. |
| `[AV]select` | ⚠️ effectively broken | `[QS]select` (optional) | sitB detects a stock `[AV]select` via a legacy inventory fallback and hands the seat menu over, but stock select's own `[AV]sitA N` count walk then sees one sitter, useless on the multi-sitter furniture it exists for. Use `[QS]select`, or sitB's built-in picker. |

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

This pattern works in any furniture: stock has no 90097 sender, so `QS_PRESENT` stays FALSE and the stock path runs.

## See also

- [QSALIVE Discovery](qsalive-discovery.html): how plugins detect QS.
- [LinkMessage Numbers](linkmessage-numbers.html): fork-specific numbers vs stock.
- [Migration from AVsitter](migration.html): script-by-script swap procedure.
