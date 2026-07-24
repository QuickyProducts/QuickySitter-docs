---
title: Animation Sequences
sidebar: home_sidebar
permalink: animation-sequences.html
keywords: sequence, animation, chain, multi-step, plugin
toc: true
---

Animation sequences are multi-step animation chains, e.g., a "lovescene" that plays one pose for 30 s, then transitions to a second pose for 25 s, then loops. Implemented by the `[QS]sequence` plugin. Stock `[AV]sequence` runs too, but degrades on multi-sitter QS furniture: it counts sitters by probing for `[AV]sitA N` script names, which a QS prim doesn't have, so sequences only fire for slot 0. The fork exists to fix exactly that.

This page focuses on the QS-specific aspects. For the tutorial walk-through see the [upstream AVsequence page](https://avsitter.github.io/avsitter2_sequence.html).

## Notecard syntax (unchanged from stock)

Sequence definitions live in a dedicated **`[AV]sequence_settings`** notecard, read by `[QS]sequence` directly, not in the AVpos notecard. A `SEQUENCE <name>` line **starts a new named sequence**; the lines under it are its steps, until the next `SEQUENCE` line. `PLAY` is the directive that actually plays a pose — `SEQUENCE` only names the block.

```
SEQUENCE Lovescene
PLAY pose1
WAIT 30
PLAY pose2
SOUND moan|1.0
WAIT 25
LOOP
```

Step directives: `PLAY <pose>` (play a pose), `WAIT <seconds>` (hold, float), `SAY`/`WHISPER <text>` (chat), `SOUND <name>|<volume>` (play a sound — **field 2 is the volume**, 0.0–1.0, not a loop flag), and `LOOP` (jump back to the sequence's first step at the end). Without `LOOP`, the sequence stops after the last step and the final pose keeps looping.

## How it works

When a pose with a sequence is selected:

1. `[QS]sequence` looks up the sequence by name and starts its first step.
2. Steps run in order; a `WAIT` arms `llSetTimerEvent` for its duration, `PLAY` fires the pose.
3. `LOOP` at the end restarts the sequence; otherwise it stops and the last pose keeps looping.

The sequence pointer is a single global in `[QS]sequence` — **one sequence runs per furniture at a time**, not one per slot. It is stopped when any pose is played directly (90000/90008), on stand-up (90065), or on a swap (90030).

## Interaction with Re-Sync

LinkMsg 90271 re-phases pose animations, but only **SYNC** poses (those with no `P:` prefix). A sequence step that `PLAY`s a solo `POSE` entry is not re-phased; a step playing a SYNC pose is. So re-sync helps a sequence exactly when its current step is a shared couple pose.

See [Re-Sync Protocol](resync-protocol.html).

## Configuration notecard

The plugin's own `[AV]sequence_settings` notecard is never in `[DUMP]` output; the creator edits it directly. (Boot **does** round-trip the AVpos `SEQUENCE` launcher lines as `SEQUENCE <name>` in `[DUMP]` — but those live in AVpos and are handled by boot, not by this plugin.) Fork changes from stock `[AV]sequence`: the sitter-count query via QSALIVE (90096/90097) and the `Out()` verbose ladder. The product string is still the un-rebranded `"AVsitter™ sequence"`.

## Sound

A `SOUND` step plays one sound inside a sequence (`SOUND <name>|<volume>`, field 2 is the volume). It is a `[AV]sequence_settings` step directive — in an AVpos notecard `SOUND` is an unknown command and ignored. LinkMsg `90205` is the separate global sound on/off toggle.

## See also

- [`[QS]sequence`](plugin-sequence.html): plugin details.
- [Multi-Avatar Setups](multi-avatar.html): sequences across many sitters.
- [Re-Sync Protocol](resync-protocol.html): how 90271 interacts with running sequences.
- [Upstream AVsequence documentation](https://avsitter.github.io/avsitter2_sequence.html).
