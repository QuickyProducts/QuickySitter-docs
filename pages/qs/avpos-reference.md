---
title: AVpos Notecard Reference
sidebar: home_sidebar
permalink: avpos-reference.html
keywords: avpos, notecard, syntax, directive, pose, menu
toc: true
---

QuickySitter reads the same `AVpos` notecard format as stock AVsitter 2. This page summarises the syntax and points out QS-specific behavior. For tutorial-style examples, see the [upstream AVsitter2 AVpos reference](https://avsitter.github.io/avsitter2_avpos.html).

## What the notecard does

A single notecard named **`AVpos`** in the same prim as `[QS]boot`. It defines:

- Sitter slot count (one slot per `[QS]sitA` script in inventory).
- Channel-level settings (BRAND, CUSTOM_TEXT, MTYPE, ETYPE, …).
- The pose list per slot: pose name, animation, default position, default rotation.
- Menu structure: submenus, top-level menu items, button definitions.
- Optional prop attachments, face animations, animation sequences.

## Line format

Each line is **`<COMMAND> <arguments>`**. Most commands take their arguments as a single `|`-separated string after the command keyword:

```
POSE Sit casual|sit
TOMENU SITS
BUTTON [SWAP]|99
SITTER 1|Female
```

Unknown commands are silently ignored (forward-compat). Blank lines and comment-style lines (anything that isn't a recognised directive followed by data) are skipped.

There is no multi-line block syntax: `NAME`, `ANIM`, `POS`, `ROT` as standalone directives don't exist in the parser. The position/rotation of a pose lives on a separate `{<name>}<pos><rot>` line (see below).

## Global directives

These usually appear once near the top of the notecard. They apply **furniture-wide**: the parser reads them wherever they appear, the last value wins, and the same set of values is applied to every sitter slot. (Stock AVsitter parses them the same way; only `SITTER` itself and the pose lines below it are per-slot.)

| Directive | Argument | Meaning |
|-----------|----------|---------|
| `SITTER <n>` (or `SITTER <n>\|<info>`) | sitter slot index (0-based); optional info field can include `Male` / `Female` gender | Starts a new sitter channel. Subsequent POSE/SYNC/MENU/BUTTON lines belong to this slot until the next `SITTER` line. |
| `MTYPE <int>` | menu type | Dialog behavior. See upstream docs for values. |
| `ETYPE <int>` | exit type | Stand-up behavior. |
| `SET <int>` | set ID | Not a count. Tags seat assignments for prim-description pinning (`<set>-<slot>` in the prim description). Internal default -1 = auto-assign seats. See [SitTargets](sittargets.html). |
| `SWAP <int>` | 0/1/2 | Swap mode for couple/multi-sitter setups. |
| `SELECT <int>` | - | Read by `[QS]select` (optional, presence-gated) for multi-seat routing. |
| `AMENU <int>` | - | Adjustment menu style. |
| `HELPER <int>` | 0/1 | `1` reverts to the AVsitter-1-style helper bar. |
| `WARN <int>` | 0/1 | Print warning chat on bad notecard lines. |
| `VERBOSE <int>` | 0–3 | QS-specific. Diagnostic verbosity level; boot stores it in the `qs:cfg:verbose` LSD key. `0` = quiet, `3` = most verbose. |
| `KFM <int>` | 0/1 | KeyFrame motion present (for motion props). |
| `LROT <int>` | - | Local rotation flag (advanced). |
| `DFLT <int>` | 0/1 | `1` (default): seat reverts to its first pose when the last sitter stands up. `0`: the last chosen `POSE` stays as the new default. |
| `BRAND <text>` | brand label | Free-text shown in the menu header. |
| `ONSIT <text>` | - | onSit behavior. |
| `TEXT <text>` | - | Custom hovertext (escape `\n` for newlines). |
| `ADJUST <items>` | `\|`-separated menu entries | Customises the `[ADJUST]` submenu. |
| `ROLES <text>` | - | RLV designations. |

## Pose declarations

```
POSE <menu_name>|<animation_filename>
SYNC <menu_name>|<animation_filename>
```

- **`POSE`**: solo pose. Stored with the `P:` prefix in LSD.
- **`SYNC`**: multi-sitter sync pose. Same `<menu_name>` in two or more SITTERs plays them all together. Subject to [Re-Sync Protocol](resync-protocol.html). No `P:` prefix in LSD.

Example:

```
POSE Sit casual|sit
POSE Sit cross-legged|sit_generic
SYNC Cuddle|hug_female
```

## Position / rotation: `{<name>}<pos><rot>`

```
{Sit casual}<0.000000, 0.000000, 0.050000><0.000000, 0.000000, 0.000000>
```

This is a **position-update line**: it sets the default `pos` and `rot` for an already-declared pose. The pose name in `{…}` must match a previous `POSE` / `SYNC` line on the same SITTER.

Normally you don't hand-write these. `[HELPER] [SAVE]` writes them to LSD, and `[DUMP]` writes them back to the notecard. If hand-writing, the line goes anywhere after the `POSE <name>|<anim>` line for that pose.

In QS, this maps to `qs:p:<ch>:<i>` LSD keys with format `name|type|anim|pos|rot`; see [LSD Keys](lsd-keys.html). Boot ignores `{<name>}<pos><rot>` lines for pose names it can't find (no `qs_seed_find` match → silent skip).

## Menu structure

```
TOMENU <menu_name>
MENU <menu_name>
```

- **`TOMENU`**: a clickable button at a higher menu level that opens the named submenu.
- **`MENU`**: section marker. All `POSE`/`SYNC`/`TOMENU`/`BUTTON` lines below this line belong to the named submenu, until the next `MENU` line.

`MENU` and `TOMENU` are paired: `MENU SITS` is invisible without a corresponding `TOMENU SITS` above it. Common bug: adding a `MENU` section but forgetting the `TOMENU`. Then the section's poses exist in LSD but no button leads there.

Exception (deliberate): a `MENU` without `TOMENU` hides its poses from the dialog, useful for sequence-triggered or script-triggered poses. See [upstream "Hiding poses"](https://avsitter.github.io/avsitter2_sequence.html).

## BUTTON

```
BUTTON <label>|<integer>
```

Defines a clickable button that sends a link-message. `<integer>` is the LinkMsg number: `90200` is the AVprop default, `90401`/`90402`/`90403` are AVfavs commands, `99` is SWAP, etc. See [LinkMessage Numbers](linkmessage-numbers.html) for the canonical map.

```
BUTTON [SWAP]|99
BUTTON Add Fav|90401
BUTTON Quilt1
```

A `BUTTON` line without `|<integer>` defaults to integer `90200` (the AVprop rezz path).

## Prop attachments: `PROP`, `PROP1`, `PROP2`, `PROP3`

```
PROP <trigger>|<object>|<group>|<pos>|<rot>
PROP1 <trigger>|<object>|<group>|<pos>|<rot>|<attach_point>[|<scale>[|<wornpos>|<wornrot>]]
```

- `PROP`: ground prop (rezzed at the prim).
- `PROP1`: attachment prop (auto-attaches to the sitter's `<attach_point>`).
- `PROP2`: attachment prop, personal (COPY-TRANSFER NEXT).
- `PROP3`: special (persists across pose changes).

The trailing fields are optional (QS 1.25+): `<scale>` is a uniform size factor relative to the object's inventory size (empty or `1` = unchanged), `<wornpos>`/`<wornrot>` are the worn fit of an attachment prop, local to its attach point. They are written by `[DUMP]` when the prop was saved with the [prop scale & worn fit](plugin-prop.html#prop-scale-and-worn-fit-qsobjectadjust) feature (`[QS]objectadjust` companion in the prop); stock `[AV]prop` ignores them.

Example:

```
PROP Read|paper|G1|<0.550008, -0.001500, 0.142298>|<-134.045500, 75.901150, 44.793560>
PROP1 Dine|knife|G1|<0.387543, -0.311709, 0.173970>|<-0.017549, 9.899983, 90.102720>|Right Hand
```

See [`[QS]prop`](plugin-prop.html) for the prop type matrix, the `<group>` semantic, and the [`QSPROP_ATTACH` dynamic protocol](hud-integration.html#dynamic-prop-attach-qsprop_attach-90280) used by HUD addons.

## Face animations: `ANIM` (handled by `[QS]faces`)

```
ANIM <trigger_name>|<expression>|<duration>|<expression>|<duration>|...
```

Trigger is usually a pose name. Expression is an SL face animation; duration is a float (seconds). Up to three expressions per ANIM line are recommended.

```
ANIM Happy|express_laugh_emote|5|express_smile|1|express_wink_emote|2
```

Re-use an existing ANIM by referencing it by name:

```
ANIM pose1|express_laugh_emote|5|express_smile|1
ANIM pose2|pose1
```

See [`[QS]faces`](plugin-faces.html).

## `SEQUENCE` (launcher line, handled by `[QS]sequence`)

In the **AVpos** notecard, a `SEQUENCE` line is just a menu launcher: boot turns it into a button (default integer `90210`) that starts the named sequence:

```
SEQUENCE <name>
```

The actual step definitions (`PLAY`, `WAIT`, `SAY`, `WHISPER`, `SOUND`, `LOOP`) live in the **separate `[AV]sequence_settings` notecard**, not in AVpos (see [Animation Sequences](animation-sequences.html) for that grammar). Do **not** put `WAIT`/`SOUND` lines in AVpos; boot ignores them.

Unlike the step notecard, the AVpos `SEQUENCE` launcher lines **are** reconstructed in `[DUMP]` output (boot re-emits the `90210` button as `SEQUENCE <name>`).

See [`[QS]sequence`](plugin-sequence.html) and the [upstream AVsequence docs](https://avsitter.github.io/avsitter2_sequence.html) for the full sequence semantics.

## QS-specific behavior

- **Notecard must have at least one `SITTER`/`POSE` block.** boot derives the sitter-channel count from notecard content, not from installed scripts. A notecard with no pose-ish line seeds **zero** channels: boot reports "0 sitter(s) ready" and the sit scripts never leave their pre-boot state. Use the minimal block in [Getting Started](getting-started.html) as a floor.
- **Asset-key tracking.** Boot stores the notecard's asset-key in `qs:boot:asset`. Re-saving the notecard with the same content keeps the same asset-key → boot skips re-parse. Editing the text → new asset-key → boot re-seeds. See [Boot Sequence](boot-sequence.html).
- **`[HELPER] [SAVE]` edits go to LSD, not the notecard.** Stock AVsitter doesn't auto-write pose defaults anywhere; you `[DUMP]` and paste back. QS writes them to `qs:p:<ch>:<i>` LSD so they survive script reset and rerez. The notecard is the import source on next fresh boot, but in-world edits are durable. Use `[DUMP]` to back up to notecard form manually.

## Common gotchas

- **No TOMENU = no menu.** A `MENU` block without a top-level `TOMENU` button doesn't render. The button is what users click; `MENU` is just the section header.
- **`{<name>}<pos><rot>` without a matching POSE.** The position-update is silently dropped if no `qs_seed_find` match exists. Make sure the pose is declared first.
- **Notecard never saved after creation.** A freshly created notecard that's never been saved is corrupt, so boot will hang on the `dataserver` event. Open, save, reset.
- **Mixed line endings.** AVpos accepts both CR and LF; CRLF works. Pasted text from Windows is fine.
- **The viewer notecard editor truncates around 48 KB.** Edit large notecards externally and paste back. See [Known Limits](known-limits.html).
- **Bytes per line cap.** SL truncates notecard reads at 255 bytes per line. Very long ANIM / PROP lines may silently drop their tail.

## See also

- [Upstream AVsitter2 AVpos reference](https://avsitter.github.io/avsitter2_avpos.html): comprehensive tutorial with example notecards.
- [Adjustment Workflow](adjustment-workflow.html): using `[HELPER]` to author and `[SAVE]` poses interactively.
- [LSD Keys](lsd-keys.html): how AVpos entries map to `qs:p:<ch>:<i>` LSD.
- [Boot Sequence](boot-sequence.html): when boot re-reads the notecard.
