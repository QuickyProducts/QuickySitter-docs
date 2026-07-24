---
title: LSD Keys
sidebar: home_sidebar
permalink: lsd-keys.html
keywords: lsd, linkset data, keys, namespace, qs, QSO, QPP_CFG
toc: true
---

Quick-reference table for every Linkset Data key QuickySitter writes or reads. For the rationale behind the layout see [LSD Storage](lsd-storage.html); for the link-message protocol that moves data between keys see [LinkMessage Numbers](linkmessage-numbers.html).

## Namespace map

| Prefix | Owner | Purpose |
|--------|-------|---------|
| `qs:cfg:*` | `[QS]boot` writes; sitA, sitB, boot read | Per-channel furniture settings (MTYPE, ETYPE, SWAP, BRAND, ADJUST_MENU, …). |
| `qs:sitter:*` | `[QS]boot` writes; sitB reads | Per-channel sitter info row (names, gender). |
| `qs:p:*` | `[QS]boot` (seed) and `[QS]adjuster` (live edits) write; sitB, boot read | Pose defaults: one key per pose entry. |
| `qs:meta:*` | `[QS]boot` writes; sitA, sitB poll | Per-channel "seeded" marker. |
| `qs:boot:*` | `[QS]boot` writes and reads | Boot orchestration markers (currently only `qs:boot:asset`). |
| `qs:alive:*` | each optional plugin writes its own flag; sitA, sitB, adjuster read | Plugin-presence flags (`qs:alive:prop`, `…:faces`, `…:adjuster`, `…:select`, `…:rlv`). `[QS]offset` uses the **inverted** name `qs:offset:alive`. Read on demand at menu-build, re-stamped on `QS_ALIVE_CENSUS` (90079). |
| `qs:prop:*` | `[QS]prop` writes and reads | Lazy-loaded prop database (replaces parsing PROP entries from the notecard on every play). |
| `qs:sec:adjust` | `[QS]root-security` writes; sitB, adjuster read | Adjust ACL level (`OWNER`/`GROUP`/`ALL`): the 1.25 gate for who may open `[ADJUST]` / `[QUICKYHUD]`. |
| `qs:nm:*`, `qs:nt:*` | `[QS]boot` writes; sitB reads | Page-oriented menu sidecar: `qs:nm:<ch>:<mi>` = child count of a menu section, `qs:nt:<ch>:<ti>` = the MENU a TOMENU navigates to. Lets sitB rebuild pages in O(1) instead of walking the pose list. |
| `qs:select:btn:*` | `[QS]select` writes; hudproxy reads | Per-slot seat-picker button labels (notecard-derived) so the HUD SWAP dialog shows real seat names instead of generic "Sitter N". |
| `qs:hud:*` | `[QS]hudadmin` writes; sitB, hudproxy, animesh read | HUD liveness / license mirror: `qs:hud:admin_alive`, `qs:hud:unlicensed`. |
| `qs:adjuster:silent` | `[QS]adjuster` writes and clears | One-shot flag: suppresses the "Ready" banner on an inventory-churn auto-reset. |
| `QSO:*` | `[QS]offset` writes and reads; sitA, hudproxy read | Personal pose offsets (per user, per slot, per pose). |
| `QPP_CFG:*` | QuickyHUD scripts write; sitA, sitB, adjuster, offset, boot read | HUD configuration. Mostly protected on the HUD side. `ADJUSTMODE`, `RAM_TIER_COUNT`, `AUTOSYNC`, and (since 1.25) `RESERVE` are written unprotected so the sitter-side scripts can read them without the HUD password. |

## Detailed key reference

### `qs:cfg:<ch>`

**Writer:** `[QS]boot` via `qs_cfg_pack()`.
**Readers:** `[QS]sitA`, `[QS]sitB`, `[QS]select` (reads `qs:cfg:0`), `[QS]boot.qs_dump_start`.
**Format:** `\n`-separated positional values, in order: MTYPE, ETYPE, SET, SWAP, SELECT, AMENU, OLD_HELPER_METHOD, WARN, HASKEYFRAME, REFERENCE, DFLT, BRAND, onSit, CUSTOM_TEXT (escaped), ADJUST_MENU (SEP-joined), RLVDesignations, GENDERS (CSV).

Persistent across rerez. `<ch>` is the 0-based sitter slot.

### `qs:cfg:verbose`

