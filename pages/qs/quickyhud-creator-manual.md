---
title: Quicky Pose HUD — Creator Manual
sidebar: home_sidebar
permalink: quickyhud-creator-manual.html
keywords: quicky hud, creator, installation, configuration, hudconfig, license, updater, adjustmode, attachmode, hudoffset
toc: true
---

This manual is for **creators** who build and sell QuickySitter / AVsitter furniture — installing the Quicky Pose HUD system into a build and configuring it. For everyday use of the HUD while sitting, see the [User Manual](quickyhud-manual.html).

> **Note:** This page covers *creator* installation — building and licensing furniture. Getting the HUD running for an end user on a finished product is a separate process and is not described here.

## Creator vs. customer builds

Every installer ships in two flavours, distinguished by an internal `LICENSE_SALT`:

| Build | Installer | `LICENSE_SALT` | Use |
|-------|-----------|----------------|-----|
| **Creator** | `[QS]installWithLicense` | `20220818` | Building and iterating on your own pieces. Runs an HTTP licence check and writes a licence token into the furniture before self-deleting. |
| **Customer** | `[QS]install` | `0` | Migration drop-in handed to buyers; no licence check. |

A Creator build refuses to be overwritten by a Customer build (and vice-versa), so a creator HUD never accidentally downgrades a piece that has already been sold.

## The toolchain

- **Quicky Updater HUD** (`[QS]updater`) — the in-world object you wear and click. It holds the script/object payload and pushes it into furniture in your region over an owner-scoped channel.
- **Drop-in installers** — `[QS]installWithLicense` (creator) / `[QS]install` (customer). You drop one into a prim; it opens the script pin, receives the push, and self-deletes.
- **`[QS]hudadmin`** — the permanent resident that stays in finished furniture. It handles every later refresh, so once a piece is built you never need an installer again.

## Installation

### Before you start

