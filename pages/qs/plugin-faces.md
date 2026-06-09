---
title: '[QS]faces'
sidebar: home_sidebar
permalink: plugin-faces.html
keywords: faces, face animations, expressions, plugin
toc: true
---

`[QS]faces` is a minimal fork of stock `[AV]faces` that publishes the QS presence flag `qs:alive:faces` so `[QS]sitB` and `[QS]adjuster` can gate `[FACES]` / `[EXPRESSION]` menu items without an inventory probe.

Behavior is otherwise identical to stock — drop a stock `[AV]faces` into a QS prim and faces still play, but with no `qs:alive:faces` flag the menu gating can't see it on a multi-sitter linkset; drop `[QS]faces` into a stock-AVsitter prim and it works too.

## Notecard syntax (unchanged from stock)

Face animations are declared with `ANIM` directives in `AVpos`, one per line. The format is:

```
ANIM <trigger>|<expression>|<duration>|<expression>|<duration>|...
```

Where `<trigger>` is a pose name (the face animation plays when that pose is selected), and each `<expression>|<duration>` pair specifies an SL face animation plus how long to hold it (in seconds). Up to three expressions per line is the practical limit.

Examples:

```
ANIM Happy|express_laugh_emote|5|express_smile|1|express_wink_emote|2
ANIM Sleep|express_disdain|1|express_smile|1
```

You can also reference an existing `ANIM` line by trigger name to reuse it:

```
ANIM pose1|express_laugh_emote|5|express_smile|1
ANIM pose2|pose1
```

Full reference in the [upstream AVfaces page](https://avsitter.github.io/avsitter2_faces.html).

## QS addition: presence via `qs:alive:faces`

`[QS]faces` advertises its presence by writing the LSD flag `qs:alive:faces` early in `state_entry` (the offset uses the inverted `qs:offset:alive`; faces uses the plain `qs:alive:faces`). It re-stamps the flag whenever boot broadcasts `QS_ALIVE_CENSUS` (90079), and removes nothing on its own — boot wipes all `qs:alive:*` on a census and only the survivors re-write, so a removed faces script simply stops re-stamping.

`[QS]sitB` and `[QS]adjuster` read the flag **on demand at menu-build time** — never cached — and gate menu items on it:

```lsl
// at menu build, on demand:
integer faces_present = (llLinksetDataRead("qs:alive:faces") != "");
```

If `qs:alive:faces` is unset when the menu is built, `[FACES]` / `[EXPRESSION]` buttons don't appear. No inventory probe and no script-name binding, so the gating works regardless of whether the script is named `[QS]faces`, `[FOO]faces`, or anything else.

The HELLO presence broadcasts (the retired `90088`–`90092` band, which once included a `QS_FACES_HELLO`) were removed in 0.9951 in favour of the `qs:alive:*` flags.

## Stock-diff summary

Total changes from stock `[AV]faces`:

1. **Presence via the `qs:alive:faces` LSD flag** — written in `state_entry`, re-stamped on `QS_ALIVE_CENSUS` (90079), read on demand. Same pattern as `[QS]prop`.
2. **QSDUMP integration** — announces on 90095 so face entries are included in `[DUMP]` output.
3. Version string + header comment block.

## See also

- [Upstream AVfaces documentation](https://avsitter.github.io/avsitter2_faces.html) — `ANIM` section syntax.
- [QSALIVE Discovery](qsalive-discovery.html) — sibling presence broadcasts.
- [LinkMessage Numbers](linkmessage-numbers.html) — fork-specific number map.
