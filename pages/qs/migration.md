---
title: Migration from AVsitter
sidebar: home_sidebar
permalink: migration.html
keywords: migration, avsitter, convert, port, swap
toc: true
---

If you already have a working AVsitter 2 furniture and want to move it to QuickySitter, this page covers the procedure and what to expect.

## At a glance

1. Keep the existing `AVpos` notecard — no edits needed. (It is mandatory: boot ERRORs without it.)
2. Delete `[AV]sitA` + `[AV]sitB`. Add `[QS]boot`, `[QS]sitA`, `[QS]sitB`. These three plus the notecard are the only required ingredients.
3. Swap the feature plugins your piece uses — `[AV]adjuster` → `[QS]adjuster`, `[AV]prop` → `[QS]prop`, `[AV]faces` → `[QS]faces`, `[AV]sequence` → `[QS]sequence`. **Required for those features to work**, not just tidy-up: the stock versions probe `[AV]sitA` names that don't exist here, so multi-sitter playback fails and the `qs:alive:*`-gated menu entries (`[FACES]`, `[HELPER]`) never appear. For seat-picking use `[QS]select` or sitB's built-in picker — don't leave stock `[AV]select` in. Add `[QS]offset` for personal-offset persistence.
4. Reset the prim. Boot reads the existing AVpos and seeds LSD.

Stock `[AV]camera`, `[AV]favs`, `[AV]helperscript`, and the LockGuard / LockMeister / Xcite! plugins all work unchanged inside a QS linkset and don't need to be replaced. QS forks its own root family (`[QS]root`, `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV`); stock `[AV]root-control` / `[AV]root-security` keep working, but stock `[AV]root-RLV` detects extra sitters by name-probing `[AV]sitA 1`, so on multi-sitter pieces its RLV capture and seat-relocation misfire (basic restraints still work) — use `[QS]root-RLV` there.

## What changes for the end user

Swap the base scripts **and** the matching `[QS]` plugins and nothing is user-visible — the pose menu, sit-target, sequences, faces and props behave identically; QS is structurally different inside but presents the same surface to sitters. Swap only the base and leave stock plugins, and those plugin features degrade (see the swap step above).

Differences become visible if you add `[QS]offset` (personal-offset persistence across reset) or `[QS]adjuster` plus the QuickyHUD addon (`[QS]hudproxy`/`[QS]hudadmin` from the separate QuickyHUD project) for HUD-driven adjustment with live `[SAVE]` writing into LSD.

## What changes for the creator

### `[HELPER] [SAVE]` actually persists

In stock AVsitter, `[SAVE]` updates the in-memory pose default but **does not** write back to the `AVpos` notecard. The `[DUMP]` button is the manual workaround — you copy-paste the dump output back into the notecard.

In QuickySitter, `[SAVE]` writes the new pose offset to LSD (`qs:p:<ch>:<i>`), which survives object rerez, script reset, and region restart. The notecard isn't touched, but the next boot reads LSD ahead of re-parsing if the asset-key matches, so your live edits are preserved.

You can still `[DUMP]` to back up the LSD state to the AVsitter settings service or the chat console — see [Adjustment Workflow](adjustment-workflow.html). Boot will overwrite LSD with notecard content whenever the notecard's asset-key changes (typically when you re-save the notecard).

### Larger configs become stable

Stock `[AV]sitB` holds the full `DATA_LIST` (pose → animation names) and `POS_ROT_LIST` (positions/rotations) in script memory. At 1000+ pose entries, sitB pushes past Mono's 64 KB cap and either fails to compile or runs out of free bytes at runtime.

QuickySitter's `[QS]sitB` reads `qs:p:<ch>:<i>` from LSD on demand via `qs_pose_data(idx)`. Per-call cost is ~50 µs (Mono hashmap lookup); total script memory is independent of pose count. You won't notice the difference until you cross stock's memory limit.

### Which stock plugins keep working

Purely protocol-driven stock plugins (camera, the lock/adult plugins, favs, texture, helperscript) work unchanged in a QS prim. Stock plugins that **find the engine by script name** do not: `llGetInventoryType("[AV]sitA")` presence probes and `[AV]sitA N` sitter-count walks come up empty in a QS linkset (the scripts are named `[QS]sitA`), so `[AV]faces`, `[AV]select`, `[AV]adjuster`, `[AV]sequence`, `[AV]prop` and `[AV]root-RLV` degrade to single-sitter behavior at best. Their QS menu entries also never appear — the gates read `qs:alive:<name>` presence flags (`qs:alive:prop`, `qs:alive:faces`, …) that only the `[QS]` variants publish. Swap those five (step 3 below); see the [Compatibility Matrix](compatibility-matrix.html) for the per-plugin detail.

## Step-by-step migration

