---
title: Migration from AVsitter
sidebar: home_sidebar
permalink: migration.html
keywords: migration, avsitter, convert, port, swap
toc: true
---

If you already have a working AVsitter 2 furniture and want to move it to QuickySitter, this page covers the procedure and what to expect.

## TL;DR

1. Keep the existing `AVpos` notecard — no edits needed. (It is mandatory: boot ERRORs without it.)
2. Delete `[AV]sitA` + `[AV]sitB`. Add `[QS]boot`, `[QS]sitA`, `[QS]sitB`. These three plus the notecard are the only required ingredients.
3. Optional but recommended: also swap `[AV]select` → `[QS]select`, `[AV]adjuster` → `[QS]adjuster`, `[AV]prop` → `[QS]prop`, `[AV]faces` → `[QS]faces`, `[AV]sequence` → `[QS]sequence`. Add `[QS]offset` for personal-offset persistence.
4. Reset the prim. Boot reads the existing AVpos and seeds LSD.

Stock `[AV]camera`, `[AV]favs`, `[AV]helperscript`, and the LockGuard / LockMeister / Xcite! plugins all work unchanged inside a QS linkset and don't need to be replaced. QS forks its own root family (`[QS]root`, `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV`), but the stock `[AV]root-*` scripts also keep working if you leave them in.

## What changes for the end user

Nothing user-visible if you swap only the base scripts. The pose menu, sit-target, animation sequences, and props all behave identically — QS is structurally different inside but presents the same surface to sitters.

Differences become visible if you add `[QS]offset` (personal-offset persistence across reset) or `[QS]adjuster` plus the QuickyHUD addon (`[QS]hudproxy`/`[QS]hudadmin` from the separate QuickyHUD project) for HUD-driven adjustment with live `[SAVE]` writing into LSD.

## What changes for the creator

### `[HELPER] [SAVE]` actually persists

In stock AVsitter, `[SAVE]` updates the in-memory pose default but **does not** write back to the `AVpos` notecard. The `[DUMP]` button is the manual workaround — you copy-paste the dump output back into the notecard.

In QuickySitter, `[SAVE]` writes the new pose offset to LSD (`qs:p:<ch>:<i>`), which survives object rerez, script reset, and region restart. The notecard isn't touched, but the next boot reads LSD ahead of re-parsing if the asset-key matches, so your live edits are preserved.

You can still `[DUMP]` to back up the LSD state to the AVsitter settings service or the chat console — see [Adjustment Workflow](adjustment-workflow.html). Boot will overwrite LSD with notecard content whenever the notecard's asset-key changes (typically when you re-save the notecard).

### Larger configs become stable

Stock `[AV]sitB` holds the full `DATA_LIST` (pose → animation names) and `POS_ROT_LIST` (positions/rotations) in script memory. At 1000+ pose entries, sitB pushes past Mono's 64 KB cap and either fails to compile or runs out of free bytes at runtime.

QuickySitter's `[QS]sitB` reads `qs:p:<ch>:<i>` from LSD on demand via `qs_pose_data(idx)`. Per-call cost is ~50 µs (Mono hashmap lookup); total script memory is independent of pose count. You won't notice the difference until you cross stock's memory limit.

### Stock plugins keep working

Drop a stock `[AV]prop` or `[AV]faces` into a QS prim and it works. Stock plugins use legacy script-name inventory probes (`llGetInventoryType("[AV]sitA")`) to find the main script; QS satisfies those probes, so the stock plugin latches on as usual. Some QS-specific gating (the `[PROP]` menu item being available, for example) requires the QS variant of the plugin, because that is what publishes the relevant `qs:alive:<name>` presence flag (`qs:alive:prop`, `qs:alive:faces`) that the menu reads. The old HELLO broadcasts (90088–90092) that used to carry this presence were retired in 0.9951.

## Step-by-step migration

1. **Backup.** Take the existing prim to your inventory, then take a second copy and rename it (e.g., "Foo (QS port)"). Work on the copy.
2. **Swap the base scripts.** In the prim's contents:
   - Delete `[AV]sitA`, `[AV]sitA 2`, `[AV]sitA 3`, … (one per sitter slot).
   - Delete `[AV]sitB`, `[AV]sitB 2`, `[AV]sitB 3`, … (matching count).
   - Add `[QS]boot` (one instance, no slot suffix).
   - Add `[QS]sitA`, `[QS]sitA 2`, `[QS]sitA 3`, … (same count as before).
   - Add `[QS]sitB`, `[QS]sitB 2`, `[QS]sitB 3`, … (same count).
3. **(Optional) Swap plugins.** Replace `[AV]select`, `[AV]adjuster`, `[AV]prop`, `[AV]faces`, `[AV]sequence` with their `[QS]` counterparts. (`[QS]select` is optional — sitB has a built-in picker.) Add `[QS]offset` if you want personal-offset persistence; if you want the forked root family, swap in `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV`.
4. **Leave stock plugins alone.** Keep `[AV]camera`, `[AV]LockGuard`, `[AV]LockMeister`, `[AV]Xcite!`, `[AV]favs`, `[AV]texture`, and any stock `[AV]root-*` scripts if present. They work as-is.
5. **Keep the AVpos notecard.** No edits needed — same syntax.
6. **Reset.** Right-click the prim, edit, contents, hit "Reset Scripts in Selection" (or just `llResetScript` on `[QS]boot`). Boot detects no prior `qs:boot:asset`, parses AVpos, seeds LSD, broadcasts reload. Total time ~1 – 5 s depending on AVpos size.

You should see `llOwnerSay` chatter from boot reporting parsed channels. Sit on the prim — your menus, poses, and adjuster should work as before.

## What stays stock by design

- **`[AV]camera`** — stock camera's only name-bound code (`get_number_of_scripts` via `main_script="[AV]sitA"`) is dead, and all working paths are protocol-based. There's no `[QS]camera` planned.
- **`[AV]LockGuard` / `[AV]LockMeister` / `[AV]Xcite!`** — third-party lock/Xcite plugins. QS doesn't fork them.
- **`[AV]favs`** — favourites are user-state, stored in sitA via 90401/90402/90403. Stock favs works unchanged.
- **`[AV]helperscript`** — only relevant during the import workflow, not at runtime.

QS **does** fork the root family — `[QS]root`, `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV` (the last publishes `qs:alive:rlv`). The stock `[AV]root-*` scripts still work if you leave them, so swapping them is optional. There is no `[AV]root-RLV-extra` in the set.

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