**Writer:** `[QS]boot.state_entry` (from the AVpos `VERBOSE n` directive).
**Readers:** `[QS]sitA`, `[QS]sitB`, `[QS]adjuster`, `[QS]faces`, `[QS]offset`, `[QS]prop`, `[QS]select`, `[QS]sequence`, `[QS]root-RLV`: each reads it in `state_entry`.
**Format:** `"0"`–`"3"`. Project-wide verbose ladder (0 = errors only, 1 = boot banner, 2 = runtime status, 3 = debug). Singleton, not per-channel.

### `qs:cfg:slots:<ch>`

**Writer:** `[QS]boot` (`qs:cfg:slots:<ch>` written at seed); `[QS]sitB` rewrites it during sidecar rebuild.
**Readers:** `[QS]sitB` (`SLOTS`, replaces `llGetListLength(MENU_LIST)`); `[QS]boot` reads `qs:cfg:slots:0` in the skip-seed check (an asset-key match with a missing sidecar forces one re-parse).
**Format:** integer string: the channel's pose-entry count. Added in 0.9952.

### `qs:sitter:<ch>`

**Writer:** `[QS]boot`.
**Readers:** `[QS]boot.qs_dump_start`, `[QS]sitB`, `[QS]select` (per-slot seat labels).
**Format:** `SEP`-joined sitter info row.

`SEP` is U+FFFD, initialized at runtime via `llUnescapeURL("%EF%BF%BD")` because the SL script editor mangles a literal U+FFFD on upload.

### `qs:p:<ch>:<i>`

**Writers:** `[QS]boot.qs_p_write()` during seed; `[QS]adjuster.qs_save_pose_offset` / `qs_insert_pose` for live `[HELPER] [SAVE]` and `[NEW]` edits.
**Readers:** `[QS]sitB.qs_pose_data()`, `[QS]adjuster.qs_find_index` / `qs_p_count`, `[QS]boot.qs_dump_tick`.
**Format:** `name|type|anim|pos|rot`. `type` is a single char: `P` (pose, solo), `S` (sync), `M` (menu), `T` (tomenu), `B` (button).

One key per pose entry. `<i>` is the 0-based entry index within the channel.

This is the key that replaces stock `DATA_LIST` + `POS_ROT_LIST` in sitB memory.

### `qs:meta:<ch>`

**Writer:** `[QS]boot`.
**Readers:** `[QS]sitA`, `[QS]sitB` (`state_entry` poll).
**Format:** literal `"qs1"`.

Presence of this key = "channel has been seeded." sitA and sitB poll for it before reading `qs:cfg:<ch>` / `qs:p:<ch>:*`.

### `qs:boot:asset`

**Writer:** `[QS]boot`, written **last** in `finalize_boot`, after all `qs:meta:<ch>` keys, so the marker only exists if everything before it succeeded.
**Readers:** `[QS]boot` (skip-check in `state_entry`).
**Format:** notecard asset-key as string.

Compared against `llGetInventoryKey("AVpos")` at `state_entry`. Match → skip re-parse, sitA/sitB read existing LSD. Mismatch → fresh seed.

See [Boot Sequence](boot-sequence.html) for the full skip-seed / fresh-seed decision.

### `qs:alive:<name>` (and `qs:offset:alive`)

**Writers:** each optional plugin writes its own flag in `state_entry`: `[QS]prop` → `qs:alive:prop`, `[QS]faces` → `qs:alive:faces`, `[QS]adjuster` → `qs:alive:adjuster`, `[QS]select` → `qs:alive:select`, `[QS]root-RLV` → `qs:alive:rlv`. `[QS]offset` writes the **inverted** name `qs:offset:alive`.
**Readers:** `[QS]sitB` (`select_present()`, `rlv_present()`, `[FACES]`/`[HELPER]` gating), `[QS]adjuster` (`[PROP]`/`[FACE]`), `[QS]boot` (self-check), `[QS]sitA` + hudproxy read `qs:offset:alive`.
**Format:** `"1"` when present (absent = plugin not loaded).

Read on demand at menu-build, never cached. `[QS]boot` wipes every `qs:alive:*` + `qs:offset:alive` only on the plugin add/remove path (a script-count change), then broadcasts `QS_ALIVE_CENSUS` (90079) so survivors re-stamp. `finalize_boot` only re-broadcasts the census (no wipe), covering the rare full-reset case where the flags were cleared without the plugins resetting. See [QSALIVE Discovery](qsalive-discovery.html).

### `qs:prop:*`

**Writer / Reader:** `[QS]prop` only.
**Layout:** lazy-loaded prop database, replacing repeated notecard parsing:

