---
title: FAQ / Troubleshooting
sidebar: home_sidebar
permalink: faq.html
keywords: faq, troubleshooting, problems, help
toc: true
---

## My existing AVsitter prim: can I just swap in QS scripts?

Yes. The minimum change is to delete `[AV]sitA` / `[AV]sitB` and add `[QS]boot` + `[QS]sitA` + `[QS]sitB`. Those three QS scripts plus the existing `AVpos` notecard are the whole mandatory set; the notecard works unchanged. `[QS]select` is **optional**: only add it if you want the QS multi-seat picker (sitB has a built-in picker otherwise). See [Migration from AVsitter](migration.html) for the full procedure.

## Do I need to replace ALL the AVsitter plugins?

Not all of them. Stock `[AV]camera`, the lock/adult plugins (LockGuard, LockMeister, Xcite!) and `[AV]favs` work unchanged, because they are purely link-message-driven. `[AV]faces`, `[AV]select`, `[AV]adjuster`, `[AV]sequence`, `[AV]prop` and `[AV]root-RLV` find the engine via `[AV]sitA` script names and degrade in a QS linkset (single-sitter at best, no QS menu entries; `[AV]root-RLV` name-probes `[AV]sitA 1`, so on multi-sitter pieces its capture and seat-relocation misfire), so use their `[QS]` variants. See [Compatibility Matrix](compatibility-matrix.html).

## What happens to existing pose adjustments after migration?

Pose **defaults** (from the AVpos notecard) carry over fine, because boot re-parses on first run.

Pose **personal offsets** stored in stock `CUSTOMS` (set via `[SAVE]` in the `[ADJUST]` personal-adjust menu) live in the script's memory and are wiped when you replace the script. Users who saved personal positions in the old prim will need to re-save in the new one. This is the same as any AVsitter script reset.

If you install `[QS]offset`, future personal offsets persist across script resets and re-rezzes (LSD-backed). See [Personal Pose Offsets](personal-pose-offsets.html).

## Why does my prim's chat show `Load complete; 0 sitter(s) ready`?

On a fresh boot `[QS]boot` reports `Load complete; N sitter(s) ready.` (or `Cached boot; N sitter(s) ready.` on the skip-seed path). `N` is the number of **`SITTER` directives / pose sections boot parsed out of the `AVpos` notecard**, not a count of `[QS]sitA` scripts. `0` means boot found no seatable pose data: the `AVpos` notecard is empty, malformed, or has no `SITTER` / pose lines. Boot does **not** count sitA scripts to derive the channel count.

A missing or misnamed `[QS]sitA` is a separate failure: it surfaces as a `self_check_report` line (`ERROR: [QS]sitA missing`), not as a `0 sitter(s)` count. Script names are case-sensitive, so `[QS]SitA` (capital S in "Sit") won't be recognised there either.

## I changed the AVpos notecard but the new pose isn't appearing.

Three possibilities:

