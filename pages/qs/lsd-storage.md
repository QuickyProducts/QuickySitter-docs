---
title: LSD Storage
sidebar: home_sidebar
permalink: lsd-storage.html
keywords: linkset data, lsd, storage, persistence, mono
toc: true
---

QuickySitter moves most of stock AVsitter's per-script state onto [Linkset Data (LSD)](http://wiki.secondlife.com/wiki/Category:LSL_Linkset_Data). The two motivations are heap pressure (stock keeps everything in `[AV]sitB` memory, which pushes past Mono's 64 KB cap on large configs) and durability across script resets and re-rezzes.

This page is the **state layout reference**: where every piece of runtime state lives, and whether it survives a reset. The companion link-message protocol is on [LinkMessage Numbers](linkmessage-numbers.html).

## Quick reference

| What | Where | Persistent? |
|------|-------|-------------|
| **Pose defaults** (`<pos><rot>` from AVpos) | LSD `qs:p:<ch>:<i>`, written by `[QS]boot` at seed and by `[QS]adjuster` on `[HELPER] [SAVE]` | ✅ survives rerez |
| **Personal user offsets** (per-(user, slot, pose), incl. `M#T!` per-slot all-poses fallback) | `[QS]offset`: LSD `QSO:<short>:<slot>:<pose>` when room allows, else RAM `CUSTOMS` list | ✅ LSD persistent, ❌ RAM volatile fallback |
| **Pose runtime state** (which pose is playing, menu navigation, speed) | `[QS]sitB` per-sitter globals | ❌ volatile per session |
| **Playback state** (`CURRENT_POSITION` / `CURRENT_ROTATION`, anim filename, `MY_SITTER`) | `[QS]sitA` per-sitter globals | ❌ volatile |
| **Channel settings** (MTYPE, ETYPE, SWAP, BRAND, CUSTOM_TEXT, ADJUST_MENU, …) | LSD `qs:cfg:<ch>` (boot writes) + in-memory cache in sitA/sitB | ✅ LSD persistent; memory is cache |
| **Sitter info** (names, gender) | LSD `qs:sitter:<ch>` | ✅ |
| **Boot marker** (channel already seeded?) | LSD `qs:meta:<ch>` (per-channel) + `qs:boot:asset` (notecard asset-key) | ✅ |
| **Plugin-presence flags** (which optional plugins are loaded) | LSD `qs:alive:<name>` (`prop`/`faces`/`adjuster`/`select`/`rlv`) + inverted `qs:offset:alive` | ✅ LSD, but re-stamped each boot / `QS_ALIVE_CENSUS` (90079) |
| **Prop database** (parsed PROP entries, lazy-loaded) | LSD `qs:prop:*` (`meta`/`<i>`/`trig:`/`sit:`/`grp:`), `[QS]prop` only | ✅ until notecard-key change, then wiped + re-parsed |
| **Dump output state** (cache, webkey, webcount) | `[QS]boot` globals | ❌ volatile per dump |

## Linkset Data layout

Most keys are namespaced `qs:*` (the `QSO:*` personal-offset and `QPP_CFG:*` HUD namespaces sit deliberately outside it, see below). `<ch>` is the sitter slot (0-based, matches `SCRIPT_CHANNEL` in sitA/sitB).

