---
title: Personal Pose Offsets ([QS]offset)
sidebar: home_sidebar
permalink: personal-pose-offsets.html
keywords: offset, customs, personal, ssot, QSO
toc: true
---

QuickySitter moves personal (per-user, per-slot) pose offsets out of `[AV]sitA`'s inline `CUSTOMS` list into a dedicated `[QS]offset` script with a two-tier store. The slot is in the key because SYNC couple poses share a pose name across multiple slots, but each slot has its own DEFAULT (sit-target offset relative to root); a flat `(user, pose)` key would let a save on slot 1 overwrite a save on slot 0 for the same pose name.

`[QS]offset` is **optional and presence-gated** (it publishes the inverted flag `qs:offset:alive`). It is also the *only* personal-offset store: with no `[QS]offset` in the linkset there is **no persistent offset storage and no sitA fallback**: saves have nowhere to go and seated avatars always land on the pose DEFAULT.

## Storage tiers (owned exclusively by `[QS]offset`)

### LSD tier: `QSO:<short>:<slot>:<pose>`

Persistent across script reset and re-rez. Used while LSD has at least `LSD_MIN_FREE_POSES` worth of free space left past the `QPP_CFG:RESERVE` budget that the HUD sets. (This LSD floor is independent of the 200-entry RAM cap below, so don't conflate the two.)

Keys are written **unprotected**: the proprietary QuickyHUD `LSD_PASS` is intentionally absent from this MPL-licensed source; `QPP_CFG:*` keys (license, reserve, migration flag) stay protected on the QuickyHUD side. Pose offsets aren't security-sensitive, so unprotected reads/writes are acceptable.

`QPP_CFG:ADJUSTMODE` is the deliberate exception, unprotected by design because `[QS]adjuster` reads it (capability detection via `llLinksetDataFindKeys`, plus the state checks in its adjust gates) and writes it via the 90266 link-message; `[QS]sitB` also reads it to enrich its menu state. `[QS]hudadmin` migrates the key from its old protected form on init (`migrateAdjustmodeToUnprotected`, idempotent); hudproxy only does a protected-read fallback.

### RAM tier: `CUSTOMS` list

Volatile overflow, LRU-evicted at 200 entries. Used when LSD is too tight (below the `LSD_MIN_FREE_POSES` floor plus `QPP_CFG:RESERVE`), or in legacy / stock AVsitter setups where there's no reserve to honor.

Stride is 5: `[pose, short, slot, pos, rot]` per entry.

Defensive caps:

- `LRU_CAP = 200`: hard cap on entries. Picked so that `200 × ~150` bytes worst-case + ~12 KB script code/state stays well under Mono's 64 KB cap. Front-evicted by `cull_to_cap` after each save (single batch `llDeleteSubList`).
- `EMERGENCY_FREE_BYTES = 3000`: `save_offset` calls `emergency_shrink()` *before* the `+=` and evicts one entry at a time until free memory ≥ this threshold or the list is empty. Defends against Stack-Heap Collision if the per-entry estimate diverges from reality (very long Unicode pose names, heap fragmentation from other scripts).

## Single source of truth

`[QS]offset` is the **sole owner** of both tiers. `[QS]sitA` holds **no authoritative copy**: it reads LSD directly for the LSD tier, and mirrors only the RAM tier in a session-local list (`RAM_OVERFLOW`) populated by 90260 push. The mirror is fully replaced/cleared on sit-down (90261 request), CLEAR (90265 broadcast), and stand-up.

This eliminates the cache-coherence problem that the previous `MY_CUSTOMS`-as-full-cache design had: any LSD mutation in `[QS]offset` is automatically visible to `apply_current_anim`'s next read, no invalidation broadcast needed for LSD-tier values. Only the small RAM-tier subset has push-based invalidation, with three well-defined events: 90260 push for save, 90263 for adjuster overwrite, 90265 for full wipe.

## Read path in `[QS]sitA.apply_current_anim`

```
1. Build key = "QSO:" + llGetSubString(MY_SITTER, 0, 7) + ":"
              + (string)SCRIPT_CHANNEL + ":" + CURRENT_POSE_NAME
2. Read LSD at key. If non-empty → parse "<pos>|<rot>", apply, done.
3. Look up CURRENT_POSE_NAME in RAM_OVERFLOW. If found → apply, done.
4. Read LSD at "QSO:<short>:<slot>:M#T!". If non-empty → apply, done.
5. Look up "M#T!" in RAM_OVERFLOW. If found → apply, done.
6. No personal offset.
```

LSD reads are Mono hashmap lookups (~50 µs); the four-read worst case stays well under one Sim frame. Pose-specific entries always win over `M#T!` (the all-poses fallback), regardless of which tier they're in.

## Write path

`save_offset` writes to LSD when `lsdHasRoom()` returns TRUE, otherwise to RAM `CUSTOMS`. When the write went to RAM (not LSD), `save_offset` also fires 90260 to the originating sitA so its `RAM_OVERFLOW` mirror stays in sync immediately, because sitA wouldn't see the value otherwise (it only direct-reads LSD).

`push_customs_for(sitter, slot)` (the 90261 handler) enumerates **only the RAM tier** and emits one 90260 per matching `(user_short, slot, pose)` entry. LSD entries are not pushed because sitA reads them directly on demand. The slot filter in the lookup ensures each sitA's `RAM_OVERFLOW` only ever contains its own slot's data.

## Link messages

| Num    | Direction | `msg` | `id` | Meaning |
|--------|-----------|-------|------|---------|
| 90260 | `[QS]offset` → `[QS]sitA` + `[QS]hudproxy` | `pose_name\|pos\|rot` | sitter UUID | "Mirror this RAM-tier personal offset into your local cache." Sent once per matching RAM-tier entry when a sitter sits, once per RAM-tier `save_offset`. **ZERO/ZERO is the delete sentinel**: receivers drop the matching entry. |
| 90261 | `[QS]sitA` → `[QS]offset` | `(string)slot` | sitter UUID | "Push every RAM-tier cached offset for this (sitter, slot) pair to me." Sent on sit and on hudproxy pose change. Only enumerates `CUSTOMS` (RAM tier). |
| 90262 | `[QS]sitA` + `[QS]hudproxy` → `[QS]offset` | `slot\|pose_name\|pos\|rot` | sitter UUID | "Save this offset for (sitter, slot, pose)." Magic name `M#T!` is the all-poses offset saved by the sitter's `[OFFSET ALL]` button (which first asks for an `[ALL POSES]` confirm). |
| 90263 | `[QS]adjuster` → `[QS]sitA` + `[QS]offset` | `(string)sitter_slot` | pose_name (as `key`) | "The creator just overwrote this pose's default on this slot via `[HELPER] [SAVE]`. Drop every pose-specific entry on this slot that matches. `M#T!` survives, and other slots keep their offsets." |
| 90264 | hudproxy → `[QS]offset` | `""` | ignored | "Wipe ALL personal offsets: both LSD `QSO:*` and RAM `CUSTOMS`." Triggered by the HUD settings menu's `CLEAR offset storage` confirm. |
| 90265 | `[QS]offset` → all `[QS]sitA` + `[QS]hudproxy` | `""` | `NULL_KEY` | "Clear your RAM-tier mirror." Broadcast on `wipe_all_offsets` (90264 follow-up). LSD-tier values don't need invalidation. |

## Why 90263 exists

In stock AVsitter, pressing `[SAVE]` in the helper-bar adjuster only updates the pose default in memory; the currently seated avatar is **not** repositioned live, so the stale `[pose, user_short]` `CUSTOMS` entries never get a chance to re-apply on top of the new default.

QuickySitter's `[QS]sitB` 90301 handler forwards the new pos/rot straight from the 90301 payload to `[QS]sitA` via 90055, so the seated avatar reflects the new default immediately (better UX). It deliberately does **not** call `send_anim_info()` / route through `apply_current_anim`: re-reading LSD there would race with the adjuster's save and could re-apply a stale personal offset on top of the new default (a visible "snap"). There is no `MY_CUSTOMS`; personal offsets live only in the `QSO:*` LSD tier and the `RAM_OVERFLOW` mirror.

90263 is sent by `[QS]adjuster` **before** 90301 in the `[SAVE]` loop, so sitA and `[QS]offset` drop the matching pose-specific personal offsets on that slot ahead of the 90055 re-apply. The seated avatar lands on the helper-bar position; future re-sits start from the new default with no carry-over offset.

`M#T!` (the all-poses personal offset) is intentionally preserved: it isn't tied to the saved pose name, and the user's intent ("I always sit X cm forward") still applies after a default change.

## 90260 late-arrival re-apply (RAM-tier only)

`run_time_permissions` in `[QS]sitA` fires 90261 (request RAM-tier push) and 90000 (play pose) back-to-back when an avatar sits. The two messages race two independent round-trips:

- 90261 → `[QS]offset` → 90260 (one per matching RAM-tier entry)
- 90000 → `[QS]sitB` → 90055 → `apply_current_anim` reads LSD direct

For LSD-tier offsets the race is gone post-SSoT-refactor: `apply_current_anim` reads the LSD value synchronously inside the handler, so winning or losing the 90260 race no longer matters.

For RAM-tier offsets the race is still possible: if 90055 wins, `apply_current_anim` reads LSD (miss), checks `RAM_OVERFLOW` (empty until 90260 arrives), and lands on `DEFAULT_POSITION`. The 90260 then populates `RAM_OVERFLOW`, and nobody re-applies, so the RAM-tier offset is silently ignored.

`[QS]sitA`'s 90260 handler resolves this by re-applying the offset inside the handler when CURRENT still equals DEFAULT (i.e., apply_current_anim already ran but didn't see our entry). It uses the same selection rule as apply_current_anim: specific pose wins over `M#T!`, just-pushed RAM-tier value is checked first.

Mid-session adjustments (`X+/Y+/Z+` from the `[Adjust]` dialog) are not overridden because they shift CURRENT away from DEFAULT, breaking the equality check.

Since RAM-tier writes only happen when LSD is at the floor (rare in practice), this race-fix code path is rarely traversed but kept as a defensive measure for the edge case.

## RAM-tier visibility: `QPP_CFG:RAM_TIER_COUNT`

`[QS]offset` writes the current `CUSTOMS` entry count to the unprotected LSD key `QPP_CFG:RAM_TIER_COUNT` whenever the count changes (save, drop, wipe, eviction). `[QS]hudadmin` reads this key in `getStorageReport()` (hudproxy delegates the storage dialog to it via 90267) so the CLEAR-confirm dialog can show how many offsets sit in RAM tier (would be lost on script reset). Empty or `"0"` means none.

## See also

- [LSD Storage](lsd-storage.html): overall persistence layout.
- [HUD Integration](hud-integration.html): hudproxy is also a writer/reader of personal offsets.
- [QSALIVE Discovery](qsalive-discovery.html): the `customs90260` and `offsetlsd_v1` capability bits.
