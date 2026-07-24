---
title: '[QS]sequence'
sidebar: home_sidebar
permalink: plugin-sequence.html
keywords: sequence, animation chains, plugin
toc: true
---

`[QS]sequence` is a thin fork of stock `[AV]sequence`. Its fork-specific changes are two: it learns the sitter count over QSALIVE (90096/90097) instead of probing for sitter scripts by name, and it carries the project `Out()`/`OutForce()` verbose ladder (reads `qs:cfg:verbose`). It does **not** publish a `qs:alive:*` presence flag, does not announce on QSDUMP, and its product string is still the upstream `AVsitter(TM) sequence`.

Behaviour is otherwise identical to stock from the user's perspective. The step definitions in `[AV]sequence_settings` are the plugin's own notecard and are never in `[DUMP]` output. (Separately, boot **does** reconstruct the AVpos `SEQUENCE` *launcher* lines — the ones that put a sequence button in the menu — as `SEQUENCE <name>` in `[DUMP]`; those are AVpos content, handled entirely by boot, not by this plugin.)

## Notecard syntax

Sequences live in the dedicated `[AV]sequence_settings` notecard (the same separate notecard stock `[AV]sequence` uses), not in `AVpos`. Each `SEQUENCE <name>` line **starts a new named sequence**; the lines under it are its steps, until the next `SEQUENCE` line. The step directives are `PLAY`, `WAIT`, `SAY`, `WHISPER`, `SOUND`, and `LOOP`.

```
SEQUENCE Lovescene
PLAY pose1
WAIT 30
PLAY pose2
SOUND moan|1.0
WAIT 25
LOOP
```

| Directive | Meaning |
|-----------|---------|
| `PLAY <pose>` | Play a pose (fires 90003 to sitA). This is what advances the animation; `SEQUENCE` names the block, `PLAY` plays. |
| `WAIT <seconds>` | Hold for that many seconds before the next step (float). |
| `SAY <text>` / `WHISPER <text>` | Emit chat / whisper on channel 0. |
| `SOUND <name>\|<volume>` | Play a sound. **Field 2 is the volume** (float 0.0–1.0), not a loop flag. |
| `LOOP` | At the end of a sequence, jump back to its first step. Without it, the sequence stops after the last step and the final pose keeps looping. |

`DEBUG <n>` is a settings-level toggle (not a step). There is no `NAME` or `STEP` directive.

Full reference in the [upstream AVsequence page](https://avsitter.github.io/avsitter2_sequence.html).

## What the fork does *not* change

To be explicit, `[QS]sequence` deliberately does **not** add the features some of the other QS plugins have:

- **No QSDUMP announce.** It does not announce dump capability. Its config lives in `[AV]sequence_settings`, which the creator edits directly. (The AVpos `SEQUENCE` launcher lines are still round-tripped by boot's dump — that's boot, not this plugin.)
- **No `qs:alive:*` presence flag.** Unlike `[QS]prop`/`[QS]faces`, sequence does not publish a presence flag.
- **Un-rebranded product string.** The script still reports `product = "AVsitter(TM) sequence"`.

## Sound

The `SOUND` step plays one sound inside a sequence (`SOUND <name>|<volume>`). Separately, LinkMsg `90205` toggles whether sounds play at all (the sound on/off switch). The two are unrelated: `90205` is a global mute, `SOUND` is a per-step action.

## Link messages

`[QS]sequence` adds no link-messages of its own; all are stock-AVsitter numbers:

| Num | Direction | Use |
|-----|-----------|-----|
| `90003` | sequence → sitA | Play a pose from a `PLAY` step (LINK_THIS; ignored by sequence itself so it doesn't stop its own run). |
| `90205` | any → sequence | Toggle sound on/off. |
| `90210` | AVpos → boot | Default integer for a `SEQUENCE` line: boot turns it into a menu launcher button. |

## See also

- [Upstream AVsequence documentation](https://avsitter.github.io/avsitter2_sequence.html).
- [Animation Sequences](animation-sequences.html): using sequences in multi-avatar setups.
- [QSALIVE Discovery](qsalive-discovery.html): the sitter-count discovery sequence's one fork change relies on.
