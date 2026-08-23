---
title: QuickySitter Pro Manual
sidebar: home_sidebar
permalink: quickysitter-pro-manual.html
redirect_from:
  - /quickyhud-creator-manual.html
keywords: quickysitter pro, creator, toolset, quicky hud, configuration, hudconfig, design, adjustmode, attach mode, hud offset, verbose
toc: true
---

**QuickySitter Pro (Creator Edition)** is the creator toolset: the plugins and tools that make building and selling QuickySitter furniture easier. The HUD is one of those plugins, the Animesh Adjust Dummies are another. This manual covers the creator side: getting the kit into a piece and configuring how it behaves. For everyday use of the HUD while sitting, see the [HUD Manual](quickyhud-manual.html). For setting up couples / group poses without a second avatar, see [Animesh Adjust Dummies](quickysitter-pro-animesh.html).

> **Which product is which?** From release 1.27 there are three names. **QuickySitter** is the sitter engine on its own. **QuickySitter Pro (Creator Edition)** is this kit, the one you build and sell furniture with. **QuickySitter Pro (Personal Edition)** is the same system for your own furniture, without the right to build products for sale. Both editions were previously sold together as "QuickyHUD + QuickySitter". The HUD itself keeps its name, QuickyHUD.

> **Note:** This page is the *creator* side: preparing and configuring furniture you sell. The Creator Edition is built for the **QuickySitter engine**, which is what you get the most out of it on. The HUD's in-prim components (`[QS]hudproxy` / `[QS]hudadmin`) are deliberately kept able to run on a stock AVsitter linkset too; only the SYNC / Re-Sync features are structurally bound to `[QS]sitA` and need the QuickySitter engine.

## Setting up a piece

From your Creator Edition kit you only handle two things: the **installer** (a script object you drop into the furniture) and the **Quicky Updater HUD** (the in-world HUD you wear and click).

### Converting an existing AVsitter piece

Take a finished AVsitter piece and convert it to Quicky in place. Drop the **creator installer** in and click your **Quicky Updater HUD**. It migrates the piece and clears out the old parts. (The same path also repairs or updates an older Quicky piece.)

The payoff goes beyond the QuickyHUD adjustment workflow: conversion moves the pose data out of script memory into Linkset Data, lifting the piece past AVsitter's **stack-heap collision** ceiling, the Mono memory limit that makes large pose sets (roughly 1,000+ poses) crash on stock AVsitter. A converted piece stays slim no matter how many poses you add. See [Known Limits](known-limits.html) for the detail.

### A brand-new piece

1. Drop the **creator installer** into the furniture's root prim.
2. It asks how many **seats** the piece has, so pick the number.
3. Click your **Quicky Updater HUD**.

The complete pose system is installed for you and the installer removes itself. The piece is now Quicky-enabled. Add your poses with an **AVpos** notecard as usual.

### Before you sell: finalizing a piece

A piece you have been building still carries the creator tools. Those are licensed to you, not to your customers, so take them out before you hand the piece over. Do it while you still own the piece: it is one command, and it cannot be undone from inside the furniture.

There are two ways, and which one you want depends on whether your customers should keep the `[HELPER]` menu.

| | Removes | Leaves |
|---|---|---|
| `/5 cleanup` typed in chat | the animesh toolkit **and** the `[HELPER]` menu, the helper bars object and `[QS]adjuster` | the running furniture: poses, menus, the HUD |
| `[FINALIZE]` in the `[ANIMESH]` menu | the animesh toolkit: the dummy bodies, the `animeshconfig` notecard and both animesh scripts | everything else, `[HELPER]` menu included |

Both are owner-only. Keeping the `[HELPER]` menu in a piece you sell is perfectly fine, it belongs to the open-source sitter engine, and some creators deliberately leave it in so their customers can fine-tune poses themselves. The animesh toolkit is the part that must not travel.

`/5 cleanup` also takes the Adjust access level out of the `[SECURITY]` menu, because with the adjust tools gone there is no adjust workflow left to gate.

### Before you sell: script permissions

The Quicky scripts in a finished piece must be **copy-only** for the next owner: copy on, modify off, transfer off. The buyer can copy and use the furniture, but the scripts cannot be modified or taken out and passed on. This is a requirement of the creator license agreement, not a recommendation.

Finalizing checks this for you. Both paths above look at `[QS]hudproxy` and `[QS]hudadmin` and tell you in chat if either of them is set differently, naming the script and the permission:

```
⚠ Delivery check: [QS]hudproxy and [QS]hudadmin must ship COPY ONLY
(next owner: copy on, modify off, transfer off). The licence agreement
allows nothing else.
[QS]hudadmin: TRANSFER is on, the buyer can pass it on
Fix this in Edit > Contents before you hand the piece out.
Nothing was changed here.
```

The check reports, it does not act: no permission is changed and no script is removed, so you can correct it in **Edit > Contents** and finalize again, or simply fix it and deliver.

Two things it cannot see. `hudmatrix` and `hudcontrol` live inside the QuickyHUD object itself, and no script can read another object's contents, so check those by hand. And a piece with no HUD scripts in it produces no message at all, since there is nothing for the check to look at.

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
| `ATTACHMODE` | `menuplus` | `menuplus` *(default)* = no auto-attach; the HUD adds its own `💠 POSE HUD` entry to the furniture's `[ADJUST]` menu, so sitters take it when they want it and you need to do nothing. `auto` = the HUD attaches by itself when someone sits (needs the AVsitter Experience; without it every sit raises a permission prompt). `menu` = no auto-attach and no entry either; you place the button yourself (see below). |
| `TEXTURE` | *(empty)* | The default HUD design, as a texture UUID. Leave empty to keep the built-in design. |
| `HUDOFFSET` | `<0,0,0>` | Where the HUD sits on screen when attached (see below). |

