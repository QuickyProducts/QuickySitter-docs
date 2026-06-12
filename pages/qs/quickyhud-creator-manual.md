---
title: QuickyHUD QuickySitter — Creator Manual
sidebar: home_sidebar
permalink: quickyhud-creator-manual.html
keywords: quicky hud, creator, configuration, hudconfig, design, adjustmode, attach mode, hud offset, verbose
toc: true
---

This manual is for **creators** who build and sell QuickySitter furniture — setting up QuickyHUD in a piece and configuring how it looks and behaves. For everyday use of the HUD while sitting, see the [User Manual](quickyhud-manual.html).

> **Note:** This page is the *creator* side — preparing and configuring furniture you sell. What you build here is a **product combination of QuickyHUD and QuickySitter**: this creator edition of QuickyHUD is **no longer compatible with stock AVsitter** — it requires the QuickySitter engine as its base.

## Setting up a piece

From your Quicky creator kit you only handle two things: the **installer** (a script object you drop into the furniture) and the **Quicky Updater HUD** (the in-world HUD you wear and click).

### A brand-new piece

1. Drop the **creator installer** into the furniture's root prim.
2. It asks how many **seats** the piece has — pick the number.
3. Click your **Quicky Updater HUD**.

The complete pose system is installed for you and the installer removes itself. The piece is now Quicky-enabled — add your poses with an **AVpos** notecard as usual.

### Converting an existing AVsitter or older Quicky piece

Drop the **creator installer** in and click your **Quicky Updater HUD** — it converts the piece in place and clears out the old parts.

### Before you sell: script permissions

Before you sell a finished piece, set the Quicky scripts inside to **copy-only** for the next owner — the buyer can copy and use the furniture, but the scripts can't be modified or transferred out.

### Installing vs. updating

The two are separate jobs handled by different scripts:

- **Installing / converting / repairing** is the one-shot installer's job. `[QS]install` (customer) and `[QS]installWithLicense` (creator) do a fresh build, an AVsitter-to-Quicky migration, or a repair, then **remove themselves**. The `WithLicense` variant runs an **HTTP license check** before it proceeds; the plain `[QS]install` does not.
- **Routine updates** don't go through the installer at all. They're driven by the resident **`[QS]hudadmin`** script that stays in the piece — the Quicky Updater HUD's region-wide push talks to it, so you don't drop anything into the prim to update an already-installed piece.

## Configuration

This is where you, as the creator, decide how the HUD looks and behaves.

### The `hudconfig` notecard

Drop a notecard named **`hudconfig`** into the furniture. Its **first line** holds four pipe-separated fields:

```
RESERVE|ATTACHMODE|TEXTURE|HUDOFFSET
```

For example — default reserve, auto-attach, the built-in design, and a small HUD screen offset:

```
0|auto||<0.45, -0.2, 0>
```

(The empty third field — the `||` — keeps the built-in design.)

| Field | Default | What it does |
|-------|---------|--------------|
| `RESERVE` | `0` | The system already keeps a sensible memory reserve by default. Only raise this above `0` if you know the piece needs more free space. |
| `ATTACHMODE` | `auto` | `auto` = the HUD attaches by itself when someone sits (via the AVsitter Experience). `menu` = no auto-attach; the user attaches it from a menu / button instead. |
| `TEXTURE` | *(empty)* | The default HUD design, as a texture UUID. Leave empty to keep the built-in design. |
| `HUDOFFSET` | `<0,0,0>` | Where the HUD sits on screen when attached (see below). |

No `hudconfig` notecard — or a blank first line — leaves everything at its default. The config has to be on the very first line; comment lines above it aren't supported.

**Attaching by hand (`ATTACHMODE = menu`).** With auto-attach off, give users a way to attach the HUD by hand — add a button to your **AVpos** notecard:

```
BUTTON Quicky HUD|90510|Quicky-HUD
```

The button label (`Quicky HUD`) is yours to change; the number **`90510`** and the **`Quicky-HUD`** parameter are what trigger the attach, so leave those exactly as shown.

