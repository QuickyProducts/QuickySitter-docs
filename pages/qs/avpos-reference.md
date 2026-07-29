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

There is no multi-line block syntax. A pose is one line, and its position and rotation live on a separate `{<name>}<pos><rot>` line (see below); `NAME`, `POS` and `ROT` as standalone directives don't exist in the parser. (`ANIM` does exist, but it defines a facial expression rather than a pose attribute: see [Face animations](#face-animations-anim-handled-by-qsfaces).)

## Global directives

These usually appear once near the top of the notecard. They apply **furniture-wide**: the parser reads them wherever they appear, the last value wins, and the same set of values is applied to every sitter slot. (Stock AVsitter parses them the same way; only `SITTER` itself and the pose lines below it are per-slot.)

| Directive | Argument | Meaning |
|-----------|----------|---------|
| `SITTER <n>` (or `SITTER <n>\|<label>\|<gender>`) | sitter slot index (0-based); optional label; optional gender `M` or `F` | Starts a new sitter channel. Subsequent POSE/SYNC/MENU/BUTTON lines belong to this slot until the next `SITTER` line. See [The SITTER line in detail](#the-sitter-line-in-detail). |
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

## The SITTER line in detail

```
SITTER <n>|<label>|<gender>
```

The gender is the **third** field, and only `M` or `F` are recognised (exact,
case-sensitive). Both trailing fields are optional, but their *positions* are
not:

| Line | Label | Gender |
|------|-------|--------|
| `SITTER 0` | – | – |
| `SITTER 0\|Driver` | `Driver` | – |
| `SITTER 0\|F` | `F` | – |
| `SITTER 0\|\|F` | – | `F` |
| `SITTER 0\|Driver\|M` | `Driver` | `M` |

So a gender with no label keeps the empty label field. `SITTER 0|F` makes `F`
the label and leaves the slot genderless; there is no "a single suffix means
gender" rule. Longer spellings like `Male` or `Female` are **not** recognised.

When an avatar sits, `[QS]sitA` puts them in the first unoccupied slot whose
gender matches their shape. Slots without a gender match anyone.

{% include warning.html content="A trailing space used to be fatal here: `SITTER 0|F|F ` parsed as gender -1 before 1.27, and the seat went to the next gender-matching slot instead. boot trims both ends now, but a notecard carrying trailing spaces is still worth cleaning." %}

## Pose declarations

```
POSE <menu_name>|<animation_filename>
SYNC <menu_name>|<animation_filename>
POSE <menu_name>|<animation_filename>|<M or F>
```

- **`POSE`**: solo pose. Stored with the `P:` prefix in LSD.
- **`SYNC`**: multi-sitter sync pose. Same `<menu_name>` in two or more SITTERs plays them all together. Subject to [Re-Sync Protocol](resync-protocol.html). No `P:` prefix in LSD.

Example:

```
POSE Sit casual|sit
POSE Sit cross-legged|sit_generic
SYNC Cuddle|hug_female
```

A trailing `M` or `F` marks that pose as the default for a sitter of that shape
gender. It needs an animation in front of it: in `POSE Sit relaxed|F` the `F`
becomes the animation name, not a marker.

```
POSE Sit relaxed|sit_relaxed|F
```

**Names are truncated to 23 characters.** boot cuts `POSE`, `SYNC`, `MENU`,
`TOMENU` and `BUTTON` names at 23, so a longer name silently becomes a
different name than the one a `{<name>}` line or a `PROP` trigger refers to.

**Names are trimmed on both ends**, on the pose line and in the `{<name>}`
lookup, so `SYNC  Relax` (two spaces) and `{Relax}` are the same pose.

**A duplicate name inside one sitter is legal but ambiguous.** Each pose gets
its own LSD row and `[QS]sitB` dispatches by index, so every button plays its
own animation. Only the name-keyed bindings collapse onto the first occurrence:
the `{<name>}` position, a `PROP` trigger and an `ANIM` trigger.

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

Defines a clickable button that sends a link-message when clicked. `<integer>`
is the LinkMsg number: `90030` is SWAP, `90200` is the AVprop default,
`90210` starts a sequence, `90401`/`90402`/`90403` are AVfavs commands. See
[LinkMessage Numbers](linkmessage-numbers.html) for the canonical map.

```
BUTTON [SWAP]|90030
BUTTON Add Fav|90401
BUTTON Quilt1
```

A `BUTTON` line without `|<integer>` defaults to integer `90200` (the AVprop
rezz path).

### Extra fields after the number

Everything past the number is payload for whoever listens. The receiver gets it
as the link-message string, so one script can serve several buttons and switch
on the label:

```
BUTTON 'Swap F>M'|90030|0|1
BUTTON 'Swap Girls'|90030|0|2
```

Those two are SWAPs between specific seat pairs: `90030` takes the two slot
indexes as payload.

### Third-party numbers

Creators wire add-on products in through this mechanism, usually on a number of
the vendor's own choosing. Numbers outside the `90000`–`90500` band are a
reliable sign of that:

```
BUTTON [BENTOFACE]|-31450010
BUTTON ✘ Lock/Kick|99
```

Those are not AVsitter features and nothing QuickySitter ships listens for
them. They work only while the matching third-party script sits in the same
linkset. `99` in particular is **not** a stock number of any kind, despite
appearing in some notecards in the wild.

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

### `<attach_point>` uses AVsitter's spelling, not the viewer's

`[QS]prop` carries its own list of 40 point names and resolves the field by
uppercasing it and taking the first list name that occurs as a **substring** of
it. Matching is therefore case-insensitive and forgiving about extra words:
`right hand`, `Right Hand` and `on my right hand` all work.

What it is not forgiving about is a different word. Several viewer names differ
from AVsitter's:

| Viewer says | AVsitter wants |
|-------------|----------------|
| Skull | `head` |
| Spine | `back` |
| Belly | `stomach` |
| Left Pec | `left pectoral` |
| L Forearm | `left lower arm` |
| Left Eyeball | `left eye` |

A field that matches nothing resolves to point **0** and the prop attaches
nowhere. Two spellings seen in real notecards that silently fail this way:
`left forearm` (wants `left lower arm`) and `Pelvic` (wants `pelvis`).

See [`[QS]prop`](plugin-prop.html) for the prop type matrix, the `<group>` semantic, and the [`QSPROP_ATTACH` dynamic protocol](hud-integration.html#dynamic-prop-attach-qsprop_attach-90280) used by HUD addons.

## Face animations: `ANIM` (handled by `[QS]faces`)

```
ANIM <trigger_name>|<expression>|<duration>|<expression>|<duration>|...
```

The trigger is a pose name: `[QS]faces` starts the sequence when it sees the pose-played broadcast (`90045`) for that name. Everything after the trigger is read strided, as expression / duration pairs. Up to three expressions per ANIM line are recommended.

```
ANIM Happy|express_laugh_emote|5|express_smile|1|express_wink_emote|2
```

### Duration has three forms

| Duration | Effect |
|----------|--------|
| `5` | Hold the expression for five seconds. |
| `-5` | Re-apply the expression every second for five seconds. Facial expressions decay in SL, so this is what a long expression usually wants. |
| `-` | Stop the sequence after playing, rather than looping back to the start. |
| *(omitted)* | Legal. The field is simply absent, and `[QS]faces` reads an empty duration. |

### An expression field can also name another ANIM

```
ANIM pose1|express_laugh_emote|5|express_smile|1
ANIM pose2|pose1
```

`pose2` re-uses `pose1`'s whole sequence. This is resolved at **play** time, not while reading the notecard: on `90045`, `[QS]faces` takes the matched entry's sequence text and looks for another ANIM in the same sitter whose *trigger* equals it. Two consequences worth knowing:

- The fallback is silent. If no ANIM by that name exists, the field is started as an **animation name** instead, which is how custom facial animations in the prim are used: `ANIM Kiss|MY_CUSTOM_FACE` plays the animation `MY_CUSTOM_FACE`.
- So a single field after the trigger is inherently ambiguous, and both readings are valid. Nothing warns you either way; if a name is neither another ANIM nor an animation in the prim, the expression simply does not play.

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
- **A notecard cannot exceed 65 536 bytes**, which is a storage limit rather than a reading one: `llGetNotecardLine` documents no total-size limit. Below it a notecard works, however large; a 50 032-byte AVpos opens complete in-world. The viewer editor's own cutoff bites on the **save** path, not on opening, so for notecards around 50 KB and up, paste the whole content back in one go rather than editing in-world. See [Known Limits](known-limits.html).
- **Bytes per line cap: 1024.** `llGetNotecardLine` returns at most 1024 bytes of a line and drops the rest, so a very long chain of animation names or a fully loaded `PROP` line can lose its tail. It was 255 bytes until server 2021-10-25, and older documentation (including this page) still quoted that figure.

## See also

- [Upstream AVsitter2 AVpos reference](https://avsitter.github.io/avsitter2_avpos.html): comprehensive tutorial with example notecards.
- [Adjustment Workflow](adjustment-workflow.html): using `[HELPER]` to author and `[SAVE]` poses interactively.
- [LSD Keys](lsd-keys.html): how AVpos entries map to `qs:p:<ch>:<i>` LSD.
- [Boot Sequence](boot-sequence.html): when boot re-reads the notecard.