No `hudconfig` notecard, or a blank first line, leaves everything at its default. The config has to be on the very first line; comment lines above it aren't supported.

**No auto-attach: two ways to give users the button.**

With `ATTACHMODE = menuplus`, the default, there is nothing to do. The HUD registers a `💠 POSE HUD` entry in the furniture's `[ADJUST]` menu by itself, and removes it again if you later switch the mode. Sitters get a HUD when they ask for one, which is why this is the default: attaching one to everybody was a constraint from the HUD's AVsitter-plugin days, not a decision.

With `ATTACHMODE = menu` you place the button yourself. Add it to your **AVpos** notecard:

```
BUTTON Quicky HUD|90510|Quicky-HUD
```

The button label (the part before the first `|`) is yours to change. What triggers the HUD is the number **`90510`** together with the parameter after it, and that parameter has to read exactly **`Quicky-HUD`**.

Pressing the button while the user already wears the HUD takes it off again, so one button covers both directions. The same is true of the `menuplus` entry.

{% include note.html content="Both modes suppress auto-attach, so `menu` and `menuplus` differ only in who places the button. Existing furniture on `menu` keeps behaving exactly as before, including after a script update." %}

### HUD screen position (`HUDOFFSET`)

Second Life resets a HUD's position every time it attaches, so the HUD re-applies your `HUDOFFSET` on each attach. The value in `hudconfig` is the default you ship; a user can nudge it afterwards and their own choice is remembered.

### HUD design / texture

Set the shipped design in the `TEXTURE` field. Users can also switch designs live: tap the design button for a built-in one, or drop a texture UUID for a custom design. The chosen design sticks across re-attach. (See the [HUD Manual](quickyhud-manual.html).)

### ADJUSTMODE: authoring poses live

ADJUSTMODE is your working mode while building. With it on, every move and rotate you make with the HUD writes **straight into the pose data** instead of being stored as a personal offset. Tune your poses, then use the **`[DUMP]`** function to write the result into a fresh AVpos notecard. Enter it with the **`[HELPER HUD]`** button in the furniture's `[ADJUST]` menu, leave it with `[DONE]` or `[ADJUST OFF]` in the pose menu; standing up ends it too. See [HUD Manual → ADJUSTMODE for Creators](quickyhud-manual.html#adjustmode-for-creators) for the full workflow.

On a piece in **menu mode** (no auto-attach on sit), turning ADJUSTMODE on also makes sure you have a HUD to drive it: the in-prim hudproxy fires `90274 ATTACH_FOR_ADJUST` to hudadmin, which attaches a HUD to the seated operator. So you can enter ADJUSTMODE on a menu-mode piece without first attaching the HUD by hand.

The **`[HELPER HUD]`** button in the furniture's `[ADJUST]` submenu (driven by `[QS]adjuster`) is the only way in since 1.26. It appears when `[QS]adjuster` is present (`qs:alive:adjuster`), the `QPP_CFG:ADJUSTMODE` key exists, the build is licensed, and the clicker passes the **Adjust ACL**. In menu mode it also attaches a HUD for the operator. There is no separate *Quicky HUD* option inside `[HELPER]`, and the toggle that used to live in the HUD's Settings menu is gone.

Note what that means for a finalized piece: `/5 cleanup` removes `[QS]adjuster`, so the button disappears with it and ADJUSTMODE can no longer be entered at all. That is the point of the presence-based model, and it is why personal pose offsets are deliberately kept working without the adjuster.

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
| **Pose pad** (`[QS]huddialog`) | Turns the `[POSE]` entry in `[ADJUST]` into the HUD's arrow controls as a menu: camera-relative directions, three step sizes, a Move and a Rotate page, every press saved right away. Each sitter gets their own pad. Part of the Quicky HUD set, so installs and updates deliver it with the rest; delete it from a piece and `[POSE]` goes back to the classic position dialog. |

QuickySitter ships its own take on some of these (expressions, sequences, props, RLV, the seat picker) with extra integration: where a Quicky version is included, use that.

For **camera, favourites and the lock/adult plugins** the stock version works either way. **Expressions, sequences, props and RLV need the Quicky versions:** stock `[AV]faces` / `[AV]sequence` / `[AV]prop` / `[AV]root-RLV` find the engine by probing `[AV]sitA` script names that a QS linkset doesn't have, so they mis-read the sitter count (RLV drops to a single seat on multi-sitter pieces). Props have a second reason: the HUD's auto-attach is wired to `[QS]prop` (it listens for the `90280 QSPROP_ATTACH` hook that `[QS]prop` publishes). A prop dropped in as `[AV]prop` would rez, but the HUD won't auto-attach to its sitter. If you take `[QS]root-RLV`, run `[QS]root-control` and `[QS]root-security` with it: the control-suite scripts address each other by name, so don't mix `[QS]` and `[AV]` there.

Your delivery package also contains **`[QS]objectadjust`**: drop it into a prop next to the stock `[AV]object` and the prop becomes resizable and body-fittable with `[SAVE]` persistence, see [prop scale & worn fit](plugin-prop.html#prop-scale-and-worn-fit-qsobjectadjust).

Full plugin-by-plugin detail: [Compatibility Matrix](compatibility-matrix.html).

## See also

- [HUD Manual](quickyhud-manual.html): using the HUD while sitting.
