---
title: QuickySitter Pro Manual
sidebar: home_sidebar
permalink: quickyhud-creator-manual.html
keywords: quickysitter pro, creator, toolset, quicky hud, configuration, hudconfig, design, adjustmode, attach mode, hud offset, verbose
toc: true
---

**QuickySitter Pro** is the creator toolset: the plugins and tools that make building and selling QuickySitter furniture easier. The HUD is one of those plugins, the Animesh Adjust Dummies are another. This manual covers the creator side: getting Pro into a piece and configuring how it behaves. For everyday use of the HUD while sitting, see the [User Manual](quickyhud-manual.html). For setting up couples / group poses without a second avatar, see [Animesh Adjust Dummies](quickyhud-animesh.html).

> **Note:** This page is the *creator* side: preparing and configuring furniture you sell. QuickySitter Pro is built for the **QuickySitter engine**, which is what you get the most out of it on. The HUD's in-prim components (`[QS]hudproxy` / `[QS]hudadmin`) are deliberately kept able to run on a stock AVsitter linkset too; only the SYNC / Re-Sync features are structurally bound to `[QS]sitA` and need the QuickySitter engine.

## Setting up a piece

From your QuickySitter Pro kit you only handle two things: the **installer** (a script object you drop into the furniture) and the **Quicky Updater HUD** (the in-world HUD you wear and click).

### Converting an existing AVsitter piece

Take a finished AVsitter piece and convert it to Quicky in place. Drop the **creator installer** in and click your **Quicky Updater HUD**. It migrates the piece and clears out the old parts. (The same path also repairs or updates an older Quicky piece.)

The payoff goes beyond the QuickyHUD adjustment workflow: conversion moves the pose data out of script memory into Linkset Data, lifting the piece past AVsitter's **stack-heap collision** ceiling, the Mono memory limit that makes large pose sets (roughly 1,000+ poses) crash on stock AVsitter. A converted piece stays slim no matter how many poses you add. See [Known Limits](known-limits.html) for the detail.

### A brand-new piece

1. Drop the **creator installer** into the furniture's root prim.
2. It asks how many **seats** the piece has, so pick the number.
3. Click your **Quicky Updater HUD**.

The complete pose system is installed for you and the installer removes itself. The piece is now Quicky-enabled. Add your poses with an **AVpos** notecard as usual.

### Before you sell: script permissions

Before you sell a finished piece, set the Quicky scripts inside to **copy-only** for the next owner: the buyer can copy and use the furniture, but the scripts can't be modified or transferred out.

### Installing, updating, repairing

All of it is the one-shot installer's job. `[QS]installWithLicense` covers a fresh build, an AVsitter-to-Quicky migration, a repair, and updating an already-installed piece. The flow is always the same: drop it into the prim that carries the script base, it runs an **HTTP license check**, the Quicky Updater HUD pushes the current scripts, and the installer **removes itself** when done.

## Configuration

This is where you, as the creator, decide how the HUD looks and behaves.

### The `hudconfig` notecard

Drop a notecard named **`hudconfig`** into the furniture. Its **first line** holds four pipe-separated fields:

```
RESERVE|ATTACHMODE|TEXTURE|HUDOFFSET
```

For example: default reserve, auto-attach, the built-in design, and a small HUD screen offset:

```
0|auto||<0.45, -0.2, 0>
```

(The empty third field, the `||`, keeps the built-in design.)

| Field | Default | What it does |
|-------|---------|--------------|
| `RESERVE` | `0` | The system already keeps a sensible memory reserve by default. Only raise this above `0` if you know the piece needs more free space. |
| `ATTACHMODE` | `auto` | `auto` = the HUD attaches by itself when someone sits (via the AVsitter Experience). `menu` = no auto-attach; the user attaches it from a menu / button instead. |
| `TEXTURE` | *(empty)* | The default HUD design, as a texture UUID. Leave empty to keep the built-in design. |
| `HUDOFFSET` | `<0,0,0>` | Where the HUD sits on screen when attached (see below). |

No `hudconfig` notecard, or a blank first line, leaves everything at its default. The config has to be on the very first line; comment lines above it aren't supported.

**Attaching by hand (`ATTACHMODE = menu`).** With auto-attach off, give users a way to attach the HUD by hand: add a button to your **AVpos** notecard:

```
BUTTON Quicky HUD|90510|Quicky-HUD
```

The button label (`Quicky HUD`) is yours to change; the number **`90510`** and the **`Quicky-HUD`** parameter are what trigger the attach, so leave those exactly as shown.