1. **You didn't save the notecard after editing.** SL viewer's notecard editor caches your edits until Save is hit. Without save, the asset-key doesn't change.
2. **You hit save but boot didn't notice.** `changed(CHANGED_INVENTORY)` should fire; if you suspect it didn't, reset `[QS]boot` manually.
3. **The notecard is large and the viewer editor dropped part of your edit on save.** See [Notecard size limits](known-limits.html#notecard-size-65536-bytes-per-line-cap-1024-bytes-and-the-viewer-editors-paste-cutoff). Reading is not the problem: a 50 KB notecard is read in full, and only a single line over 1024 bytes loses its tail. But the built-in editor has been seen to truncate large content when saving, so for a notecard around 50 KB, edit it externally and paste the whole thing back rather than making small in-world edits.

## Why doesn't the `[FACES]` / `[PROP]` button show up even though I have the scripts?

Each QS plugin advertises its presence by writing a `qs:alive:<name>` flag to Linkset Data in `state_entry` (`qs:alive:prop`, `qs:alive:faces`, `qs:alive:adjuster`, …; `[QS]offset` uses the inverted `qs:offset:alive`). `[QS]sitB` reads those flags on demand when it builds the menu, not script-name inventory. If the plugin is present but the button is missing, the script most likely crashed at `state_entry` (check chat for compile errors) so its flag was never written. Boot's `QS_ALIVE_CENSUS` (90079) broadcast asks every live plugin to re-stamp its `qs:alive:<name>` flag; `finalize_boot` sends it after each boot without wiping anything. The wipe-then-census (clear all `qs:alive:*`, then broadcast so a removed plugin drops out) runs **only** on the plugin add/remove path: a `CHANGED_INVENTORY` where the notecard is unchanged. The retired HELLO broadcasts (90088-90092) are no longer used. See [QSALIVE Discovery](qsalive-discovery.html).

## My couple pose drifts between sitters over time. What do I do?

Multi-avatar SYNC poses drift because viewers restart the looped animation locally on culling events (camera zoom, region crossing, draw-distance). The fix is the SYNC button on a QuickyHUD-compatible HUD, which sends LinkMsg 90271 to re-phase all sitters. See [Re-Sync Protocol](resync-protocol.html).

If you don't have QuickyHUD, any in-prim script can send the trigger:

```lsl
llMessageLinked(LINK_SET, 90271, "", "");
```

Any current `[QS]sitA` handles 90271 (the Re-Sync trigger has shipped since the unified-version release). Per-script versions drift between releases; a release stamps the whole set to one number, so "uniform version" only holds at release boundaries.

## Can I rename `[QS]sitA` to keep the AVsitter brand on the prim?

Yes for the publicly-visible script name, no for trademark reasons. You can rename QS scripts to whatever prefix you like: `[FOO]sitA`, `[Bar]sitB`, etc. Discovery is script-name-independent: presence is the `qs:alive:*` LSD flags (re-confirmed by the 90079 census), and the QSALIVE count/version handshake (90096/90097) keys off slot, not name. The one naming constraint is internal to the sitter pair: sitA counts its sibling slots by walking its **own** basename with contiguous ` 1`, ` 2`, … suffixes (so keep the numbering gap-free, second instance is ` 1`), and it locates its menu partner by scanning inventory for a script whose name contains `sitB`, so a renamed pair must share a prefix (or at least keep `sitB` in the menu script's name). Boot itself does no script-name counting.

You cannot, however, distribute renamed scripts as if they were the AVsitter or QuickySitter project. See the [TRADEMARK](https://avsitter.github.io/TRADEMARK.mediawiki) guidelines.

## Where do I report bugs?

[GitHub Issues on the QuickySitter repo](https://github.com/QuickyProducts/QuickySitter/issues) for code bugs.

[GitHub Issues on this docs repo](https://github.com/QuickyProducts/QuickySitter-docs/issues) for documentation bugs.

## Will stock AVsitter scripts run inside a QS linkset?

The link-message contracts at the plugin boundary are unchanged, so purely protocol-driven stock scripts (camera, the lock/adult plugins, `[AV]root-control`, `[AV]root-security`, favs) run as-is. Stock plugins that probe `[AV]sitA` script names for presence or sitter count (`[AV]faces`, `[AV]select`, `[AV]adjuster`, `[AV]sequence`, `[AV]prop`, and `[AV]root-RLV`) mis-detect the QS-named engine and degrade (`[AV]root-RLV`'s multi-sitter capture and seat-relocation misfire on a failed `[AV]sitA 1` probe), so use their `[QS]` variants. See [Compatibility Matrix](compatibility-matrix.html).

The reverse (QS scripts in a stock prim) does NOT work, because the QS scripts expect `[QS]boot` to have seeded LSD.

## I see `qs:*` entries in my LSD after removing all QS scripts. Are they safe?

Yes, they're inert. Stock AVsitter doesn't read them. If you want to clean up, `llLinksetDataDeleteFound("^qs:", "")` removes all of them. `QSO:*` (personal offsets) and `QPP_CFG:*` (HUD config) follow the same pattern with their own prefixes.

## See also

- [Known Limits](known-limits.html): hard limits in SL that QS works around but can't eliminate.
- [Support](support.html): where to get human help.
