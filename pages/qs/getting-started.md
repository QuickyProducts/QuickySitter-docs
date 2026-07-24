---
title: Getting Started
sidebar: home_sidebar
permalink: getting-started.html
keywords: install, setup, getting started, basic
toc: true
---

This page walks through getting QuickySitter scripts running in a Second Life or OpenSim furniture prim. It assumes basic familiarity with AVsitter 2. If you've never used AVsitter, the [upstream AVsitter2 instructions](https://avsitter.github.io/avsitter2_home.html) cover the conceptual foundation (poses, menus, sitters, the AVpos notecard).

## What you need

- The QuickySitter scripts. The current source is at [github.com/QuickyProducts/QuickySitter](https://github.com/QuickyProducts/QuickySitter); a packaged in-world distribution is available on the SL Marketplace (link to be added once the QS Marketplace store is live).
- A Second Life viewer with Mono compilation. Firestorm and the official viewer both work; the only difference is how you transfer scripts (Firestorm saves Mono-by-default; other viewers need the Ctrl-drop workaround documented in the [stock import guide](https://github.com/QuickyProducts/QuickySitter/blob/master/avstock/IMPORT_GUIDE.md)).
- An `AVpos` notecard with at least one `SITTER`/`POSE` block (a ready-to-paste minimal one is in step 3 below). The notecard format is identical to stock AVsitter; see [AVpos Reference](avpos-reference.html).

## Minimum script set

The smallest install that gets you a working QuickySitter prim is three scripts plus a notecard:

1. **`[QS]boot`**: the LSD seeder. Parses `AVpos` once per fresh boot and writes the persistent state.
2. **`[QS]sitA`**: main pose-and-sit script. One per sitter slot, named `[QS]sitA`, `[QS]sitA 1`, `[QS]sitA 2`, etc. The numbering starts at ` 1` for the second instance and must be contiguous; a gap breaks the slot count.
3. **`[QS]sitB`**: menu-and-state companion to sitA. Must match the count and numbering of the sitA scripts (`[QS]sitB`, `[QS]sitB 1`, ...).
4. **an `AVpos` notecard**: boot's self-check ERRORs (red hovertext) if it is missing. boot seeds `qs:cfg`/`qs:p`/`qs:meta`/`qs:sitter` from it (later `[HELPER]` `[SAVE]`/`[NEW]` edits update `qs:p` live via `[QS]adjuster`), so without the notecard sitA/sitB never leave their pre-boot state.

Everything else is optional and presence-gated: each plugin announces itself via a `qs:alive:<name>` LSD flag, and the feature degrades silently (or with a warn) when absent:

- **`[QS]select`**: sit-time seat-select picker for multi-sitter / multi-furniture setups. sitB has a built-in picker, so this is optional; sitB reads `qs:alive:select` (falling back to an `[AV]select` inventory probe for stock-AVsitter compat).
- **`[QS]adjuster`**: enables the `[HELPER]` menu and live `[SAVE]`-to-LSD writing. Without it, you can still play poses, but creators can't fine-tune positions in-world.
- **`[QS]prop`**: handles `PROP*` directives in the notecard and the dynamic-prop attach protocol used by HUD addons.
- **`[QS]faces`**: handles face/expression animations.
- **`[QS]offset`**: dedicated personal-offset storage. It owns the `CUSTOMS` store; without it there is **no** personal-offset persistence and no sitA fallback.
- **`[QS]sequence`**: animation sequences.

The stock AVsitter plugins (`[AV]camera`, `[AV]favs`, `[AV]helperscript`) work unchanged inside a QuickySitter linkset; see [Compatibility Matrix](compatibility-matrix.html). QS forks its own root family (`[QS]root`, `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV`); there is no `[QS]camera` fork (stock `[AV]camera` is used as-is).

## Step-by-step

1. **Import the scripts.** Follow the same procedure as the [stock AVsitter import guide](https://github.com/QuickyProducts/QuickySitter/blob/master/avstock/IMPORT_GUIDE.md). The only difference is the script names: `[QS]sitA` instead of `[AV]sitA`, etc. Mono must be enabled when saving (Firestorm does this automatically).
2. **Prepare a host prim.** Any modifiable prim works. Rez a box.
3. **Create the AVpos notecard.** In your inventory, create a new notecard, name it exactly `AVpos`, and paste this minimal content (it uses the built-in SL `sit` animation, so no animation assets are needed):

   ```
   SITTER 0|Sitter

   POSE Sit|sit
   {Sit}<0.0, 0.0, 0.5><0.0, 0.0, 0.0>
   ```

   **Save it.** boot derives the channel count from the notecard content, so at least one `SITTER`/`POSE` block is required: a notecard without one seeds **zero** channels and the sit scripts stay in their pre-boot state.
4. **Drop the scripts into the prim.** Drag all `[QS]*` scripts plus the `AVpos` notecard into the prim's contents.
5. **Verify boot.** boot parses the notecard and reports `Load complete; 1 sitter(s) ready...` in owner chat, writing one `qs:meta:<ch>` per `SITTER` block found in the notecard. The channel count comes from the notecard, not from the number of installed scripts. If the `AVpos` notecard is missing entirely, the self-check sets red hovertext and the prim stays unresponsive.
6. **Sit on the prim.** You should be seated in the built-in `sit` animation. Touch the prim to bring up the menu: your `Sit` pose plus the built-in entries (`[ADJUST]`, `[STOP]`, ...).

## Adding your first pose

Drop your animations into the prim, then, with `[QS]adjuster` installed, pick `[HELPER]` → `[NEW]` from the menu. A type picker opens (`[POSE]`, `[SYNC]`, `[SUBMENU]`, ...); choose `[POSE]`, pick the animation from the numbered picker, and enter the menu name in the text-box dialog. The new pose is written to LSD immediately. Adjust the position with the helper arrows; click `[SAVE]` to persist the new default.

For a richer authoring workflow, the [QuickyHUD addon](hud-integration.html) provides per-axis nudge buttons and a live offset preview.

## Editing AVpos directly

You can also edit the `AVpos` notecard by hand:

```
MTYPE 2

POSE Sit casual|sit
{Sit casual}<0.0, 0.0, 0.05><0.0, 0.0, 0.0>
```

Each directive lives on its own line, with `|` separating the arguments of a single directive. `POSE <name>|<animation>` declares the pose; `{<name>}<pos><rot>` sets the position and rotation for an already-declared pose. The position/rotation line is normally written by `[HELPER] [SAVE]` or `[DUMP]`, so you usually don't hand-write it.

Save the notecard, and boot detects the asset-key change via `changed(CHANGED_INVENTORY)`, re-seeds LSD, and broadcasts the reload so sitB picks up the new pose list without a manual reset. See [AVpos Reference](avpos-reference.html) for the full directive list.

## Troubleshooting

| Symptom | Likely cause |
|---------|--------------|
| Prim is silent on sit | `[QS]sitA` or `[QS]sitB` missing → boot self-check should have shouted; check your inventory. |
| Red hovertext: "AVpos notecard missing" | No `AVpos` notecard in the prim. boot's self-check hard-fails, so add a saved `AVpos` notecard and reset. |
| `llOwnerSay`: "[QS]sitA: missing [QS]sitB" | sitter-count mismatch: same number of sitA and sitB scripts required. |
| Boot reports "Load complete; 0 sitter(s) ready" and sitting does nothing | The `AVpos` notecard has no `SITTER`/`POSE` content, so zero channels were seeded. Add at least the minimal block from step 3 and save the notecard. |
| `[PROP]` button missing despite `PROP*` lines in AVpos | `[QS]prop` not installed. Boot self-check warns about this case. |
| `[FACES]` button missing | `[QS]faces` not installed. |
| Adjuster arrows make no visible change | Check you're adjusting the intended slot: the owner chat command `/5 targets` labels each seat prim with its `SET-SLOT` assignment. See [SitTargets](sittargets.html). |

## See also

- [Migration from AVsitter](migration.html): for converting an existing AVsitter prim.
- [AVpos Reference](avpos-reference.html): notecard syntax.
- [Boot Sequence](boot-sequence.html): what happens on first run.
- [Compatibility Matrix](compatibility-matrix.html): mixing QS and stock plugins.
