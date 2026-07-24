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
| `MENU` | Marks the start of a section. All `POSE`/`SYNC` entries below this line belong to the section. | ❌ (section marker only). |
| `TOMENU` | A clickable **button** at a higher menu level that, when pressed, opens the menu section it targets. | ✅ (this is the user-facing button). |
| `POSE` / `SYNC` | Individual pose entries inside a section. | ✅ (appear as buttons inside the submenu). |

**Key insight:** `MENU` and `TOMENU` are two different things. `MENU` says "a section starts here." `TOMENU` says "draw a clickable button here that opens that section." A `MENU` without a corresponding `TOMENU` is invisible: the section exists in the data model but no one can navigate to it.

## Structure example

```
TOMENU Couples
TOMENU Solos

MENU Couples
POSE Cuddle|cuddle_anim
SYNC Hug|hug_anim

MENU Solos
POSE Sit|sit_anim
POSE Stand|stand_anim
```

Each directive is one line, with `|` separating arguments. `POSE <menu_name>|<animation>` declares a pose; the position is set later by a separate `{<menu_name>}<pos><rot>` line (normally written by `[HELPER] [SAVE]` / `[DUMP]`).

Result: the top-level menu shows two buttons, `Couples` and `Solos`. Clicking `Couples` opens a submenu with `Cuddle` and `Hug`. Clicking `Solos` opens a submenu with `Sit` and `Stand`.

## Top-level menu

The top-level menu is built from `TOMENU` entries above the first `MENU` line. If you have only one section of poses, you don't need a TOMENU, and sitB renders the poses directly. If you have multiple sections, every section needs a corresponding TOMENU above to be reachable.

A common bug: a creator adds a second `MENU Solos` section but forgets the second `TOMENU Solos` line. The Solos section exists in LSD but never appears in the menu.

## Menu pagination

An SL dialog has 12 button slots, and the number left for pose entries is **dynamic**: it is 12 minus whatever nav and control buttons the current menu renders. In a submenu a `[BACK]` button is always present (it takes you up to the parent menu). With the default `AMENU=2` the menu also carries an `[ADJUST]` button, so a typical submenu leaves **10 pose slots** (8 once paging is active). The bare minimum case (no `[ADJUST]`, so only `[BACK]` consuming a slot) is 11, dropping to 9 with paging. When a section has more entries than fit, QS paginates: `[<<]` and `[>>]` paging buttons appear and consume two more slots. The navigation buttons are `[<<]` / `[>>]`, and there is no `[NEXT]` button.

On the top-level pose menu, control buttons (`[OPTIONS]`, `[SWAP]`, plugin-registered buttons, etc.) consume slots first, before poses are laid out.

## Menu navigation messages

For plugin authors: menu choices are reported via LinkMsg 90050 (pose selection), 90051 (TOMENU selection), 90100 (menu choice), and 90101 (general menu choice). See [LinkMessage Numbers](linkmessage-numbers.html) for the payload format.

## QS-specific behaviors

- **`[NEW]` button appears** when `[QS]adjuster` is present and the user is in HELPER mode (or in QuickyHUD ADJUSTMODE). Clicking it opens a **type picker** (`[POSE]`/`[SYNC]`/`[SUBMENU]`/`[PROP]`/`[FACE]`/`[CAMERA]`). For a pose you then pick the animation(s) and confirm with `[DONE]`, after which a text box prompts for the entry name; the new entry is inserted into the menu at that point.
- **`[QUICKYHUD]` button** in the Adjust dialog is gated on the full chain: the caller passes the **Adjust ACL** (`qs:sec:adjust`, owner always passes, widenable to GROUP/ALL via `[SECURITY]`), `qs:alive:adjuster` is set, the `QPP_CFG:ADJUSTMODE` LSD key exists, and `qs:hud:unlicensed` is not `"1"`. See [HUD Integration](hud-integration.html).
- **`[DONE]`** in the main pose menu appears in HELPER mode or QuickyHUD ADJUSTMODE (`QPP_CFG:ADJUSTMODE == "On"`). Clicking exits the mode (sitB broadcasts 90100 `[DONE]`, `[QS]adjuster` does the tear-down, including 90266 `"Off"` to hudproxy) and opens the adjust submenu.

## See also

- [AVpos Reference](avpos-reference.html): full directive list.
- [Chat Commands](chat-commands.html): text-based shortcuts that bypass the menu.
- [Adjustment Workflow](adjustment-workflow.html): `[HELPER]` and its menus.