| Key | Format | Writer | Readers |
|-----|--------|--------|---------|
| `qs:cfg:<ch>` | `\n`-separated positional values: MTYPE, ETYPE, SET, SWAP, SELECT, AMENU, OLD_HELPER_METHOD, WARN, HASKEYFRAME, REFERENCE, DFLT, BRAND, onSit, CUSTOM_TEXT (escaped), ADJUST_MENU (SEP-joined), RLVDesignations, GENDERS (CSV) | boot's `qs_cfg_pack()` | sitA, sitB, select, boot's `qs_dump_start` |
| `qs:cfg:slots:<ch>` | integer string: the channel's pose-entry count (since 0.9952) | boot; sitB rewrites on sidecar rebuild | sitB (`SLOTS`), boot (skip-seed check) |
| `qs:cfg:verbose` | `"0"`–`"3"` verbose ladder (singleton, not per-channel) | boot (from AVpos `VERBOSE n`) | every fork plugin in `state_entry` |
| `qs:sitter:<ch>` | `SEP`-joined sitter info row | boot | boot's `qs_dump_start`, sitB, select |
| `qs:p:<ch>:<i>` | `name\|type\|anim\|pos\|rot` (type is single char: `P`/`S`/`M`/`T`/`B`) | boot's `qs_p_write()`, adjuster's `qs_save_pose_offset` / `qs_insert_pose` | sitB's `qs_pose_data()`, adjuster's `qs_find_index` / `qs_p_count`, boot's `qs_dump_tick` |
| `qs:nm:<ch>:<mi>`, `qs:nt:<ch>:<ti>` | page-oriented menu sidecar: section child count, TOMENU target | boot | sitB (page rebuild) |
| `qs:meta:<ch>` | `"qs1"` (presence = "channel seeded") | boot | sitA, sitB (`state_entry` poll) |
| `qs:boot:asset` | notecard asset-key as string, written last in `finalize_boot` after all `qs:meta:<ch>` | boot | boot's `state_entry` skip-check |
| `qs:alive:<name>` (+ inverted `qs:offset:alive`) | `"1"` when plugin present | each optional plugin | sitB, adjuster, boot |
| `qs:sec:adjust` | Adjust ACL level `OWNER`/`GROUP`/`ALL` (since 1.25) | root-security | sitB, adjuster |
| `qs:select:btn:<i>` | per-slot seat-picker label | select | hudproxy |
| `qs:prop:*` (`meta`/`<i>`/`trig:`/`sit:`/`grp:`) | lazy-loaded prop database (the `<i>` row is 11 tab-fields since 1.25) | prop | prop |
| `QSO:<short>:<slot>:<pose>` | `<pos>\|<rot>` (Euler degrees, both `vector`-string), unprotected | `[QS]offset` `save_offset` (when `lsdHasRoom()`) | `[QS]offset` `push_customs_for` / `drop_pose_for_slot`, sitA's `lookup_personal_offset`, hudproxy's `lookupEffectiveOffset` |

`SEP` is U+FFFD, initialized at runtime via `llUnescapeURL("%EF%BF%BD")` because the SL script editor mangles a literal U+FFFD on upload.

The `QSO:*` namespace is intentionally outside the `qs:*` family, because it lives outside the seed-and-forget layout, managed lazily across script lifetimes by `[QS]offset`. It also shares the prim with QuickyHUD's protected `QPP_CFG:*` keys without colliding.

## Why pose defaults moved to LSD

Stock AVsitter holds *everything* in `[AV]sitB`'s script memory: `DATA_LIST` (pose-anim mappings) and `POS_ROT_LIST` (positions/rotations). On `[HELPER] [SAVE]`, stock updates only those in-memory lists, and the AVpos notecard is **not** auto-written. The `[DUMP]` button exists exactly for that reason: the creator copy-pastes the dump output back into the notecard manually, or loses unsaved changes on the next script reset.

QuickySitter writes pose defaults to LSD on `[HELPER] [SAVE]`. They survive rerezes and re-imports as long as `qs:boot:asset` matches the notecard's current asset-key (boot then skips re-seeding from the notecard). The legacy `[DUMP]` is still there for human backup, but you no longer lose work by forgetting to use it.

Side benefit: at scale (1000+ poses), keeping `DATA_LIST` and `POS_ROT_LIST` in sitB memory would push it past Mono's 64 KB cap. With on-demand LSD reads via `qs_pose_data(idx)`, sitB stays slim regardless of config size.

## Why personal offsets stayed RAM-first (with optional LSD)

Stock keeps `CUSTOMS` in sitA's memory (per sitter slot). QuickySitter moved them out into a dedicated `[QS]offset` script with a two-tier store: LSD when there's room past the `QPP_CFG:RESERVE` budget, RAM otherwise. The LSD tier is persistent across script reset and re-rez; the RAM tier is volatile.

The split exists because LSD has a hard size cap (128 KiB; SL imposes no per-write throttle, only the cap). We don't want personal offsets to crowd out pose defaults when storage gets tight. The RAM tier is the safety valve.

