---
title: '[QS]faces'
sidebar: home_sidebar
permalink: plugin-faces.html
keywords: faces, face animations, expressions, plugin
toc: true
---

`[QS]faces` is a minimal fork of stock `[AV]faces` that adds the QS presence broadcast (90090) so `[QS]sitA` and `[QS]adjuster` can gate `[FACES]` / `[EXPRESSION]` menu items without an inventory probe.

Behavior is otherwise identical to stock — drop a stock `[AV]faces` into a QS prim and faces still work; drop `[QS]faces` into a stock-AVsitter prim and it works too.

## Notecard syntax (unchanged from stock)

The `ANIM` section in `AVpos` defines face animations:

```
ANIM
NAME smile
ON Sit casual
ANIMATION smile_face
```

Full reference in the [upstream AVfaces page](https://avsitter.github.io/avsitter2_faces.html).

## QS addition: QS_FACES_HELLO (90090)

| Num | Direction | `msg` | `id` | Meaning |
|-----|-----------|-------|------|---------|
| 90090 | `[QS]faces` → all | `""` | `<script_name>` | "I'm present and handle face animations." Sent on `state_entry`, `on_rez`, and in response to a slot-0 sitA QSALIVE-reply. |

`[QS]sitA` and `[QS]adjuster` latch this flag and gate menu items on it:

```lsl
integer QS_FACES_HELLO = 90090;
integer faces_present;

link_message(integer s, integer num, string msg, key id)
{
    if (num == QS_FACES_HELLO) {
        faces_present = TRUE;
    }
}
```

If `faces_present` is FALSE when the menu is built, `[FACES]` / `[EXPRESSION]` buttons don't appear. No inventory probe required, so the gating works regardless of whether the script is named `[QS]faces`, `[FOO]faces`, or anything else.

## Stock-diff summary

Total changes from stock `[AV]faces`:

1. **QSALIVE-based sitter presence** — same pattern as `[QS]prop`.
2. **Unsolicited 90090 broadcast** on `state_entry`, `on_rez`, and as part of QSALIVE-reply handling.
3. **QSDUMP integration** — announces on 90095 so face entries are included in `[DUMP]` output.
4. Version string + header comment block.

## See also

- [Upstream AVfaces documentation](https://avsitter.github.io/avsitter2_faces.html) — `ANIM` section syntax.
- [QSALIVE Discovery](qsalive-discovery.html) — sibling presence broadcasts.
- [LinkMessage Numbers](linkmessage-numbers.html) — fork-specific number map.
