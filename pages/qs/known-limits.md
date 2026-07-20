---
title: Known Limits
sidebar: home_sidebar
permalink: known-limits.html
keywords: limits, limitations, second life, mono, notecard, http, dump
toc: true
---

QuickySitter works around several Second Life script-runtime limits but can't eliminate them. This page catalogues the ones creators are most likely to hit.

## Mono 64 KB script-memory cap

LSL scripts compiled with Mono have a hard 64 KB cap on combined code + stack + heap.

**What QS does:** Moves `DATA_LIST` and `POS_ROT_LIST` out of `[QS]sitB`'s memory and into LSD (`qs:p:<ch>:<i>`). At 1000+ pose entries, stock AVsitter pushes past the cap; QS reads on demand and stays slim.

**What it still can't fix:** A single script's own logic + globals + local-variable peaks still must fit in 64 KB. Very large `[QS]adjuster` configurations (deep `ADJUST_MENU` trees, many custom strings) can push adjuster past the cap. If you see `Stack-Heap Collision` errors in chat, check whether any script is consistently above ~55 KB used. (The fork moved the `[DUMP]` pipeline out of `[QS]adjuster` and into `[QS]boot`, which frees a chunk of adjuster heap, and adjuster now only kicks the cascade with a single 90098 message.)

## LSD storage limits

Each linkset has a Linkset Data cap of **128 KiB** total. Keys and values both count toward that byte budget, and there is no separate key-count limit. Values are stored as strings. A typical `qs:p:*` pose row costs ~75–80 bytes (key + value), so the pool holds roughly **1,700 poses** if nothing else competes for it.

**What QS does:**

- `[QS]boot` writes the bulk of `qs:*` keys exactly once per fresh seed, so the writes are batched and don't hit the per-second write throttle.
- `[QS]offset` checks `llLinksetDataAvailable()` against `QPP_CFG:RESERVE` and `LSD_MIN_FREE_POSES × LSD_BYTES_PER_ENTRY` before each LSD write; if free space is too tight, the offset goes into the RAM tier instead. See [Personal Pose Offsets](personal-pose-offsets.html).

**What it can't fix:** The pool is shared: `qs:p:*` pose rows, `qs:prop:*` prop records, `QSO:*` personal offsets and QuickyHUD's `QPP_CFG:*` keys all draw from the same 128 KiB, so a pose-heavy build leaves less room for everything else. The 200-poses-of-headroom default for personal offsets is the practical safety margin.

## Notecard read limit (64 KiB) and editor cutoff (~48 KB)

LSL can read notecards up to 64 KiB via `llGetNotecardLine`. **The viewer's notecard editor truncates content past about 49 248 bytes**, so anything past that point exists but isn't visible or editable in-world.

**Practical impact:**

- Large AVpos notecards work at runtime (boot reads the full 64 KiB).
- They become unmaintainable in-world. Open them in the viewer editor, scroll to the end, and you see them mid-line.
- **Edit large AVpos notecards externally** and paste in via the viewer (CTRL-A → CTRL-V over the existing content).
- A bytes-per-pose-entry rule of thumb: roughly 80 – 120 bytes for a non-trivial entry with POS/ROT and a long animation name. 48 KB ≈ 400 – 600 entries before you cross the editor cutoff.

## Animation-loop drift between viewers

Multiple avatars in a SYNC pose can drift visually because the viewer determines loop phase locally at the `llStartAnimation` event, not from a network-wide source.

**What QS does:** `[QS]sitA` accepts LinkMsg 90271 to Stop+Sleep+Start every sitter's main pose in the same Sim frame, re-phasing them. Policy (when to fire) lives in HUD scripts. See [Re-Sync Protocol](resync-protocol.html).

**What it can't fix:** Two viewers running at very different framerates still won't be pixel-perfect synchronized, and the re-sync just brings the drift back to a few frames. Aggressive re-sync intervals (e.g., every 5 s) help but cause visible "restart" flickers; 30 s is the practical sweet spot.

## Permission-trigger races

`PERMISSION_TRIGGER_ANIMATION` is granted asynchronously. Between the sit event and the `run_time_permissions` callback, the script can't play animations.

