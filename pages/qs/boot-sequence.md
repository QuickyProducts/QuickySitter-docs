---
title: Boot Sequence ([QS]boot)
sidebar: home_sidebar
permalink: boot-sequence.html
keywords: boot, seed, lsd, notecard, asset key
toc: true
---

`[QS]boot` is the one-shot LSD writer. It parses the `AVpos` notecard once per fresh boot and seeds the entire fork's persistent state into Linkset Data. After seeding, boot is idle until something invalidates the seed.

## State machine, in plain language

`state_entry` decides between two branches:

1. **Skip seed.** `qs:boot:asset` matches the AVpos notecard's current asset-key → LSD already has fresh data from a previous boot. sitA and sitB will read it directly. Boot does no parsing work.
2. **Fresh seed.** Otherwise → parse the AVpos notecard line by line, write `qs:cfg:<ch>`, `qs:sitter:<ch>`, `qs:p:<ch>:<i>`, `qs:meta:<ch>`, and **finally** `qs:boot:asset` so the marker is only written if everything before it succeeded.

After seeding completes, boot broadcasts `QS_BOOT_RELOAD` (90023) so any already-running sitB scripts re-read `MENU_LIST` from the freshly-written LSD instead of staying on the stale list from their last `state_entry`. Without this broadcast, a notecard re-save would require a manual reset on every sitB.

## Asset-key as durability marker

`qs:boot:asset` stores the notecard's UUID string as returned by `llGetInventoryKey("AVpos")`. Two properties make it the right primitive for "have we seeded this content already?":

- Re-uploading a notecard with **the same content** yields the **same asset-key** in Second Life — the viewer dedups identical assets. Skip-seed works.
- Editing the notecard and saving yields a **new asset-key**. The skip-check fails, boot re-seeds, and live `[HELPER] [SAVE]` edits applied to LSD between boots are deliberately overwritten by the notecard's current text.

`changed(CHANGED_INVENTORY)` clears `qs:*` and re-runs the seed path. Manual script reset / region restart hits the same code without the LSD wipe — if the marker survived, skip-seed runs.

## Boot self-check — 90077 / 90078

`[QS]boot` verifies the minimum base scripts are present in the linkset right after seeding. Two failure modes get surfaced as `llOwnerSay` errors so the creator catches a missing-script install before the first sit attempt instead of seeing a silent no-menu / no-animation furniture:

1. **Hard-fail.** `[QS]sitA` or `[QS]sitB` missing — no animation or no menu. Sets `llSetText` red so the prim is visibly broken in-world.
2. **Conditional warn.** AVpos has `PROP*` directives but `[QS]prop` is not installed — props won't be rezzed.

Adjuster presence is deliberately **not** checked. `[QS]sitA` already gates the `[HELPER]` menu item on `QS_ADJUSTER_HELLO` (90091), so an end-user (read-only) install just doesn't expose the Adjust path — nothing is broken from the user's view.

### Probes

| Num    | Direction | `msg` | `id` | Meaning |
|--------|-----------|-------|------|---------|
| 90077  | `[QS]boot` → `[QS]sitB` | `""` | `""` | Probe: "is the menu pipeline present?" Sent once from boot's `state_entry`. |
| 90078  | `[QS]sitB` → `[QS]boot` | `""` | `""` | Hello: reply to 90077. One reply per probe is enough; boot's handler only sets a flag. |

