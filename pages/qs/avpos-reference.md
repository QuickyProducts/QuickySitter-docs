---
title: AVpos Notecard Reference
sidebar: home_sidebar
permalink: avpos-reference.html
keywords: avpos, notecard, syntax, directive, pose, menu
toc: true
---

QuickySitter reads the same `AVpos` notecard format as stock AVsitter 2. This page summarises the syntax and points out QS-specific behavior. For the complete tutorial-style upstream reference (with worked examples), see the [AVsitter2 AVpos page](https://avsitter.github.io/avsitter2_avpos.html).

## What the notecard does

A single notecard named **`AVpos`** in the same prim as `[QS]boot`. It defines:

- Sitter slot count (one slot per `[QS]sitA` script you have in inventory).
- Channel-level settings (BRAND, CUSTOM_TEXT, MTYPE, ETYPE, …).
- The pose list per slot — pose name, animation, default position, default rotation.
- Menu structure — submenus, top-level menu items, button definitions.
- Optional prop attachments, face animations, animation sequences.

## Top-level structure

Sections are introduced by uppercase directives at the start of a line:

```
SETUP
  channel-level settings
  …

POSE | SYNC | MENU
  per-pose data
  …

PROP1
  prop attachments
  …
```

`[QS]boot` parses the notecard line by line; unknown directives are ignored (forward-compat).

## SETUP directives

| Directive | Argument | Meaning |
|-----------|----------|---------|
| `MENU` (top of file) | — | Just the section marker. Top-level menu items follow as `TOMENU` entries. |
| `TOMENU` | menu label | A button that appears at the top of the dialog menu when the sitter has multiple sub-categories of poses. **Without a TOMENU there is no clickable top-level menu** — common gotcha. |
| `BRAND` | label | Free-text brand string shown in the menu header. |
| `CUSTOM_TEXT` | text | Custom message above the menu (escape special chars per stock rules). |
| `MTYPE` | integer | Menu type. See upstream docs for values. |
| `ETYPE` | integer | Exit type for stand-up behavior. |
| `SET` | integer | Number of sit-target sets (defaults to 1). |
| `SWAP` | 0/1/2 | Swap mode for couple poses. |
| `SELECT` | integer | Used by `[QS]select` for multi-furniture routing. |
| `AMENU` | integer | Adjustment-menu style. |
| `OLD_HELPER_METHOD` | 0/1 | Reverts to AVsitter 1 helper bar. |
| `WARN` | 0/1 | Print warning chat on bad notecard lines. |
| `HASKEYFRAME` | 0/1 | KeyFrame motion present (for motion props). |
| `REFERENCE` | name | Reference position by name (advanced). |
| `DFLT` | integer | Default sit-target index. |
| `RLVDesignations` | text | RLV path mapping. |
| `GENDERS` | CSV | Allowed gender CSV per slot. |

## POSE / SYNC / MENU sections

Each pose entry is a block of two to four lines:

```
POSE
NAME Sit casual
ANIM sit
POS <0.0, 0.0, 0.05>
ROT <0.0, 0.0, 0.0>
```

| Directive | Value |
|-----------|-------|
| `POSE` | Solo pose. Stored with the `P:` prefix in LSD. |
| `SYNC` | Multi-sitter sync pose. No `P:` prefix. Subject to [Re-Sync Protocol](resync-protocol.html). |
| `MENU` | Submenu marker. Following `POSE`/`SYNC` entries appear inside this submenu. |
| `NAME` | Pose name as shown in the menu. |
| `ANIM` | SL animation name (must be in the prim's inventory). |
| `POS` | Position offset `<x, y, z>` relative to sit-target. |
| `ROT` | Rotation Euler `<x, y, z>` in degrees. |

QS stores these as `qs:p:<ch>:<i>` LSD keys with format `name|type|anim|pos|rot` — see [LSD Keys](lsd-keys.html). The on-disk notecard is what creators edit; LSD is what runtime reads.

## PROP1 / PROP2 / PROP3 sections

Each entry attaches a prop on a specific pose:

```
PROP1
TRIGGER Sit casual
PROP CofeeMug
TYPE 1
POINT right hand
```

See the [`[QS]prop` page](plugin-prop.html) for the prop type matrix and the [`QSPROP_ATTACH` dynamic protocol](hud-integration.html#dynamic-prop-attach--qsprop_attach-90280) used by HUD addons.

## ANIM section ([AV]faces)

Used by `[QS]faces` to play overlay face animations:

```
ANIM
NAME smile
ON Sit casual
ANIMATION smile_face
```

## SEQUENCE section ([AV]sequence)

For multi-step animation chains:

```
SEQUENCE
NAME Wave
STEP wave_open 0.5
STEP wave_close 0.5
```

Full syntax in the upstream docs.

## BUTTON directives

`BUTTON` lines define `[AV]control` / RLV / `[AV]favs`-style buttons:

```
BUTTON [GIVE FOLDER]
COMMAND 90200
FOLDER MyFolder
```

The exact button-command mapping matches stock — see [LinkMessage Numbers](linkmessage-numbers.html) for the integer column.

## QS-specific behavior

- **Empty notecard.** A notecard with only `This notecard is empty` (or any single non-directive line) is valid and boot accepts it. The prim sits but plays no poses.
- **Asset-key tracking.** Boot stores the notecard's asset-key in `qs:boot:asset`. Re-saving the notecard with the same content keeps the same asset-key → boot skips re-parse. Editing the text → new asset-key → boot re-seeds. See [Boot Sequence](boot-sequence.html).
- **`[HELPER] [SAVE]` edits go to LSD, not notecard.** Stock auto-writes pose defaults nowhere. QS writes them to `qs:p:<ch>:<i>` so they survive script reset. The notecard is still the import source on next fresh boot, but edits made in-world are durable. Use `[DUMP]` to back up to the notecard format manually.

## Common gotchas

- **No TOMENU = no menu.** A `MENU` block without a top-level `TOMENU` button doesn't render. The button is what users click; the section is just a label.
- **Notecard not saved after creation.** A freshly created notecard that's never been saved is corrupt — `[QS]boot` will hang waiting on the dataserver event. Open, save, reset.
- **Mixed line endings.** AVpos accepts both CR and LF; CRLF works. If pasted from Windows externally, line breaks should be fine.
- **The viewer notecard editor truncates around 48 KB.** Edit large notecards externally. See [Known Limits](known-limits.html).

## See also

- [Upstream AVsitter2 AVpos reference](https://avsitter.github.io/avsitter2_avpos.html) — comprehensive tutorial.
- [Adjustment Workflow](adjustment-workflow.html) — using the `[HELPER]` menu to author poses interactively.
- [LSD Keys](lsd-keys.html) — how AVpos entries map to LSD.
- [Boot Sequence](boot-sequence.html) — when boot re-reads the notecard.