### HUD screen position (`HUDOFFSET`)

Second Life resets a HUD's position every time it attaches, so the HUD re-applies your `HUDOFFSET` on each attach. The value in `hudconfig` is the default you ship; a user can nudge it afterwards and their own choice is remembered.

### HUD design / texture

Set the shipped design in the `TEXTURE` field. Users can also switch designs live — tap the design button for a built-in one, or drop a texture UUID for a custom design. The chosen design sticks across re-attach. (See [User Manual → Quicky Design HUD](quickyhud-manual.html#quicky-design-hud).)

### ADJUSTMODE — authoring poses live

ADJUSTMODE is your working mode while building. With it on, every move and rotate you make with the HUD writes **straight into the pose data** instead of being stored as a personal offset. Tune your poses, then use the **`[DUMP]`** function to write the result into a fresh AVpos notecard. Toggle ADJUSTMODE (with a confirmation) from the HUD's Settings menu; it stays on until you turn it off. See [User Manual → ADJUSTMODE for Creators](quickyhud-manual.html#adjustmode-for-creators-owner-only) for the full workflow.

On a piece in **menu mode** (no auto-attach on sit), turning ADJUSTMODE on also makes sure you have a HUD to drive it: the in-prim hudproxy fires `90274 ATTACH_FOR_ADJUST` to hudadmin, which attaches a HUD to the seated operator. So you can enter ADJUSTMODE on a menu-mode piece without first attaching the HUD by hand.

You can also enter ADJUSTMODE from QuickySitter's own in-world adjust tool, **`[QS]adjuster`** — the `[HELPER]` bar in the pose menu (shown when `[QS]adjuster` is present, gated on the `qs:alive:adjuster` flag). Picking its *Quicky HUD* option flips ADJUSTMODE on through the same path.

### Diagnostics (`VERBOSE`)

For troubleshooting, add a `VERBOSE n` line to the AVpos notecard:

| Level | Output |
|-------|--------|
| `0` | errors / warnings only (default) |
| `1` | + startup banner |
| `2` | + runtime status |
| `3` | + full debug detail |

Leave it at `0` for anything you ship.

## Adding plugins

QuickySitter keeps the AVsitter 2 plugin protocol, so protocol-driven stock **AVsitter plugins work unchanged** alongside the Quicky scripts — and QS ships its own variants where the stock plugin would mis-detect the engine. The ones you'd add yourself:

| Plugin | Adds |
|--------|------|
| **Camera** (`[AV]camera`) | A custom camera view per pose. |
| **Control / RLV** (`[AV]control` — LockMeister, LockGuard, Xcite!, RLV) | RLV restraints and lock / adult interaction. |
| **Favourites** (`[AV]favs`) | Sitters can save and recall favourite poses. |
| **Expressions** (`[QS]faces`) | Facial expressions per pose. |
| **Sequences** (`[QS]sequence`) | Auto-advancing pose sequences. |
| **Helper** (`[AV]helperscript`) | The classic stock pose-adjust helper — QuickySitter's HUD + ADJUSTMODE already cover this, so you rarely need it. |

QuickySitter ships its own take on some of these (expressions, sequences, props, the seat picker) with extra integration — where a Quicky version is included, use that.

For **camera, control and favourites** the stock plugin works either way. **Expressions, sequences and props need the Quicky versions:** stock `[AV]faces` / `[AV]sequence` / `[AV]prop` detect the engine by probing `[AV]sitA` script names, which a QS linkset doesn't have. Props have a second reason: the HUD's auto-attach is wired to `[QS]prop` (it listens for the `90280 QSPROP_ATTACH` hook that `[QS]prop` publishes) — a prop dropped in as `[AV]prop` would rez, but the HUD won't auto-attach to its sitter.

Full plugin-by-plugin detail: [Compatibility Matrix](compatibility-matrix.html).

## See also

- [User Manual](quickyhud-manual.html) — using the HUD while sitting.
