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

`[QS]boot` verifies the minimum base ingredients are present in the linkset right after seeding. Failure modes get surfaced as `llOwnerSay` errors so the creator catches a broken install before the first sit attempt instead of seeing a silent no-menu / no-animation furniture:

1. **Hard-fail.** `[QS]sitA` missing, `[QS]sitB` missing, **or** the `AVpos` notecard missing ([`[QS]boot.lsl:725-732`](https://github.com/QuickyProducts/QuickySitter/blob/master/qs/%5BQS%5Dboot.lsl)) — no animation, no menu, or nothing to seed. Sets `llSetText` red so the prim is visibly broken in-world. These three plus `[QS]boot` itself are the only mandatory ingredients; everything else is optional and presence-gated.
2. **Conditional warn.** AVpos has `PROP*` directives but `[QS]prop` is not installed — props won't be rezzed.

Adjuster presence is deliberately **not** treated as a failure. The `[HELPER]` / `[QUICKYHUD]` menu gate lives in `[QS]sitB`, keyed on the `qs:alive:adjuster` LSD flag, so an end-user (read-only) install just doesn't expose the Adjust path — nothing is broken from the user's view.

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
| 90098 | `[QS]adjuster` → `[QS]boot` | `(string)channel` | mode marker (`"quiet"` / `""`) | "Start streaming this channel's dump." Sent on `[DUMP]` for channel 0; boot's own 90021 cascade re-sends it for each subsequent channel. The `id` selects quiet vs. normal output. |
| 90099 | `[QS]boot` → self | `(string)channel` | `""` | "Process the next pose entry for the channel currently being dumped." Self-trigger between ticks. |

State lives in two boot globals: `qs_dump_ch` (the channel being streamed, `-1` when idle) and `qs_dump_pi` (next entry index). Only one channel streams at a time.

## QSDUMP — plugin announce for the DUMP cascade

`[QS]boot`'s DUMP cascade used to hardcode the participating plugin script names. Once `[AV]prop` was forked into `[QS]prop`, that constant had to be edited too — and any third-party DUMP-capable plugin would still be invisible to the cascade without a boot patch. QSDUMP turns plugin discovery dynamic: plugins announce themselves, boot collects.

| Num   | Direction | `msg` | `id` | Meaning |
|-------|-----------|-------|------|---------|
| 90094 | `[QS]boot` → all plugins | `""` | `""` | QSDUMP probe — "if you're DUMP-capable, announce yourself now." Sent once from boot's `state_entry`. |
| 90095 | DUMP plugin → `[QS]boot` | `""` | `<script_name>` | QSDUMP hello — "I respond to 90020 DUMP messages addressed to my script name." Sent unsolicited from the plugin's `state_entry` and `on_rez`, and in response to 90094. |

Boot maintains `list dump_plugins` — a deduped list of announced plugin names. The 90021 cascade iterates `dump_plugins + [camera_script]` per channel; the stock `[AV]camera` script name stays hardcoded because there is no `[QS]camera` fork to announce itself via QSDUMP. Boot still `llGetInventoryType`-checks each name before sending 90020, so a stale announce (plugin script deleted from inventory) is silently skipped rather than hanging the cascade waiting for a 90021 echo that never comes.

A plugin that never announces still works in stock-AVsitter furniture (no boot → no listener); QSDUMP is purely additive on top of stock's 90020/90021/90022 contract.

### Joining the cascade, step by step

Your plugin needs this if it keeps **its own directive lines in the AVpos notecard**: without joining the cascade, those lines are missing from the `[DUMP]` settings copy — and silently lost the next time the creator replaces AVpos with that dump. A plugin without notecard directives can skip all of this.

1. **Announce.** Send `90095` with your script name in `id` — from `state_entry`, from `on_rez`, and again whenever the `90094` probe arrives. Boot dedupes, so repeat announces are harmless.
2. **Answer `90020`.** During a dump, boot walks the announced scripts once per sitter channel, addressing each by name: `num == 90020`, `id` = your script name, `msg` = the channel. Emit the AVpos lines belonging to that sitter's section as `90022` link messages (`msg` = the line, `id` = the channel).
3. **Always echo `90021`** (`msg` = the channel, `id` = your script name) when you're done — even if you emitted nothing for that channel. Boot waits for the echo before moving on and uses your `id` to find its place in the walk.

```lsl
integer QSDUMP_PROBE = 90094;
integer QSDUMP_HELLO = 90095;

announce() { llMessageLinked(LINK_SET, QSDUMP_HELLO, "", llGetScriptName()); }

default
{
    state_entry()     { announce(); }
    on_rez(integer p) { announce(); }

    link_message(integer sender, integer num, string msg, key id)
    {
        if (num == QSDUMP_PROBE) { announce(); return; }
        if (num == 90020 && (string)id == llGetScriptName())
        {
            // msg = sitter channel being dumped. Emit this channel's
            // AVpos lines; per-furniture (global) lines go out once,
            // during the channel-0 pass.
            if ((integer)msg == 0)
                llMessageLinked(LINK_THIS, 90022, "SWING SPEED|2.0", msg);
            // ALWAYS echo — boot waits for this before moving on.
            llMessageLinked(LINK_THIS, 90021, msg, llGetScriptName());
        }
    }
}
```

Emitting many lines? Throttle (`llSleep(0.2)` between `90022` sends, like `[QS]faces` does) so boot's collector queue keeps up. Everything except the announce is the stock AVsitter dump round-trip — the `dump90098` capability token in the [QSALIVE reply](qsalive-discovery.html) just tells you the announce will actually be heard.

### Plugin participation

- `[QS]prop` — announces via QSDUMP ✅. Separately publishes the `qs:alive:prop` LSD flag so the `[PROP]` menu item can be gated without an inventory probe.
- `[QS]faces` — announces via QSDUMP ✅. Separately publishes the `qs:alive:faces` LSD flag so the `[FACES]` (sitB) / `[FACE]` (adjuster) menu items can be gated.
- `[AV]camera` — stock, hardcoded in boot's cascade. No `[QS]camera` fork planned: stock `[AV]camera`'s only name-bound code is dead, and all working paths are protocol-based and script-name-agnostic.

The old HELLO presence broadcasts (90088–90092: QS_OFFSET / PROP / FACES / ADJUSTER / SELECT_HELLO) were **retired in 0.9951** and replaced by the `qs:alive:<name>` LSD-flag model — flags are written in `state_entry`, re-stamped on the `QS_ALIVE_CENSUS` (90079) sweep, and read on demand at menu-build time. The retired numbers are reserved, not reused. See [QSALIVE Discovery](qsalive-discovery.html).

## See also

- [LSD Storage](lsd-storage.html) — full layout of what boot writes.
- [QSALIVE Discovery](qsalive-discovery.html) — the sitA-side handshake boot also uses for its self-check.
- [LinkMessage Numbers](linkmessage-numbers.html) — complete fork link-message map.
