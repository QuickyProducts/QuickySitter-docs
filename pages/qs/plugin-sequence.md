---
title: '[QS]sequence'
sidebar: home_sidebar
permalink: plugin-sequence.html
keywords: sequence, animation chains, plugin
toc: true
---

`[QS]sequence` is a thin fork of stock `[AV]sequence` that adds the QSDUMP integration so animation-sequence entries are included in `[DUMP]` output.

Behavior is identical to stock from the user's perspective. Animation sequences (chained pose / animation runs with timing) are defined the same way in the notecard.

## Notecard syntax (unchanged from stock)

A sequence is a block of directives starting with `SEQUENCE <pose_or_label>`. Following `WAIT <seconds>` and `SOUND <name>|<flag>` lines belong to that step until the next `SEQUENCE` line.

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

## QS addition: QSDUMP announce

`[QS]sequence` broadcasts QSDUMP_HELLO (90095) on `state_entry`, `on_rez`, and in response to boot's QSDUMP_PROBE (90094). This registers the script for the DUMP cascade so `[DUMP]` output includes its SEQUENCE entries.

Stock `[AV]sequence` was hardcoded by name in `[AV]adjuster`'s dump cascade. The QS fork removes that hardcoding and lets sequence announce itself, so creator-renamed forks of sequence work without patching boot.

## Sound and music

`[QS]sequence` handles the `SOUND` directive too — toggling music playback per pose. See [upstream AVsequence](https://avsitter.github.io/avsitter2_sequence.html) for the syntax.

## Link messages

Stock-AVsitter numbers used unchanged:

| Num | Direction | Use |
|-----|-----------|-----|
| `90003` | sitA → sequence | Play overlay (sequence ignores 90003 to avoid re-triggering itself). |
| `90205` | any → sequence | Toggle sound. |
| `90210` | various | BUTTON-line default integer for sequence triggers. |

Plus QSDUMP additions:

| Num | Direction | Use |
|-----|-----------|-----|
| `90094` | `[QS]boot` → `[QS]sequence` | QSDUMP probe. |
| `90095` | `[QS]sequence` → `[QS]boot` | QSDUMP hello. |

## See also

- [Upstream AVsequence documentation](https://avsitter.github.io/avsitter2_sequence.html).
- [Animation Sequences](animation-sequences.html) — using sequences in multi-avatar setups.
- [Boot Sequence § QSDUMP](boot-sequence.html#qsdump--plugin-announce-for-the-dump-cascade).
