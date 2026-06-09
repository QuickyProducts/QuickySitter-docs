---
title: '[QS]sequence'
sidebar: home_sidebar
permalink: plugin-sequence.html
keywords: sequence, animation chains, plugin
toc: true
---

`[QS]sequence` is a very thin fork of stock `[AV]sequence`. Its **only** fork-specific change is that it learns the sitter count over QSALIVE (90096/90097) instead of probing for sitter scripts by name. It does **not** participate in the `[DUMP]` cascade, does not publish a `qs:alive:*` presence flag, and its product string is still the upstream `AVsitter(TM) sequence`.

Behaviour is otherwise identical to stock from the user's perspective. Animation sequences (chained pose / animation runs with timing) are defined the same way. Note that `[QS]sequence` reads its own separate `[AV]sequence_settings` notecard — **not** the main `AVpos` notecard — and SEQUENCE lines are therefore **not** part of the `[DUMP]` output.

## Notecard syntax (unchanged from stock)

Sequences live in the dedicated `[AV]sequence_settings` notecard (the same separate notecard stock `[AV]sequence` uses), not in `AVpos`. A sequence is a block of directives starting with `SEQUENCE <pose_or_label>`. Following `WAIT <seconds>` and `SOUND <name>|<flag>` lines belong to that step until the next `SEQUENCE` line.

```
SEQUENCE Lovescene
WAIT 30
SEQUENCE poseB
WAIT 25
SEQUENCE poseC
SOUND beat|1
WAIT 60
```

Each line is one directive. `WAIT` takes a single float (seconds); `SOUND <name>|<flag>` plays a sound on the step (`flag = 1` to loop). The sequence plays steps in order; after the last step the timer stops and the running pose continues to loop.

Full reference in the [upstream AVsequence page](https://avsitter.github.io/avsitter2_sequence.html).

## What the fork does *not* change

To be explicit, `[QS]sequence` deliberately does **not** add the features some of the other QS plugins have:

- **No `[DUMP]` participation.** It does not announce on QSDUMP (90094/90095) and SEQUENCE lines never appear in `[DUMP]` output. Its config lives in `[AV]sequence_settings`, which the creator edits directly.
- **No `qs:alive:*` presence flag.** Unlike `[QS]prop`/`[QS]faces`, sequence does not publish a presence flag.
- **Un-rebranded product string.** The script still reports `product = "AVsitter(TM) sequence"`.

The single fork change is sitter-count discovery over QSALIVE (90096/90097) in place of stock's script-name probing.

## Sound and music

`[QS]sequence` handles the `SOUND` directive too — toggling music playback per pose. See [upstream AVsequence](https://avsitter.github.io/avsitter2_sequence.html) for the syntax.

## Link messages

All stock-AVsitter numbers, used unchanged — `[QS]sequence` adds no link-messages of its own:

| Num | Direction | Use |
|-----|-----------|-----|
| `90003` | sequence → sitA | Play the next pose/animation in the sequence step. |
| `90205` | any → sequence | Toggle sound. |
| `90210` | various | BUTTON-line default integer for sequence triggers. |

## See also

- [Upstream AVsequence documentation](https://avsitter.github.io/avsitter2_sequence.html).
- [Animation Sequences](animation-sequences.html) — using sequences in multi-avatar setups.
- [QSALIVE Discovery](qsalive-discovery.html) — the sitter-count discovery sequence's one fork change relies on.
