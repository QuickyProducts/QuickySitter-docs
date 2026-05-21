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
| `qs:p:*` | `[QS]boot` (seed) and `[QS]adjuster` (live edits) write; sitB, boot read | Pose defaults — one key per pose entry. |
| `qs:meta:*` | `[QS]boot` writes; sitA, sitB poll | Per-channel "seeded" marker. |
| `qs:boot:*` | `[QS]boot` writes and reads | Boot orchestration markers (currently only `qs:boot:asset`). |
| `QSO:*` | `[QS]offset` writes and reads; sitA reads | Personal pose offsets (per user, per slot, per pose). |
| `QPP_CFG:*` | QuickyHUD scripts write; sitA, sitB, adjuster, offset read | HUD configuration. Mostly protected on the HUD side. `ADJUSTMODE` and `RAM_TIER_COUNT` are the unprotected exceptions. |

## Detailed key reference

### `qs:cfg:<ch>`

**Writer:** `[QS]boot` via `qs_cfg_pack()`.
**Readers:** `[QS]sitA`, `[QS]sitB`, `[QS]boot.qs_dump_start`.
**Format:** `\n`-separated positional values, in order: MTYPE, ETYPE, SET, SWAP, SELECT, AMENU, OLD_HELPER_METHOD, WARN, HASKEYFRAME, REFERENCE, DFLT, BRAND, onSit, CUSTOM_TEXT (escaped), ADJUST_MENU (SEP-joined), RLVDesignations, GENDERS (CSV).

Persistent across rerez. `<ch>` is the 0-based sitter slot.

### `qs:sitter:<ch>`

**Writer:** `[QS]boot`.
**Readers:** `[QS]boot.qs_dump_start`, `[QS]sitB`.
**Format:** `SEP`-joined sitter info row.

`SEP` is U+FFFD, initialized at runtime via `llUnescapeURL("%EF%BF%BD")` because the SL script editor mangles a literal U+FFFD on upload.

### `qs:p:<ch>:<i>`

**Writers:** `[QS]boot.qs_p_write()` during seed; `[QS]adjuster.qs_save_pose_offset` / `qs_add_pose` for live `[HELPER] [SAVE]` edits.
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

**Writer:** `[QS]boot` — written **last** in `finalize_boot`, after all `qs:meta:<ch>` keys, so the marker only exists if everything before it succeeded.
**Readers:** `[QS]boot` (skip-check in `state_entry`).
**Format:** notecard asset-key as string.

Compared against `llGetInventoryKey("AVpos")` at `state_entry`. Match → skip re-parse, sitA/sitB read existing LSD. Mismatch → fresh seed.

See [Boot Sequence](boot-sequence.html) for the full skip-seed / fresh-seed decision.

### `QSO:<short>:<slot>:<pose>`

**Writer:** `[QS]offset.save_offset` (when `lsdHasRoom()` returns TRUE), ≥ 0.09.
**Readers:** `[QS]offset.push_customs_for`, `drop_pose_for_slot`; `[QS]sitA.apply_current_anim`; hudproxy's `lookupEffectiveOffset`.
**Format:** `<pos>|<rot>` (Euler degrees, both `vector`-string).

Unprotected. The slot in the key lets each sitter slot keep its own offset for the same pose name (SYNC couple poses on multiple slots had a flat (user, pose) key before 0.09 and would overwrite each other on save).

`<short>` is the first 8 characters of the user's UUID; `<pose>` is the pose name. Magic name `M#T!` is the all-poses fallback set via `[SAVE ALL]`.

See [Personal Pose Offsets](personal-pose-offsets.html).

### `QPP_CFG:ADJUSTMODE`

**Writer:** `[QS]hudproxy.state_entry` (initial), `[QS]hudproxy.link_message` (90266 from adjuster), `[QS]adjuster.timer` (delete when hudproxy probe fails).
**Readers:** `[QS]sitA`, `[QS]sitB`, `[QS]adjuster` (capability detection via `llLinksetDataFindKeys`).
**Format:** `"On"` / `"Off"` (or absent).

Unprotected by design — adjuster needs delete rights for the uninstall path. See [HUD Integration](hud-integration.html).

### `QPP_CFG:RAM_TIER_COUNT`

**Writer:** `[QS]offset` whenever the RAM-tier `CUSTOMS` count changes (save, drop, wipe, eviction).
**Reader:** hudproxy's `getStorageReport()`.
**Format:** integer string. Empty or `"0"` means none.

Lets the CLEAR-confirm dialog show how many offsets sit in RAM tier (and would be lost on script reset).

### `QPP_CFG:RESERVE`

**Writer:** hudprop (proprietary, in the QuickyHUD repo).
**Readers:** `[QS]offset.lsdHasRoom`.
**Format:** integer string (bytes).

The reserve hudprop expects to remain available for its own protected keys. `[QS]offset` won't write LSD-tier offsets that would push free bytes below this floor + `LSD_MIN_FREE_POSES × LSD_BYTES_PER_ENTRY`.

## See also

- [LSD Storage](lsd-storage.html) — the rationale and tier breakdown.
- [Boot Sequence](boot-sequence.html) — how boot populates the `qs:*` namespace.
- [Personal Pose Offsets](personal-pose-offsets.html) — the `QSO:*` design.
- [HUD Integration](hud-integration.html) — `QPP_CFG:*` boundaries.
