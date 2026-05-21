---
title: Animation Sequences
sidebar: home_sidebar
permalink: animation-sequences.html
keywords: sequence, animation, chain, multi-step, plugin
toc: true
---

Animation sequences are multi-step animation chains — e.g., a "wave" sequence that runs `wave_open` for 0.5 s, then `wave_close` for 0.5 s, looping. Implemented by the `[QS]sequence` plugin (or stock `[AV]sequence`, which is interchangeable).

This page focuses on the QS-specific aspects. For the tutorial walk-through see the [upstream AVsequence page](https://avsitter.github.io/avsitter2_sequence.html).

## Notecard syntax (unchanged from stock)

```
SEQUENCE
NAME Wave
STEP wave_open 0.5
STEP wave_close 0.5
```

`STEP <animation> <duration_seconds>`. The sequence plays steps in order; after the last step, the last animation loops indefinitely (or until something else changes the sitter's pose).

For multi-sitter sequences, declare the sequence per slot with the same `NAME`. Each slot's section can have different STEP lines.

## How it works

When a pose with a sequence is selected:

1. `[QS]sitA` looks up the sequence by name and starts the first step.
2. A `llSetTimerEvent` fires at the step's duration.
3. On timer, `[QS]sitA` advances to the next step.
4. After the last step, the timer stops and the last step's animation continues to loop.

The sequence pointer (`SEQUENCE_POINTER`) is per-slot in sitA's per-sitter globals. A sit-down resets it.

## Interaction with Re-Sync

LinkMsg 90271 re-phases the **main** pose animation. For sequences, this means:

- The current STEP animation (whatever's playing now) gets the Stop+Start cycle.
- Steps that are about to play continue normally — the timer wasn't interrupted.

In practice, sequences with short steps (< 1 s) are visible-loop fast enough that drift between sitters is dominated by within-step phase, which 90271 handles. Long-step sequences (e.g., a 30-second slow-dance loop) benefit more from re-sync.

See [Re-Sync Protocol](resync-protocol.html).

## QSDUMP integration

`[QS]sequence` participates in the DUMP cascade via QSDUMP_HELLO (90095). `[DUMP]` output includes all SEQUENCE entries reconstructed from the plugin's state.

Stock `[AV]sequence` was hardcoded by name in stock adjuster's dump cascade. QS's dynamic announce protocol means a renamed or forked sequence plugin still gets picked up.

## Sound and music sequences

`[QS]sequence` also handles sound/music playback tied to poses — toggle via LinkMsg 90205, or via the `SOUND` directive in AVpos. See the upstream docs for the audio-related syntax.

## See also

- [`[QS]sequence`](plugin-sequence.html) — plugin details.
- [Multi-Avatar Setups](multi-avatar.html) — sequences across many sitters.
- [Re-Sync Protocol](resync-protocol.html) — how 90271 interacts with running sequences.
- [Upstream AVsequence documentation](https://avsitter.github.io/avsitter2_sequence.html).
