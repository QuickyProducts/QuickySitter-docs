---
title: Debug Flags
sidebar: home_sidebar
permalink: debug-flags.html
keywords: debug, verbose, Out, stress, troubleshooting, diagnostics
toc: true
---

QuickySitter has three layers of diagnostics, in increasing order of how invasive they are:

1. The project-wide **`Out(level, …)` verbose ladder**: runtime logging that creators can dial up from the `AVpos` notecard without editing scripts.
2. The **`[QS]debug` script**: an owner-only `/88` chat tool that inspects Linkset Data and can fire synthetic load at the sitter chain.
3. Per-script **`bDebug`-style developer toggles**: compile-time flags a contributor flips while chasing a specific bug.

## The verbose ladder: `Out(level, msg)`

Every fork script that emits runtime chatter routes it through a small helper:

```lsl
// Verbose convention: 0=error/warn floor (default), 1=boot banner,
// 2=runtime status, 3=debug. OutForce() bypasses for critical messages.
integer verbose = 0;
Out(integer level, string msg)
{
    if (verbose >= level)
        llOwnerSay(llGetScriptName() + "[" + version + "] " + msg);
}
OutForce(string msg)   // unconditional, for must-see errors
{
    llOwnerSay(llGetScriptName() + "[" + version + "] " + msg);
}
```

The level scale is the same across the fork:

| Level | Meaning | Examples |
|-------|---------|----------|
| `0` | Error / warning floor: always shown. | boot "notecard missing" ERROR, offset "emergency shrink" WARN. |
| `1` | Boot / ready banner: once per script start. | `[QS]sitA[…] Ready, Mem=…`, offset `Ready. LSD room=…`. |
| `2` | Runtime status: per-operation chatter. | `Loading…` during a pose/expression load. |
| `3` | Debug: fine-grained internal state. | reserved for deep tracing. |

### Setting the level from the notecard

The single source is the `AVpos` notecard directive `VERBOSE n`. `[QS]boot` parses it during seed and writes it to the **`qs:cfg:verbose`** LSD key (a singleton, not per-channel). Every fork script reads that key in `state_entry` to initialise its own `verbose` global *before* its first `Out()` call:

```lsl
// in state_entry, before any Out(...)
string v = llLinksetDataRead("qs:cfg:verbose");
if (v != "") verbose = (integer)v;
```

