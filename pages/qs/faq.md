---
title: FAQ / Troubleshooting
sidebar: home_sidebar
permalink: faq.html
keywords: faq, troubleshooting, problems, help
toc: true
---

## My existing AVsitter prim — can I just swap in QS scripts?

Yes. The minimum change is to delete `[AV]sitA` / `[AV]sitB` and add `[QS]boot` + `[QS]sitA` + `[QS]sitB`. Those three QS scripts plus the existing `AVpos` notecard are the whole mandatory set; the notecard works unchanged. `[QS]select` is **optional** — only add it if you want the QS multi-seat picker (sitB has a built-in picker otherwise). See [Migration from AVsitter](migration.html) for the full procedure.

## Do I need to replace ALL the AVsitter plugins?

Not all of them. Stock `[AV]camera`, `[AV]control` (LockGuard, LockMeister, Xcite!, RLV) and `[AV]favs` work unchanged — they are purely link-message-driven. `[AV]faces`, `[AV]select`, `[AV]adjuster`, `[AV]sequence` and `[AV]prop` find the engine via `[AV]sitA` script names and degrade in a QS linkset (single-sitter at best, no QS menu entries) — use their `[QS]` variants. See [Compatibility Matrix](compatibility-matrix.html).

## What happens to existing pose adjustments after migration?

Pose **defaults** (from the AVpos notecard) carry over fine — boot re-parses on first run.

Pose **personal offsets** stored in stock `CUSTOMS` (set via `[ADJUSTER] [SAVE]`) live in the script's memory and are wiped when you replace the script. Users who had `[SAVE OFFSET]`-ed positions in the old prim will need to re-save in the new one. This is the same as any AVsitter script reset.

If you install `[QS]offset`, future personal offsets persist across script resets and re-rezzes (LSD-backed) — see [Personal Pose Offsets](personal-pose-offsets.html).

## Why does my prim's chat show `Boot complete: 0 channel(s) seeded`?

Most likely you have no `[QS]sitA` script in the prim, or its name doesn't match exactly. `[QS]boot` counts channels by finding `[QS]sitA`, `[QS]sitA 2`, `[QS]sitA 3`, … in inventory. Check the script names exactly — `[QS]SitA` (capital S in "Sit") won't match.

## I changed the AVpos notecard but the new pose isn't appearing.

Three possibilities:

1. **You didn't save the notecard after editing.** SL viewer's notecard editor caches your edits until Save is hit. Without save, the asset-key doesn't change.
2. **You hit save but boot didn't notice.** `changed(CHANGED_INVENTORY)` should fire; if you suspect it didn't, reset `[QS]boot` manually.
3. **You're hitting [the SL notecard 48 KB editor limit](known-limits.html#notecard-editor-cutoff).** Notecards larger than ~49 248 bytes are truncated in the viewer editor. The script can still read the full notecard up to 64 KiB, but you can't safely *edit* large notecards in-world. Edit externally and paste.

## Why doesn't the `[FACES]` / `[PROP]` button show up even though I have the scripts?

Each QS plugin advertises its presence by writing a `qs:alive:<name>` flag to Linkset Data in `state_entry` (`qs:alive:prop`, `qs:alive:faces`, `qs:alive:adjuster`, …; `[QS]offset` uses the inverted `qs:offset:alive`). `[QS]sitB` reads those flags on demand when it builds the menu — not script-name inventory. If the plugin is present but the button is missing, the script most likely crashed at `state_entry` (check chat for compile errors) so its flag was never written. Boot re-confirms the flags on every `QS_ALIVE_CENSUS` (90079): it wipes all `qs:alive:*`, broadcasts, and only live scripts re-stamp themselves. The retired HELLO broadcasts (90088–90092) are no longer used. See [QSALIVE Discovery](qsalive-discovery.html).

## My couple pose drifts between sitters over time. What do I do?

Multi-avatar SYNC poses drift because viewers restart the looped animation locally on culling events (camera zoom, region crossing, draw-distance). The fix is the SYNC button on a QuickyHUD-compatible HUD, which sends LinkMsg 90271 to re-phase all sitters. See [Re-Sync Protocol](resync-protocol.html).

If you don't have QuickyHUD, any in-prim script can send the trigger:

```lsl
llMessageLinked(LINK_SET, 90271, "", "");
```

Any current `[QS]sitA` handles 90271 (the Re-Sync trigger has shipped since the unified-version release; all shipped scripts are at the same locked version).

## Can I rename `[QS]sitA` to keep the AVsitter brand on the prim?

Yes for the publicly-visible script name, no for trademark reasons. You can rename QS scripts to whatever prefix you like — `[FOO]sitA`, `[Bar]sitB`, etc. Discovery is script-name-independent: presence is the `qs:alive:*` LSD flags (re-confirmed by the 90079 census), and the QSALIVE count/version handshake (90096/90097) keys off slot, not name. (The old name-matching that counted `[QS]sitA`, `[QS]sitA 2`, … in inventory is the one exception — see the `Boot complete` answer above.)

You cannot, however, distribute renamed scripts as if they were the AVsitter or QuickySitter project — see the [TRADEMARK](https://avsitter.github.io/TRADEMARK.mediawiki) guidelines.

## Where do I report bugs?

[GitHub Issues on the QuickySitter repo](https://github.com/QuickyProducts/QuickySitter/issues) for code bugs.

[GitHub Issues on this docs repo](https://github.com/QuickyProducts/QuickySitter-docs/issues) for documentation bugs.

## Will stock AVsitter scripts run inside a QS linkset?

The link-message contracts at the plugin boundary are unchanged, so purely protocol-driven stock scripts (camera, the control family, favs) run as-is. Stock plugins that probe `[AV]sitA` script names for presence or sitter count (`[AV]faces`, `[AV]select`, `[AV]adjuster`, `[AV]sequence`, `[AV]prop`) mis-detect the QS-named engine and degrade — use their `[QS]` variants. See [Compatibility Matrix](compatibility-matrix.html).

The reverse — QS scripts in a stock prim — does NOT work, because the QS scripts expect `[QS]boot` to have seeded LSD.

## I see `qs:*` entries in my LSD after removing all QS scripts. Are they safe?

Yes, they're inert. Stock AVsitter doesn't read them. If you want to clean up, `llLinksetDataDeleteFound("^qs:", "")` removes all of them. `QSO:*` (personal offsets) and `QPP_CFG:*` (HUD config) follow the same pattern with their own prefixes.

## See also

- [Known Limits](known-limits.html) — hard limits in SL that QS works around but can't eliminate.
- [Support](support.html) — where to get human help.
