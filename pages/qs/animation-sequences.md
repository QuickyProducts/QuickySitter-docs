---
title: Animation Sequences
sidebar: home_sidebar
permalink: animation-sequences.html
keywords: sequence, animation, chain, multi-step, plugin
toc: true
---

Animation sequences are multi-step animation chains — e.g., a "lovescene" that plays one pose for 30 s, then transitions to a second pose for 25 s, then loops. Implemented by the `[QS]sequence` plugin (or stock `[AV]sequence`, which is interchangeable).

This page focuses on the QS-specific aspects. For the tutorial walk-through see the [upstream AVsequence page](https://avsitter.github.io/avsitter2_sequence.html).

## Notecard syntax (unchanged from stock)

A sequence is a series of `SEQUENCE <pose_or_label>` lines, each followed by a `WAIT <seconds>` (and optionally a `SOUND <name>|<flag>` line). Each `SEQUENCE` line names the pose (or label) to play for that step; `WAIT` gives the step's duration.

```
SEQUENCE Lovescene
WAIT 30
SEQUENCE poseB
WAIT 25
SEQUENCE poseC
SOUND beat|1
WAIT 60
```

The sequence plays steps in order. After the final step, the last pose continues to loop until something else changes the sitter's animation. `SOUND <name>|<flag>` (where `flag = 1` loops) is optional per step.

`SEQUENCE`, `WAIT`, and `SOUND` are three independent directives — each on its own line. There is no `NAME` or `STEP` directive.

## How it works

When a pose with a sequence is selected:

1. `[QS]sequence` looks up the sequence by name and starts the first step.
2. A `llSetTimerEvent` fires at the step's `WAIT` duration.
3. On timer, the next `SEQUENCE` step plays.
4. After the last step, the timer stops and the last step's animation continues to loop.

The sequence pointer is per-slot in sitA's per-sitter globals. A sit-down resets it.

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