- Your Updater HUD's inventory must contain **everything it may push** — in particular all ten sitA copies: `[QS]sitA`, `[QS]sitA 1`, … `[QS]sitA 9` (byte-identical, only the inventory name differs). `llRemoteLoadScriptPin` pushes *by name*, so a name that is not in the HUD cannot be delivered.
- Set the Updater HUD's **object description** to identify the release — see [Updater release metadata](#updater-release-metadata) under Configuration.
- Make sure no payload item is **modify for next owner** — see [Before you ship](#before-you-ship-strip-mod).

### Building new furniture from scratch

1. Drop **`[QS]installWithLicense`** into the empty root prim.
2. It detects that no sitter scripts are present and opens a dialog — pick the number of **sitter slots** (1–10) the piece will have.
3. Click the **Quicky Updater HUD**.
4. The base stack is installed: `[QS]sitA` (one script per slot), `[QS]sitB`, `[QS]adjuster`, `[QS]hudadmin`, `[QS]boot`, `[QS]offset`, `[QS]prop`, and the `QuickyHUD` object (which carries `hudproxy` / `hudcontrol` / `hudmatrix` inside it).
5. The installer runs its licence check and self-deletes.
6. **Drop optional extras by hand** for non-default variants:
   - `[QS]root-RLV` instead of `[QS]root` for an RLV piece.
   - `[QS]faces`, `[QS]sequence`, `[QS]select`, `[QS]debug` — added on demand, not part of the base.

For **multi-prim** furniture (extra sit-prims), drop a fresh `[QS]installWithLicense` into each extra prim, pick "1", then remove everything but `[QS]sitB` from that prim by hand.

### Migrating existing AVsitter or legacy furniture

1. Drop `[QS]installWithLicense` (creator) or `[QS]install` (customer) into the prim that hosts `[AV]sitA`.
2. Click the Quicky Updater HUD.
3. The installer reports the `[AV]*` plugins it found; the HUD pushes the matching `[QS]` forks plus the runtime stack and strips legacy Quicky names.
4. The installer self-deletes; `[QS]hudadmin` runs its one-time data migrations on first start.

### Repairing an incomplete piece

If a finished piece is missing a base item or a sitter slot — e.g. you deleted `[QS]offset` while freeing memory during debugging — drop `[QS]installWithLicense` back in. It detects the gap, enters **repair mode**, and on the next Updater click pushes **only the missing items**. Surviving local copies are left untouched.

### Refreshing already-installed furniture

No installer needed. `[QS]hudadmin` is permanent and answers the Updater directly — the HUD pushes its inventory 1:1.

> **Note:** Creator builds are protected by an update-targeting gate: a Creator HUD only updates pieces that have `[QS]installWithLicense` present as your explicit "I'm working on this one" marker, so a push does not hit every Quicky object sitting in your sandbox.

### Before you ship: strip mod

The Updater **refuses to push** while any payload item is *modify for next owner* — otherwise your script source would ship readable. Set every payload script and object to no-modify for next owner; the block clears once the perms are clean.

## Configuration

### The `hudconfig` notecard

`[QS]hudadmin` reads a notecard named **`hudconfig`** from the furniture. The **first line (line 0)** is the data line, pipe-delimited:

```
RESERVE|ATTACHMODE|TEXTURE|HUDOFFSET
```

| Field | Default | Meaning |
|-------|---------|---------|
| `RESERVE` | `0` | Extra Linkset Data bytes to reserve for this piece (stored as `QPP_CFG:RESERVE`). |
| `ATTACHMODE` | `auto` | `auto` = the HUD attaches automatically via the AVsitter Experience when someone sits (requires the Experience to be enabled). `menu` = no auto-attach; the user attaches it from a menu / button instead. |
| `TEXTURE` | *(empty)* | Default HUD design — a texture UUID. Empty keeps the stored / built-in design. |
| `HUDOFFSET` | `<0,0,0>` | Screen-space position of the HUD when attached (see below). |

No `hudconfig` notecard → all defaults. A line 0 starting with `#` also means "keep defaults". Legacy 3-field notecards (`RESERVE|ATTACHMODE|TEXTURE`) are still accepted; `HUDOFFSET` then defaults to `<0,0,0>`.

> **Note:** The notecard was renamed from `Quicky-config` to `hudconfig` in the V3 layout (hudadmin 1.1917+) — a hard rename with no fallback, so older builds need the notecard renamed to pick up config again.

### HUD screen position (`HUDOFFSET`)

Second Life resets a HUD's position every time it attaches, so the HUD **re-applies** `HUDOFFSET` on each attach. The value in `hudconfig` is the **creator default**; an owner can override it at runtime (stored in Linkset Data as `QPP_CFG:HUDOFFSET`). Resolution order at attach time is: runtime override → `hudconfig` default → `<0,0,0>`.

### HUD design / texture

The `TEXTURE` field sets the default design. Users can also switch designs live — tap the design button to pick a built-in, or drop a texture UUID via the texture changer. The chosen texture persists across rez and re-attach. (See [User Manual → Quicky Design HUD](quickyhud-manual.html#quicky-design-hud).)

### ADJUSTMODE — live pose authoring

ADJUSTMODE is the creator working mode: while it is on, every position and rotation change you make with the HUD is written **directly into AVsitter's pose data** instead of being stored as a per-avatar offset. Adjust your poses, then use AVsitter's `[DUMP]` to write the result into a fresh AVpos notecard. It is toggled (with a confirmation) from the HUD's Settings menu and persists in Linkset Data (`QPP_CFG:ADJUSTMODE`). See [User Manual → ADJUSTMODE for Creators](quickyhud-manual.html#adjustmode-for-creators-owner-only) for the full workflow.

### Diagnostics (`VERBOSE`)

The HUD scripts follow the project-wide verbose ladder:

| Level | Output |
|-------|--------|
| `0` | errors / warnings only (default) |
| `1` | + boot banner |
| `2` | + runtime status |
| `3` | + debug detail |

Set `VERBOSE n` in the AVpos notecard; `[QS]boot` writes it to `qs:cfg:verbose` and `hudproxy` / `hudadmin` pick it up. Leave it at `0` for shipped products.

### Updater release metadata

The **object description** of the Updater HUD identifies the release it ships. Format (all fields after the first are optional, `#`-separated):

```
<hudversion>#<productID>#<sitterversion>
```

| Field | Example | Purpose |
|-------|---------|---------|
| `hudversion` | `1.30` | The wire version. Receivers only accept a **newer** version, so an old HUD cannot downgrade a freshly installed piece. |
| `productID` | `22950355` | Used for the HTTP version check. The Creator product ID also marks the push as a **Creator** build (suffix `c`), so it will not overwrite a Customer build. |
| `sitterversion` | `0.93` | Informational — shown in the pre-push owner message so you know which sitter cycle this HUD pairs with. |

For test rigs a bare `1.30` is fine; `#productID` only matters once you distribute through a vendor with HTTP redelivery and the creator / customer split live.

## See also

- [User Manual](quickyhud-manual.html) — using the HUD while sitting.
- [HUD Integration](hud-integration.html) — the in-prim `hudproxy` / `hudadmin` contract.
- [LSD Storage](lsd-storage.html) and [LSD Keys](lsd-keys.html) — where config and offsets are stored.