Default is `VERBOSE 0`, silent except for the level-0 error/warning floor. Add `VERBOSE 1` to the `AVpos` notecard to see ready banners, `VERBOSE 2` for runtime status, `VERBOSE 3` for everything. Because the value lives in LSD, changing it affects every script at once and survives a script reset (boot won't clobber a user-chosen level on the skip-seed path).

### Why a ladder instead of bare `llOwnerSay`

Passive `llOwnerSay` calls multiply by N on furniture-heavy regions: each sitter slot's `[QS]sitA` would spam the owner every time a pose plays. Gating every line behind `Out(level, …)` keeps shipping builds quiet at the default floor and lets diagnostics turn on temporarily, region-wide, with one notecard token. Don't add bare `llOwnerSay` calls outside the ladder; route through `Out()` (or `OutForce()` for a genuine must-see error) so the verbosity stays under one knob. See [AVpos Reference](avpos-reference.html) for the `VERBOSE` token.

## `[QS]debug`: the `/88` LSD inspector

`[QS]debug.lsl` is an **owner-only chat tool**, not a link-message responder. Drop it into a QuickySitter prim and it listens on chat channel **`/88`** filtered to the owner; all output goes back to the owner via `llOwnerSay`. It does **not** receive or answer diagnostic link-messages, and other scripts never call into it. It is safe to leave in a prim (harmless if unused) and removes cleanly.

Type `/88 help` for the menu. Commands:

| Command | What it does |
|---------|--------------|
| `keys [pattern]` | List `qs:*` keys, or keys matching a regex. |
| `count <ch>` | Pose count for one channel. |
| `meta <ch>` | Show `qs:meta:<ch>` (the boot marker). |
| `cfg <ch>` | Show `qs:cfg:<ch>` (the 17-slot config blob). |
| `sitter <ch>` | Show `qs:sitter:<ch>`. |
| `pose <ch> <i>` | Show `qs:p:<ch>:<i>` with the fields parsed out (name / type / anim / pos / rot). |
| `poses <ch>` | Dump every pose row for a channel. |
| `grep <text>` | Find all `qs:p:*` rows whose value contains `<text>`. |
| `raw <key>` | Show the raw value of any LSD key. |
| `mem` | LSD bytes used / free, with a little usage bar. |
| `delch <ch>` | Delete all `qs:*:<ch>` keys for one channel (also drops `qs:boot:asset` so the next reset re-seeds). |
| `nuke` / `nuke yes` | Wipe **all** Linkset Data on the object (confirmation required). |
| `stress {start\|stop\|status\|speed}` | Synthetic-load generator (see below). |

This replaces the older idea of a central log-routing responder: inspection is pull-based (you ask `/88`), so there's no debug channel for other scripts to publish to. The polished, notecard-formatted equivalent of a bulk dump is still the `[DUMP]` button. See [Adjustment Workflow](adjustment-workflow.html).

## `[QS]debug` stress test

`/88 stress` makes `[QS]debug` impersonate up to **7 fake sitters** and fire synthetic sitter traffic at the rest of the linkset, to load-test hudproxy and the downstream `[QS]sitA` / `[QS]sitB` / `[QS]offset` chain. Fake UUIDs come from `llGenerateKey`, so each run exercises a fresh `QSO:<short>:*` keyspace in `[QS]offset`.

| Subcommand | Effect |
|------------|--------|
| `stress start [n]` | Spawn `n` fake sitters (default 6, max 7) and begin. |
| `stress stop` | Run the cleanup phase (one un-sit per tick). |
| `stress status` | Current phase, sitter/op counts, free LSD, debug script memory. |
| `stress speed <ms>` | Chaos-phase tick interval (min 100 ms). |

It runs in three timer-driven phases:

- **Ramp**: one fake sitter joins per tick: `90060` (sit) + `90070` (slot assign).
- **Chaos**: random ops per tick: ~70 % pose changes (`90055`), some quiet swaps (`90031`, the quiet variant so it doesn't also reopen pose menus), and the rest `90262` saves with non-zero offsets (so the `QSO:*` keyspace grows rather than hitting the ZERO/ZERO delete sentinel).
- **Cleanup**: one un-sit per tick (`90065`) until empty.

Caveat: if `[QS]debug` is reset mid-stress, the fake sitters orphan in hudproxy's state and consume listen slots, so reset hudproxy (or the prim) to recover. See [LinkMessage Numbers](linkmessage-numbers.html) for what each of those numbers means.

## Inspecting LSD by hand

If you don't have `[QS]debug` in the prim, the same idea from a temporary script or debug HUD:

```lsl
default
{
    touch_start(integer n)
    {
        // Dump all qs:* keys (slow, don't do this in shipping code)
        list keys = llLinksetDataFindKeys("^qs:", 0, 100);
        integer i;
        for (i = 0; i < llGetListLength(keys); ++i)
        {
            string k = llList2String(keys, i);
            llOwnerSay(k + " = " + llLinksetDataRead(k));
        }
    }
}
```

`llLinksetDataFindKeys` takes a regex over LSD keys. Use the `qs:` prefix as the anchor to scope the dump.

## Developer toggles (`bDebug`)

Before the verbose ladder, individual scripts used a binary `integer bDebug = FALSE;` guarding `debugSay(...)` calls. Most scripts have since migrated to `Out()`: for example `[QS]offset` replaced its `bDebug`/`debugSay` scheme outright, and its LRU-cache diagnostics now ride the ladder:

- Ready banner with current entry count: `Out(1, "Ready. LSD room=…")`.
- Emergency shrinks (memory pressure forcing RAM eviction before a save): `Out(0, "WARN: …")`, so they show even at the default floor.
- LSD writes refused because `lsdHasRoom()` returned FALSE.
- Sentinel deletes (ZERO/ZERO `90260` emitted).

Useful when debugging "the save didn't persist" reports: set `VERBOSE 1` (or `2`) to see them.

A bare `bDebug`-style compile-time flag is still a fine pattern when you're adding *throwaway* tracing to one script while chasing a bug, as long as it doesn't ship enabled. Conventions:

- **Ship quiet.** A developer flag defaults to FALSE; the notecard `VERBOSE` level is the supported user-facing knob.
- **Don't add bare `llOwnerSay` outside `Out()`.** Once a passive log lands it tends to stay and shows up in user chat forever.
- **Don't gate on runtime conditions** like `getNumberOfSitters() > 1`. Keep the toggle binary and obvious in the source.

## Recommended debug practice

When investigating a bug:

1. Add `VERBOSE 1` (or `2`/`3`) to the `AVpos` notecard and reset the prim, which lights up `Out()` across every script at once.
2. For state questions, drop in `[QS]debug` and poke at LSD with `/88 keys`, `/88 poses <ch>`, `/88 cfg <ch>`, `/88 mem`.
3. To reproduce load/offset bugs, `/88 stress start`, watch, then `/88 stress stop`.
4. **Remove the `VERBOSE` token (or set it to `0`) and any throwaway tracing before committing.** Review catches a stray passive log, but it's easier to do up front.

## See also

- [Contributing](contributing.html): why we don't ship passive `llOwnerSay` logs.
- [AVpos Reference](avpos-reference.html): the `VERBOSE` notecard directive.
- [LSD Storage](lsd-storage.html): what `qs:*` keys exist for inspection.
- [LinkMessage Numbers](linkmessage-numbers.html): the numbers the stress generator fires.
- [Boot Sequence](boot-sequence.html): how `qs:cfg:verbose` gets seeded.
