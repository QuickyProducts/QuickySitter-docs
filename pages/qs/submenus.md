---
title: Submenus
sidebar: home_sidebar
permalink: submenus.html
keywords: menu, submenu, tomenu, navigation, dialog
toc: true
---

The pose dialog menu in QuickySitter follows stock AVsitter 2 semantics exactly. This page covers the conceptual model and points out a few pitfalls. For the full tutorial walk-through, see the [upstream AVsitter2 submenus page](https://avsitter.github.io/avsitter2_home.html).

## The model

A menu is built out of three constructs in the AVpos notecard:

| Directive | What it does | Visible to user? |
|-----------|--------------|------------------|
| `MENU` | Marks the start of a section. All `POSE`/`SYNC` entries below this line belong to the section. | ❌ — section marker only. |
| `TOMENU` | A clickable **button** at a higher menu level that, when pressed, opens the menu section it targets. | ✅ — this is the user-facing button. |
| `POSE` / `SYNC` | Individual pose entries inside a section. | ✅ — appear as buttons inside the submenu. |

**Key insight:** `MENU` and `TOMENU` are two different things. `MENU` says "a section starts here." `TOMENU` says "draw a clickable button here that opens that section." A `MENU` without a corresponding `TOMENU` is invisible — the section exists in the data model but no one can navigate to it.

## Structure example

```
SETUP
  TOMENU Couples
  TOMENU Solos

MENU Couples
  POSE
  NAME Cuddle
  ...

  SYNC
  NAME Hug
  ...

MENU Solos
  POSE
  NAME Sit
  ...

  POSE
  NAME Stand
  ...
```

Result: the top-level menu shows two buttons, `Couples` and `Solos`. Clicking `Couples` opens a submenu with `Cuddle` and `Hug`. Clicking `Solos` opens a submenu with `Sit` and `Stand`.

## Top-level menu

The top-level menu is built from `TOMENU` entries in the `SETUP` section. If you have only one section of poses, you don't need a TOMENU — sitB renders the poses directly. If you have multiple sections, every section needs a corresponding TOMENU at the top to be reachable.

A common bug: a creator adds a second `MENU Solos` section but forgets the second `TOMENU Solos` line. The Solos section exists in LSD but never appears in the menu.

## Menu pagination

The SL dialog menu shows 12 buttons per page. QS automatically paginates if you have more entries than fit: a `[NEXT]` button appears, and an implicit `[BACK]` button takes you up to the parent menu.

## Menu navigation messages

For plugin authors: menu choices are reported via LinkMsg 90050 (pose selection), 90051 (TOMENU selection), 90100 (menu choice), and 90101 (general menu choice). See [LinkMessage Numbers](linkmessage-numbers.html) for the payload format.

## QS-specific behaviors

- **`[NEW]` button appears in the helper-bar branch** when `[QS]adjuster` is present and the user is in HELPER mode. Click → enter a name → fresh pose stub gets written to LSD.
- **`[QUICKYHUD]` button** in the Adjust dialog appears only if `QPP_CFG:ADJUSTMODE` exists (set by hudproxy). See [HUD Integration](hud-integration.html).
- **`[ADJUST OFF]`** in the main pose menu appears only when `QPP_CFG:ADJUSTMODE == "On"`. Clicking sends 90266 `"Off"` to hudproxy.

## See also

- [AVpos Reference](avpos-reference.html) — full directive list.
- [Chat Commands](chat-commands.html) — text-based shortcuts that bypass the menu.
- [Adjustment Workflow](adjustment-workflow.html) — `[HELPER]` and its menus.
