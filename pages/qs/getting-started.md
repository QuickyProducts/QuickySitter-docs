---
title: Getting Started
sidebar: home_sidebar
permalink: getting-started.html
keywords: install, setup, getting started, basic
toc: true
---

This page walks through getting QuickySitter scripts running in a Second Life or OpenSim furniture prim. It assumes basic familiarity with AVsitter 2 — if you've never used AVsitter, the [upstream AVsitter2 instructions](https://avsitter.github.io/avsitter2_home.html) cover the conceptual foundation (poses, menus, sitters, the AVpos notecard).

## What you need

- The QuickySitter scripts. The current source is at [github.com/QuickyProducts/QuickySitter](https://github.com/QuickyProducts/QuickySitter); a packaged in-world distribution is available on the SL Marketplace (link to be added once the QS Marketplace store is live).
- A Second Life viewer with Mono compilation. Firestorm and the official viewer both work — the only difference is how you transfer scripts (Firestorm saves Mono-by-default; other viewers need the Ctrl-drop workaround documented in the [stock import guide](https://github.com/QuickyProducts/QuickySitter/blob/master/avstock/IMPORT_GUIDE.md)).
- An empty `AVpos` notecard. The notecard format is identical to stock AVsitter — see [AVpos Reference](avpos-reference.html).

## Minimum script set

The smallest install that gets you a working QuickySitter prim is three scripts plus a notecard:

1. **`[QS]boot`** — the LSD seeder. Parses `AVpos` once per fresh boot and writes the persistent state.
2. **`[QS]sitA`** — main pose-and-sit script. One per sitter slot (rename as `[QS]sitA`, `[QS]sitA 2`, `[QS]sitA 3`, etc.).
3. **`[QS]sitB`** — menu-and-state companion to sitA. Must match the count of sitA scripts.
4. **an `AVpos` notecard** — boot's self-check ERRORs (red hovertext) if it is missing. boot is the sole writer of `qs:cfg`/`qs:p`/`qs:meta`/`qs:sitter`, so without the notecard sitA/sitB never leave their pre-boot state.

Everything else is optional and presence-gated — each plugin announces itself via a `qs:alive:<name>` LSD flag, and the feature degrades silently (or with a warn) when absent:

- **`[QS]select`** — sit-time seat-select picker for multi-sitter / multi-furniture setups. sitB has a built-in picker, so this is optional; sitB reads `qs:alive:select` (falling back to an `[AV]select` inventory probe for stock-AVsitter compat).
- **`[QS]adjuster`** — enables the `[HELPER]` menu and live `[SAVE]`-to-LSD writing. Without it, you can still play poses, but creators can't fine-tune positions in-world.
- **`[QS]prop`** — handles `PROP*` directives in the notecard and the dynamic-prop attach protocol used by HUD addons.
- **`[QS]faces`** — handles face/expression animations.
- **`[QS]offset`** — dedicated personal-offset storage. It owns the `CUSTOMS` store; without it there is **no** personal-offset persistence and no sitA fallback.
- **`[QS]sequence`** — animation sequences.

The stock AVsitter plugins (`[AV]camera`, `[AV]favs`, `[AV]helperscript`) work unchanged inside a QuickySitter linkset — see [Compatibility Matrix](compatibility-matrix.html). QS forks its own root family (`[QS]root`, `[QS]root-control`, `[QS]root-security`, `[QS]root-RLV`); there is no `[QS]camera` fork (stock `[AV]camera` is used as-is).

## Step-by-step

1. **Import the scripts.** Follow the same procedure as the [stock AVsitter import guide](https://github.com/QuickyProducts/QuickySitter/blob/master/avstock/IMPORT_GUIDE.md). The only difference is the script names: `[QS]sitA` instead of `[AV]sitA`, etc. Mono must be enabled when saving (Firestorm does this automatically).
2. **Prepare a host prim.** Any modifiable prim works. Rez a box.
3. **Create the AVpos notecard.** In your inventory, create a new notecard, name it exactly `AVpos`, open it and write a single line: `This notecard is empty` (any non-AVpos-command text is fine; even a single space works). **Save it.** A notecard that has never been saved after creation is corrupt and will cause boot to hang.
4. **Drop the scripts into the prim.** Drag all `[QS]*` scripts plus the `AVpos` notecard into the prim's contents.
5. **Verify boot.** On first run boot parses the notecard and writes `qs:meta:<ch>` per seeded channel; with one `[QS]sitA` and one `[QS]sitB` it writes `qs:meta:0` to LSD. If the `AVpos` notecard is missing, boot's self-check sets red hovertext and the prim stays unresponsive.
6. **Sit on the prim.** You should be sitting on the default sit-target and not animated (no poses defined yet). Touch the prim to bring up the menu — at this point you only see the AVsitter `[ADJUST]` and `[STOP HELP]` style entries.

## Adding your first pose

With `[QS]adjuster` installed, the menu has a `[HELPER]` entry. Pick `[HELPER]` → `[NEW]` and type a pose name in chat (e.g., `Sit casual`). The script plays the SL default `sit` animation and writes a stub `qs:p:0:0` LSD entry. Adjust the position using the helper-bar arrows; click `[SAVE]` to persist your changes.

For a richer authoring workflow, the [QuickyHUD addon](hud-integration.html) provides per-axis nudge buttons and a live offset preview.

## Editing AVpos directly

You can also edit the `AVpos` notecard by hand:

```
MTYPE 2

POSE Sit casual|sit
{Sit casual}<0.0, 0.0, 0.05><0.0, 0.0, 0.0>
```

Each directive lives on its own line, with `|` separating the arguments of a single directive. `POSE <name>|<animation>` declares the pose; `{<name>}<pos><rot>` sets the position and rotation for an already-declared pose. The position/rotation line is normally written by `[HELPER] [SAVE]` or `[DUMP]` — you usually don't hand-write it.

Save the notecard, and boot detects the asset-key change via `changed(CHANGED_INVENTORY)`, re-seeds LSD, and broadcasts the reload so sitB picks up the new pose list without a manual reset. See [AVpos Reference](avpos-reference.html) for the full directive list.

## Troubleshooting

| Symptom | Likely cause |
|---------|--------------|
| Prim is silent on sit | `[QS]sitA` or `[QS]sitB` missing → boot self-check should have shouted; check your inventory. |
| Red hovertext: "AVpos notecard missing" | No `AVpos` notecard in the prim. boot's self-check hard-fails — add a saved `AVpos` notecard and reset. |
| `llOwnerSay`: "[QS]sitA: missing [QS]sitB" | sitter-count mismatch — same number of sitA and sitB scripts required. |
| Boot hangs at "Parsing AVpos…" | Notecard was never saved after creation; open it, save it, reset the script. |
| `[PROP]` button missing despite `PROP*` lines in AVpos | `[QS]prop` not installed. Boot self-check warns about this case. |
| `[FACES]` / `[EXPRESSION]` button missing | `[QS]faces` not installed. |
| Adjuster sliders make no visible change | Helper-bar / sit-target conflict — check the sit-target via `[ADJUST] [SITTARGET]`. See [SitTargets](sittargets.html). |

## See also

- [Migration from AVsitter](migration.html) — for converting an existing AVsitter prim.
- [AVpos Reference](avpos-reference.html) — notecard syntax.
- [Boot Sequence](boot-sequence.html) — what happens on first run.
- [Compatibility Matrix](compatibility-matrix.html) — mixing QS and stock plugins.