**What QS does:** `[QS]sitA.run_time_permissions` sequences the work: first request RAM-tier personal-offset push (90261), then play pose (90000). The race between the two is handled by sitA's 90260 late-arrival re-apply.

**What it can't fix:** If the viewer denies permission entirely (rare; some custom viewers have this option), the sitter sits without animation and the menu still appears, and there's no way to re-request permission inside the same sit.

## Region restart and asset-key changes

A region restart preserves LSD for prims in the region. A re-rez of a saved prim creates a fresh inventory key for the notecard.

**What QS does:** `qs:boot:asset` stores the notecard's last-seen asset-key. On `state_entry`, boot compares against `llGetInventoryKey("AVpos")` and skips re-parse if they match, so live `[HELPER] [SAVE]` edits are preserved across region restart.

**What it can't fix:** Detaching and re-attaching the prim, or transferring to a different owner who re-imports the notecard, gives a fresh asset-key and re-parses. This is by design, because you do want to read the latest notecard text if the owner edited it externally.

## SL chat throughput

`llSay` / `llOwnerSay` / `llShout` are rate-limited per script per frame. `[DUMP]` of a large config can hit the limit and lose lines if dumped in a tight loop.

**What QS does:** `[QS]boot`'s dump cascade self-throttles via 90099 self-trigger between iterations, letting the Sim drain queued chat between batches. The output is also POST-uploaded to the AVsitter settings service (URL in the chat) so a large dump can be fetched from a URL instead of scraped out of chat, subject to its own HTTP throttle (next section).

## HTTP request throttle (the `[DUMP]` upload)

`[DUMP]` POSTs the rebuilt notecard to the AVsitter settings service so a large config can be fetched from a URL instead of scraped out of chat. Outbound `llHTTPRequest` is throttled to **25 requests per 20 s per object** (and 1000 per 20 s per owner).

**What QS does:** `[QS]boot` streams the upload in ~1 KB chunks (`web()` flushes once the escaped cache passes 1024 characters, one `llHTTPRequest` each) and **paces** them: between dump entries it arms a one-shot 0.2 s timer (`QS_DUMP_PACE`) instead of firing the next tick immediately, so the POSTs land well under SL's ~1/s rate and never burst toward the 25/20 s cap. It also **guards the throttle directly**: `llHTTPRequest` returns `NULL_KEY` synchronously when a POST is refused, and boot sets `dump_failed` on that (and on any non-200 `http_response`), so the owner gets a `[DUMP] Upload failed` notice instead of a silently truncated link. boot never reads the response body, only the status, so the 2 KB response-body limit doesn't apply, and the retrieval URL is built from the upload key.

**What it can't fix:** The 25/20 s budget is **per object, shared by every script in the linkset**. If something else in the prim is also firing HTTP requests, even the paced dump POSTs can be refused under contention. Boot flags `dump_failed` and warns, but the refused chunk is lost and the uploaded copy is incomplete. Recovery: re-run the dump, or use the loud `[HELPER]` `[DUMP]`, which also streams every line to local chat so you have the full text regardless of the upload.

## SitTarget bone offsets

SL's `llSitTarget` accepts an offset relative to the prim's pivot. The offset is **clamped** to ±1.7 m from the prim for ground prims (different behavior for attached prims). Larger offsets get silently truncated.

**Practical impact:** Pose adjustments via `[HELPER]` arrows are limited to within this clamp. If you need very long-range positioning, you'll need multiple sit-target prims or use SL's `llSetLinkPrimitiveParamsFast(LINK_THIS, [PRIM_POSITION, ...])` from a separate script.

## See also

- [LSD Storage](lsd-storage.html): the key layout that respects the 128 KB cap.
- [Personal Pose Offsets](personal-pose-offsets.html): the LSD-vs-RAM tier decision.
- [Re-Sync Protocol](resync-protocol.html): the 90271 trigger for SYNC-drift.
- [In-repo design docs](https://github.com/QuickyProducts/QuickySitter/tree/master/qs): `PROTOCOL.md`, `STORAGE.md`, `test/TESTPLAN.md`.
- [llHTTPRequest (SL Wiki)](https://wiki.secondlife.com/wiki/LlHTTPRequest): the authoritative outbound HTTP throttle and body limits.