1. **Backup.** Take the existing prim to your inventory, then take a second copy and rename it (e.g., "Foo (QS port)"). Work on the copy.
2. **Swap the base scripts.** In the prim's contents:
   - Delete `[AV]sitA`, `[AV]sitA 2`, `[AV]sitA 3`, … (one per sitter slot).
   - Delete `[AV]sitB`, `[AV]sitB 2`, `[AV]sitB 3`, … (matching count).
   - Add `[QS]boot` (one instance, no slot suffix).
   - Add `[QS]sitA`, `[QS]sitA 2`, `[QS]sitA 3`, … (same count as before).
   - Add `[QS]sitB`, `[QS]sitB 2`, `[QS]sitB 3`, … (same count).
3. **Swap the name-probing plugins.** Replace `[AV]adjuster`, `[AV]prop`, `[AV]faces`, `[AV]sequence` with their `[QS]` counterparts — these detect the engine via `[AV]sitA` script names and degrade in a QS linkset, so their features (props, faces, sequences, the `[HELPER]`/`[SAVE]` flow) won't work without the swap. (`[QS]select` is optional — sitB has a built-in picker — but remove stock `[AV]select` rather than leaving it.) Add `[QS]offset` if you want personal-offset persistence. For RLV furniture swap in `[QS]root-RLV` (stock `[AV]root-RLV` name-probes `[AV]sitA 1`, so its capture/seat-relocation misfires on multi-sitter pieces) together with `[QS]root-control` / `[QS]root-security` — the control suite addresses its members by name, so don't mix `[QS]` and `[AV]`.
4. **Leave the protocol-driven stock plugins alone.** Keep `[AV]camera`, `[AV]LockGuard`, `[AV]LockMeister`, `[AV]Xcite!`, `[AV]favs`, `[AV]texture` — they work as-is. Stock `[AV]root-control` / `[AV]root-security` are fine too; only `[AV]root-RLV` needs the `[QS]` fork on multi-sitter pieces (see step 3).
5. **Keep the AVpos notecard.** No edits needed — same syntax.
6. **Reset.** Right-click the prim, edit, contents, hit "Reset Scripts in Selection" (or just `llResetScript` on `[QS]boot`). Boot detects no prior `qs:boot:asset`, parses AVpos, seeds LSD, broadcasts reload. A large AVpos notecard can take a while to seed — wait for boot's completion chatter before sitting.

You should see `llOwnerSay` chatter from boot reporting parsed channels. Sit on the prim — your menus, poses, and adjuster should work as before.

## What stays stock by design

- **`[AV]camera`** — stock camera's only name-bound code (`get_number_of_scripts` via `main_script="[AV]sitA"`) is dead, and all working paths are protocol-based. There's no `[QS]camera` planned.
- **`[AV]LockGuard` / `[AV]LockMeister` / `[AV]Xcite!`** — third-party lock/Xcite plugins. QS doesn't fork them.
- **`[AV]favs`** — favourites are user-state, stored in sitA via 90401/90402/90403. Stock favs works unchanged.
- **`[AV]helperscript`** — only relevant during the import workflow, not at runtime.

QS **does** fork the root family — `[QS]root`, `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV` (the last publishes `qs:alive:rlv`). Stock `[AV]root-control` / `[AV]root-security` still work if you leave them; stock `[AV]root-RLV`, though, name-probes `[AV]sitA 1` to spot extra sitters, so its capture/seat-relocation misfires on multi-sitter pieces — swap it for `[QS]root-RLV` there. There is no `[AV]root-RLV-extra` in the set.

## What does NOT migrate

- **Personal offsets stored in stock `CUSTOMS`.** In stock AVsitter these live in sitA's per-session memory and are wiped on script reset. Migration replaces sitA, so any per-user offsets accumulated in the previous prim are gone — users who had `[SAVE OFFSET]`-ed positions will need to re-save after the migration. (This is the same as any stock-AVsitter reset.) Going forward, install `[QS]offset` and the re-saved offsets persist in LSD (`QSO:*`) across reset and re-rez instead of being volatile. Note that `[QS]offset` **owns** the `CUSTOMS` store: without it there is no personal-offset persistence and no sitA fallback.
- **`[DUMP]` history.** The `[DUMP]` URL written to the AVsitter settings service is tied to the prim/sitter UUID, not the script. After migration the dump URL changes; old URLs go stale.

## Reverting

If something goes wrong, your inventory backup is intact. Re-rez the original AVsitter version and the existing settings (notecard + sitter UUIDs) are all preserved.

The `qs:*` LSD keys written by the QS version don't conflict with stock AVsitter — stock doesn't read them. If you go back to AV scripts inside the same prim, the leftover `qs:*` LSD entries are dead weight but harmless. Stock simply re-reads AVpos as if nothing was there.

## See also

- [Getting Started](getting-started.html) — full install procedure for fresh prims.
- [Compatibility Matrix](compatibility-matrix.html) — plugin-by-plugin status.
- [LSD Storage](lsd-storage.html) — what boot writes during seed.
- [Boot Sequence](boot-sequence.html) — fresh-seed vs skip-seed.