See [Personal Pose Offsets](personal-pose-offsets.html) for the full read/write paths and the SSoT design.

## Reset behavior

| Trigger | Effect |
|---------|--------|
| Notecard changed (asset key differs, `CHANGED_INVENTORY` in boot) | boot broadcasts `QS_BOOT_WIPE` (90024) so sitA/sitB drop back to their pre-boot guard, wipes the notecard-derived keys (`^qs:(meta\|cfg\|sitter\|p\|nm\|nt\|boot):`), re-parses, **rewrites** them, then fires `QS_BOOT_RELOAD` (90023); each sitA/sitB reloads from fresh LSD **in place** (flag flip, no script reset). `qs:alive:*` and the other survivors are not wiped. |
| Sitter-script count changed (notecard unchanged) | boot runs a presence census (wipe + re-stamp `qs:alive:*`) rather than a re-seed. Each sitA self-resets via its own sibling probe and reloads the existing LSD. |
| Object rerez | LSD survives. Boot starts; if `qs:boot:asset` matches the notecard's current asset-key, skips re-seeding entirely (live edits via `[HELPER] [SAVE]` are preserved). Manual script reset / region restart hits the same path. |
| Owner changed | `[QS]offset` wipes both tiers: RAM `CUSTOMS` resets and `QSO:*` LSD keys are deleted (visitors' UUIDs from the previous owner's setting shouldn't follow the prim to a new owner). `qs:*` LSD survives unless the new owner re-imports the notecard. |
| Manual reset of one sitA or sitB | that script reads from LSD on `state_entry` and rejoins the running system; `[QS]offset` doesn't re-push customs until next sit triggers 90261 |

## Per-script state breakdown

### `[QS]boot.lsl` (one instance)

Notecard parser globals during seed; dump-streaming state (`qs_dump_ch`, `qs_dump_pi`, `cache`, `webkey`, `webcount`); boot orchestration (`total_channels`, `notecard_query`, `notecard_lines`). Idle once seed completes.

### `[QS]sitA.lsl` (one per sitter slot)

Per-sitter playback (`MY_SITTER`, `CONTROLLER`, `RAM_OVERFLOW` (the RAM-tier offset mirror, formerly `MY_CUSTOMS`), `DEFAULT_POSITION/ROTATION`, `CURRENT_POSITION/ROTATION`, `CURRENT_POSE_NAME`, `CURRENT_ANIMATION_FILENAME`, …); first-sit defaults; gender variants; settings cache read from `qs:cfg`; sitter list; dialog state.

### `[QS]sitB.lsl` (one per sitter slot)

`page_map` / `nav_stack` / `SLOTS` (the 0.9954 page-oriented menu state that retired the flat `MENU_LIST`), `ANIM_INDEX`, menu state, settings cache. Labels are rendered per page from `qs:p:<ch>:<i>` on demand. **Does not hold** `DATA_LIST`/`POS_ROT_LIST`, which are read on demand from LSD via `qs_pose_data(idx)`. This is the key memory-saving change vs stock.

### `[QS]offset.lsl` (one instance, optional)

Two-tier store: LSD `QSO:<short>:<slot>:<pose>` and RAM `CUSTOMS` list with stride 5. See [Personal Pose Offsets](personal-pose-offsets.html) for the full design.

### `[QS]adjuster.lsl` (one instance)

Creator-tool runtime state only (helper-bar mode, listen channel, sitter tracking, menu navigation). Pose-data writes go through `qs_save_pose_offset` / `qs_insert_pose` into the shared `qs:p:<ch>:<i>` namespace. It also owns two small flags of its own: `qs:alive:adjuster` (presence) and `qs:adjuster:silent` (one-shot banner suppression on inventory-churn resets).

## See also

- [Personal Pose Offsets](personal-pose-offsets.html): the `[QS]offset` two-tier design.
- [Boot Sequence](boot-sequence.html): how `[QS]boot` populates LSD on first run.
- [LSD Keys](lsd-keys.html): quick-reference table of every LSD key the fork uses.