### HUD screen position (`HUDOFFSET`)

Second Life resets a HUD's position every time it attaches, so the HUD re-applies your `HUDOFFSET` on each attach. The value in `hudconfig` is the default you ship; a user can nudge it afterwards and their own choice is remembered.

### HUD design / texture

Set the shipped design in the `TEXTURE` field. Users can also switch designs live: tap the design button for a built-in one, or drop a texture UUID for a custom design. The chosen design sticks across re-attach. (See the [User Manual](quickyhud-manual.html).)

### ADJUSTMODE: authoring poses live

ADJUSTMODE is your working mode while building. With it on, every move and rotate you make with the HUD writes **straight into the pose data** instead of being stored as a personal offset. Tune your poses, then use the **`[DUMP]`** function to write the result into a fresh AVpos notecard. Toggle ADJUSTMODE (with a confirmation) from the HUD's Settings menu; it stays on until you turn it off. See [User Manual → ADJUSTMODE for Creators](quickyhud-manual.html#adjustmode-for-creators-owner-only) for the full workflow.

On a piece in **menu mode** (no auto-attach on sit), turning ADJUSTMODE on also makes sure you have a HUD to drive it: the in-prim hudproxy fires `90274 ATTACH_FOR_ADJUST` to hudadmin, which attaches a HUD to the seated operator. So you can enter ADJUSTMODE on a menu-mode piece without first attaching the HUD by hand.

You can also enter ADJUSTMODE without opening the HUD's own Settings menu, using the dedicated **`[QUICKYHUD]`** button in the furniture's `[ADJUST]` submenu (driven by `[QS]adjuster`). It appears when `[QS]adjuster` is present (`qs:alive:adjuster`), the `QPP_CFG:ADJUSTMODE` key exists, the build is licensed, and the clicker passes the **Adjust ACL**. Clicking it flips ADJUSTMODE on through the same path (and, in menu mode, attaches a HUD for the operator). There is no separate *Quicky HUD* option inside `[HELPER]`.

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

QuickySitter keeps the AVsitter 2 plugin protocol, so protocol-driven stock **AVsitter plugins work unchanged** alongside the Quicky scripts, and QS ships its own variants where the stock plugin would mis-detect the engine. The ones you'd add yourself:

| Plugin | Adds |
|--------|------|
| **Camera** (`[AV]camera`) | A custom camera view per pose. |
| **Locks & adult** (`[AV]LockMeister`, `[AV]LockGuard`, `[AV]Xcite!`) | Lock and adult interaction: stock, work unchanged. |
| **RLV & access** (`[QS]root-RLV`, `[QS]root-control`, `[QS]root-security`) | RLV restraints and sit/menu access: use the QS forks (see below). |
| **Favourites** (`[AV]favs`) | Sitters can save and recall favourite poses. |
| **Expressions** (`[QS]faces`) | Facial expressions per pose. |
| **Sequences** (`[QS]sequence`) | Auto-advancing pose sequences. |
| **Helper** (`[AV]helperscript`) | The classic stock pose-adjust helper: QuickySitter's HUD + ADJUSTMODE already cover this, so you rarely need it. |

QuickySitter ships its own take on some of these (expressions, sequences, props, RLV, the seat picker) with extra integration: where a Quicky version is included, use that.

For **camera, favourites and the lock/adult plugins** the stock version works either way. **Expressions, sequences, props and RLV need the Quicky versions:** stock `[AV]faces` / `[AV]sequence` / `[AV]prop` / `[AV]root-RLV` find the engine by probing `[AV]sitA` script names that a QS linkset doesn't have, so they mis-read the sitter count (RLV drops to a single seat on multi-sitter pieces). Props have a second reason: the HUD's auto-attach is wired to `[QS]prop` (it listens for the `90280 QSPROP_ATTACH` hook that `[QS]prop` publishes). A prop dropped in as `[AV]prop` would rez, but the HUD won't auto-attach to its sitter. If you take `[QS]root-RLV`, run `[QS]root-control` and `[QS]root-security` with it: the control-suite scripts address each other by name, so don't mix `[QS]` and `[AV]` there.

Your delivery package also contains **`[QS]objectadjust`**: drop it into a prop next to the stock `[AV]object` and the prop becomes resizable and body-fittable with `[SAVE]` persistence, see [prop scale & worn fit](plugin-prop.html#prop-scale-and-worn-fit-qsobjectadjust).

Full plugin-by-plugin detail: [Compatibility Matrix](compatibility-matrix.html).

## See also

- [User Manual](quickyhud-manual.html): using the HUD while sitting.
