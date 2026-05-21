---
title: FAQ / Troubleshooting
sidebar: home_sidebar
permalink: faq.html
keywords: faq, troubleshooting, problems, help
toc: true
---

## My existing AVsitter prim — can I just swap in QS scripts?

Yes. The minimum change is to delete `[AV]sitA` / `[AV]sitB` / `[AV]select` and add `[QS]boot` + `[QS]sitA` + `[QS]sitB` + `[QS]select`. The existing `AVpos` notecard works unchanged. See [Migration from AVsitter](migration.html) for the full procedure.

## Do I need to replace ALL the AVsitter plugins?

No. Stock `[AV]camera`, `[AV]control` (LockGuard, LockMeister, Xcite!, RLV), `[AV]favs` work unchanged inside a QS linkset. You only need the QS variants of plugins you want QS-specific features from. See [Compatibility Matrix](compatibility-matrix.html).

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

Each QS plugin announces its presence via a HELLO LinkMsg (90089 for prop, 90090 for faces, 90091 for adjuster, …). The button gating reads these flags, not script-name inventory. If the plugin is present but the button is missing, the script either crashed at `state_entry` (check chat for compile errors) or hasn't bumped to the required version. See [QSALIVE Discovery](qsalive-discovery.html).

## My couple pose drifts between sitters over time. What do I do?

Multi-avatar SYNC poses drift because viewers restart the looped animation locally on culling events (camera zoom, region crossing, draw-distance). The fix is the SYNC button on a QuickyHUD-compatible HUD, which sends LinkMsg 90271 to re-phase all sitters. See [Re-Sync Protocol](resync-protocol.html).

If you don't have QuickyHUD, any in-prim script can send the trigger:

```lsl
llMessageLinked(LINK_SET, 90271, "", "");
```

Sit-A 0.22+ required.

## Can I rename `[QS]sitA` to keep the AVsitter brand on the prim?

Yes for the publicly-visible script name, no for trademark reasons. You can rename QS scripts to whatever prefix you like — `[FOO]sitA`, `[Bar]sitB`, etc. The QSALIVE handshake (90096/90097) and sibling presence broadcasts (90089–90095) are all script-name-independent.

You cannot, however, distribute renamed scripts as if they were the AVsitter or QuickySitter project — see the [TRADEMARK](https://avsitter.github.io/TRADEMARK.mediawiki) guidelines.

## Where do I report bugs?

[GitHub Issues on the QuickySitter repo](https://github.com/QuickyProducts/QuickySitter/issues) for code bugs.

[GitHub Issues on this docs repo](https://github.com/QuickyProducts/QuickySitter-docs/issues) for documentation bugs.

## Will stock AVsitter scripts run inside a QS linkset?

Yes. Drop a stock `[AV]prop` into a QS prim and it works. The fork is structured so the link-message contracts at the plugin boundary are unchanged. See [Compatibility Matrix](compatibility-matrix.html).

The reverse — QS scripts in a stock prim — does NOT work, because the QS scripts expect `[QS]boot` to have seeded LSD.

## I see `qs:*` entries in my LSD after removing all QS scripts. Are they safe?

Yes, they're inert. Stock AVsitter doesn't read them. If you want to clean up, `llLinksetDataDeleteFound("^qs:", "")` removes all of them. `QSO:*` (personal offsets) and `QPP_CFG:*` (HUD config) follow the same pattern with their own prefixes.

## See also

- [Known Limits](known-limits.html) — hard limits in SL that QS works around but can't eliminate.
- [Support](support.html) — where to get human help.