Detection has two complementary paths per base script: an explicit probe (90096 for sitA, 90077 for sitB) the script answers from its `link_message` handler, and an unsolicited HELLO emitted at the end of `qs_load_from_lsd()` in slot-0 sitA / slot-0 sitB. The probe covers the **skip-seed path** where boot is reset alone while sitA/sitB keep running (their `state_entry` doesn't re-fire, so no unsolicited HELLO); the unsolicited HELLOs cover the **fresh-boot path**, where `finalize_boot`'s 90023 broadcast triggers a fresh `qs_load_from_lsd()` in both base scripts.

PROP* detection rides on the existing notecard parser: one extra `if (command == "PROP1" || command == "PROP2" || command == "PROP3")` branch in `dataserver` sets `has_prop_in_notecard = TRUE`. The `[QS]prop` presence check reuses `dump_plugins` (populated by QSDUMP announces).

## DUMP cascade

Ownership of `[DUMP]` lives entirely in `[QS]boot`. Adjuster's involvement is exactly one line: the `[DUMP]` dialog handler sends 90098 to kick the chain.

Boot writes the `qs:cfg` / `qs:sitter` / `qs:p:*` keys during seed, so reading them back to dump is a natural fit. Both producer (streaming the LSD into 90022 messages) and receiver (formatting them into AVpos lines, chat output, HTTP upload to the AVsitter settings service) live there.

| Num   | Direction | `msg` | `id` | Meaning |
|-------|-----------|-------|------|---------|
| 90098 | `[QS]adjuster` → `[QS]boot` | `(string)channel` | `""` | "Start streaming this channel's dump." Sent on `[DUMP]` for channel 0; boot's own 90021 cascade re-sends it for each subsequent channel. |
| 90099 | `[QS]boot` → self | `(string)channel` | `""` | "Process the next pose entry for the channel currently being dumped." Self-trigger between ticks. |

State lives in two boot globals: `qs_dump_ch` (the channel being streamed, `-1` when idle) and `qs_dump_pi` (next entry index). Only one channel streams at a time.

## QSDUMP — plugin announce for the DUMP cascade

`[QS]boot`'s DUMP cascade used to hardcode the participating plugin script names. Once `[AV]prop` was forked into `[QS]prop`, that constant had to be edited too — and any third-party DUMP-capable plugin would still be invisible to the cascade without a boot patch. QSDUMP turns plugin discovery dynamic: plugins announce themselves, boot collects.

| Num   | Direction | `msg` | `id` | Meaning |
|-------|-----------|-------|------|---------|
| 90094 | `[QS]boot` → all plugins | `""` | `""` | QSDUMP probe — "if you're DUMP-capable, announce yourself now." Sent once from boot's `state_entry`. |
| 90095 | DUMP plugin → `[QS]boot` | `""` | `<script_name>` | QSDUMP hello — "I respond to 90020 DUMP messages addressed to my script name." Sent unsolicited from the plugin's `state_entry` and `on_rez`, and in response to 90094. |

Boot maintains `list dump_plugins` — a deduped list of announced plugin names. The 90021 cascade iterates `dump_plugins + [camera_script]` per channel; the camera script name stays hardcoded until `[QS]camera` is forked and adopts QSDUMP. Boot still `llGetInventoryType`-checks each name before sending 90020, so a stale announce (plugin script deleted from inventory) is silently skipped rather than hanging the cascade waiting for a 90021 echo that never comes.

A plugin that never announces still works in stock-AVsitter furniture (no boot → no listener); QSDUMP is purely additive on top of stock's 90020/90021/90022 contract.

### Migration status

- `[QS]prop` (≥ 0.020) — announces ✅ (also broadcasts QS_PROP_HELLO 90089 since 0.901 so `[QS]adjuster` can gate the `[PROP]` menu item without an inventory probe).
- `[QS]faces` (≥ 0.902) — announces ✅ (also broadcasts QS_FACES_HELLO 90090 so sitA / adjuster can gate the `[FACES]` / `[EXPRESSION]` menu items).
- `[AV]camera` — stock, hardcoded in boot's cascade. No `[QS]camera` fork planned: stock `[AV]camera`'s only name-bound code is dead, and all working paths are protocol-based and script-name-agnostic.

## See also

- [LSD Storage](lsd-storage.html) — full layout of what boot writes.
- [QSALIVE Discovery](qsalive-discovery.html) — the sitA-side handshake boot also uses for its self-check.
- [LinkMessage Numbers](linkmessage-numbers.html) — complete fork link-message map.
