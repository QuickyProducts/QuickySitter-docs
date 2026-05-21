---
title: Debug Flags
sidebar: home_sidebar
permalink: debug-flags.html
keywords: debug, bDebug, troubleshooting, diagnostics
toc: true
---

QuickySitter scripts can be made chatty for diagnostics via a single integer toggle per script. The convention:

```lsl
integer bDebug = FALSE;
```

`bDebug = TRUE` makes the script log internal state transitions via `debugSay(string)` (or equivalent), wrapped so the call is a no-op when the flag is FALSE.

## Why a flag instead of `llOwnerSay`

Passive `llOwnerSay` calls multiply by N on furniture-heavy regions — each sitter slot's `[QS]sitA` would spam the owner's chat every time a pose plays. Gating with `bDebug` keeps shipping builds quiet and lets diagnostics turn on temporarily without modifying call sites.

Conventions:

- **`bDebug` defaults to FALSE in shipped scripts.** Setting it TRUE is a local development action, not a setting the user is supposed to flip.
- **Don't add `llOwnerSay` calls outside the `bDebug` path.** Once a passive log lands, it tends to stay — and shows up in user-facing chat permanently.
- **Don't gate on `getNumberOfSitters() > 1` or similar runtime conditions.** The flag is binary and obvious in the source.

## `[QS]debug` — the dedicated diagnostics script

`[QS]debug.lsl` is a small in-prim diagnostics responder. With it present in the linkset, other scripts can call:

```lsl
llMessageLinked(LINK_SET, <debug_channel>, "Dumping state: " + (string)CURRENT_POSE_NAME, "");
```

`[QS]debug` formats and routes the message to the owner. The benefit over a direct `llOwnerSay` is that the chat target and verbosity are owned by one script, not scattered across nine.

(The exact debug channel and message contract is in [`[QS]debug.lsl`](https://github.com/QuickyProducts/QuickySitter/blob/master/qs/%5BQS%5Ddebug.lsl). It's intentionally minimal so it can be removed cleanly from shipped prims.)

## Inspecting LSD state

The most useful runtime diagnostic is reading LSD directly. From a debug HUD or a temporary script:

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

`llLinksetDataFindKeys` uses a regex over LSD keys. Use the `qs:` prefix as the anchor to scope the dump. The `[DUMP]` button does a more polished version of this for the configured `AVpos` notecard format — see [Adjustment Workflow](adjustment-workflow.html).

## bDebug in `[QS]offset`

`[QS]offset` carries `bDebug` for the LRU-cache state transitions:

- Ready (initial entry counts after `state_entry` and after `wipe_all_offsets`).
- Emergency shrinks (when memory pressure forces RAM eviction before a save).
- LSD writes refused due to `lsdHasRoom()` returning FALSE.
- Sentinel deletes (ZERO/ZERO 90260 emitted).

Useful when debugging "the save didn't persist" reports.

## Recommended debug practice

When investigating a bug:

1. Enable `bDebug` in the suspect script(s) locally.
2. Reset the prim, reproduce.
3. Inspect the chat output. The 50 µs per `bDebug` check is negligible.
4. **Set `bDebug = FALSE` again** before committing. The PR review catches this if you forget, but it's easier to do up-front.

## See also

- [Contributing](contributing.html) — why we don't ship passive `llOwnerSay` logs.
- [LSD Storage](lsd-storage.html) — what `qs:*` keys exist for inspection.
- [Boot Sequence](boot-sequence.html) — the `llOwnerSay` lines boot emits during seed.