| Key | Format |
|-----|--------|
| `qs:prop:meta` | `<notecard_key>\t<count>\t<warn>\t<groups_nl>` : lazy-load index header; a matching `notecard_key` means the parsed record is current. |
| `qs:prop:<i>` | `<trig>\t<type>\t<obj>\t<grp>\t<pos>\t<rot>\t<pt>\t<prs>\t<scl>\t<wpos>\t<wrot>` (11 fields since 1.25: `scl` = uniform scale factor, `wpos`/`wrot` = worn-fit local pos + Euler-deg rotation, all `""` when unset). One row per parsed prop entry. Legacy rows written by older versions have 8 or 9 fields and stay readable (missing trailing fields read `""`). |
| `qs:prop:trig:<trig>` | CSV of `qs:prop:<i>` indices matching this trigger string. |
| `qs:prop:sit:<sit>` | CSV of `qs:prop:<i>` indices belonging to this sitter slot. |
| `qs:prop:grp:<grp>` | CSV of `qs:prop:<i>` indices belonging to this group. |

The whole namespace is wiped (`^qs:prop:.*`) and re-parsed only when the AVpos notecard's asset key changes. A `CHANGED_INVENTORY` event with an unchanged key just re-probes presence; it does not wipe the database.

### `qs:hud:unlicensed`

**Writer:** `[QS]hudadmin` (QuickyHUD repo), which sets `"1"` when its license check fails.
**Readers:** `[QS]sitB` (`[HELPER]`/`[QUICKYHUD]` gate) and hudproxy (`[QUICKYHUD]` attach gate). `[QS]adjuster` does **not** read it. Gates only the `[QUICKYHUD]` entry, not `[HELPER]`.
**Format:** `"1"` when unlicensed; absent otherwise. Singleton flag (since 0.9935).

### `QSO:<short>:<slot>:<pose>`

**Writer:** `[QS]offset.save_offset` (when `lsdHasRoom()` returns TRUE).
**Readers:** `[QS]offset.push_customs_for`, `drop_pose_for_slot`; `[QS]sitA.apply_current_anim`; hudproxy's `lookupEffectiveOffset`.
**Format:** `<pos>|<rot>` (Euler degrees, both `vector`-string).

Unprotected. The slot in the key lets each sitter slot keep its own offset for the same pose name (an earlier flat (user, pose) key let SYNC couple poses on multiple slots overwrite each other on save).

`<short>` is the first 8 characters of the user's UUID; `<pose>` is the pose name. Magic name `M#T!` is the all-poses fallback set via the sitter dialog's `[OFFSET ALL]` button (the HUD labels the same action `[SAVE ALL]`).

See [Personal Pose Offsets](personal-pose-offsets.html).

### `QPP_CFG:ADJUSTMODE`

**Writers:** `[QS]hudproxy.state_entry` (initial), `[QS]hudproxy.link_message` (90266 from adjuster), `[QS]hudadmin` (writes `"Off"` on an LSD reset, plus the one-time protected-to-unprotected migration), `[QS]adjuster.timer` (delete when hudproxy probe fails).
**Readers:** `[QS]sitB`, `[QS]adjuster` (capability detection via `llLinksetDataFindKeys`). sitA no longer reads it.
**Format:** `"On"` / `"Off"` (or absent).

Unprotected by design, because adjuster needs delete rights for the uninstall path. See [HUD Integration](hud-integration.html).

### `QPP_CFG:RAM_TIER_COUNT`

**Writer:** `[QS]offset` whenever the RAM-tier `CUSTOMS` count changes (save, drop, wipe, eviction).
**Reader:** `[QS]hudadmin`'s `getStorageReport()`.
**Format:** integer string. Empty or `"0"` means none.

Lets the CLEAR-confirm dialog show how many offsets sit in RAM tier (and would be lost on script reset).

### `QPP_CFG:RESERVE`

**Writers:** `[QS]hudadmin` (unprotected since 1.25) and the legacy installers `installWithLicense` / `dropInFurniture` (protected). There is no separate "hudprop" script.
**Readers:** `[QS]offset.lsdHasRoom` (plain read).
**Format:** integer string (bytes).

The reserve the HUD wants kept free for its own protected keys. `[QS]offset` won't write LSD-tier offsets that would push free bytes below this floor + `LSD_MIN_FREE_POSES × LSD_BYTES_PER_ENTRY`. (Before the 1.25 unprotected-write fix, hudadmin wrote this key protected while `[QS]offset` read it plain, so the reserve read back as `""` and was never honored.)

## See also

- [LSD Storage](lsd-storage.html): the rationale and tier breakdown.
- [Boot Sequence](boot-sequence.html): how boot populates the `qs:*` namespace.
- [Personal Pose Offsets](personal-pose-offsets.html): the `QSO:*` design.
- [HUD Integration](hud-integration.html): `QPP_CFG:*` boundaries.
