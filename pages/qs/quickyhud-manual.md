---
title: QuickyHUD AVsitter — User Manual
sidebar: home_sidebar
permalink: quickyhud-manual.html
keywords: quicky hud, pose hud, manual, help, adjustmode, sync
toc: true
---

QuickyHUD AVsitter is a tool designed to simplify and speed up positioning on furniture that uses AVsitter 2 (and QuickySitter). It allows creators and users to quickly fine-tune avatar positions directly while sitting, without navigating complex menus.

## Key Features

### Fast Adjustment

The most commonly used adjustment steps are available directly on the control pad:

- **0.01** — small step
- **0.05** — middle step
- **0.1** — large step

These steps can be used immediately via the directional controls.

### Camera-Aware Movement

The HUD evaluates the camera position to determine movement direction. This ensures that positional adjustments always feel logical and intuitive, regardless of the viewing angle.

**Example:** Moving "left" on the control pad will move the avatar left relative to your current camera view, not the world axis.

### Automatic HUD Attachment (Experience Enabled)

On furniture set up for **auto-attach** (the default mode), the HUD uses the AVsitter Experience to attach automatically. When an avatar sits on such a piece:

- The HUD is automatically attached to the sitter.
- There is no need to manually search for or attach the HUD from the inventory.

Some pieces are instead configured for **menu mode**, where the HUD does not attach on its own. On those, attach it yourself by pressing the **Quicky HUD** button the creator added to the menu.

### One-Time Installation

The automatic attachment requires a one-time installation. Please follow the steps described in the Installation Manual to enable the Experience and prepare the furniture.

After the installation is completed, the HUD will attach and detach automatically whenever someone sits or stands up — on pieces left in the default auto-attach mode. (A creator can switch a piece to menu mode, where the HUD is attached from a button instead.)

Future updates work without inventory drop. You just attach the Quicky Updater HUD and click it — it broadcasts the update region-wide to every matching Quicky piece you own in the region. The update is owner-gated (only your own furniture answers) and version-gated (a piece is only touched if the HUD carries a newer version).

### Quicky Design HUD

The HUD texture can be changed without editing the script. Two ways to set the texture:

- Tap the design button on the HUD to choose one of the included designs.
- Drop a texture UUID into the HUD via the texture changer to apply a custom design.

The selected texture is stored in the HUD and is reapplied automatically after rez or attach.

### Animation Re-Sync

When several avatars share a multi-person pose, their looped animations can drift out of phase — for example, when one person sits down later than the others. The SYNC button on the HUD realigns all running animations on the furniture to a shared beat, so the cycle starts at the same moment for everyone.

**Usage:**

- Tap the SYNC button on the control pad for a one-shot manual re-sync.
- For long sessions, the owner can enable AUTOSYNC from the Settings menu: pick an interval (60 s, 120 s, or 180 s) and the HUD will re-sync automatically until you turn it off.

**Notes:**

- Re-Sync only affects looped animations on the furniture you are currently sitting on.
- It is safe to press at any time; it never interrupts or restarts a pose, only realigns the timing.
- AUTOSYNC settings persist across HUD detach/reattach.

See also: [Re-Sync Protocol](resync-protocol.html) for the technical detail behind the SYNC button.

### ADJUSTMODE for Creators (Owner Only)

ADJUSTMODE is a working mode for furniture creators. While ADJUSTMODE is active, every position and rotation change made with the HUD is written directly into AVsitter's pose data instead of being stored as a per-avatar offset.

**Workflow:**

1. Enable ADJUSTMODE from the Settings menu (a confirmation dialog appears).
2. Sit on the furniture and adjust poses with the HUD as usual.
3. When finished, use AVsitter's standard `[DUMP]` function to write the new values into a fresh AVpos notecard.
4. Disable ADJUSTMODE again from the Settings menu.

**Notes:**

- Changes apply to all sitters and are permanent. They can only be reverted by a script reset.
- ADJUSTMODE replaces the classic AVsitter helper objects for live pose tuning.

See also: [HUD Integration](hud-integration.html) and [Adjustment Workflow](adjustment-workflow.html).

### Storage You Can See and Control (Owner Only)

Saved pose offsets are stored in the furniture's Linkset Data. Open the Settings menu to access two storage functions:

- The CLEAR confirmation dialog shows a storage report with the estimated remaining capacity (number of poses that can still be saved).
- The `CLEAR offset storage` button deletes all saved offsets for all avatars and all poses in one step. A confirmation dialog prevents accidental clearing.

When storage runs full, the oldest unused entry is automatically removed to make room for new ones.

See also: [Personal Pose Offsets](personal-pose-offsets.html) for the technical detail.

## HUD Layout

The HUD provides quick access to the most important positioning functions for AVsitter adjustments. Each section of the HUD is designed for a specific category of action:

- **Directional control pad** — the X/Y/Z nudge buttons with step-size selection.
- **Action buttons** — SYNC, MENU, SWAP (seat-swap picker), SELECT (sitter picker — choose whom you adjust), SETTINGS, HELP.
- **Settings menu** — RESET (reset your SELECT target to its default position), ADJUSTMODE toggle, AUTOSYNC interval, CLEAR offset storage, texture/design.

## See also

- [Re-Sync Protocol](resync-protocol.html) — the LinkMsg 90271 trigger SYNC uses.
- [HUD Integration](hud-integration.html) — the in-prim hudproxy/hudadmin contract.
- [Personal Pose Offsets](personal-pose-offsets.html) — how offsets are stored and persisted.
- [Adjustment Workflow](adjustment-workflow.html) — the creator-side `[HELPER]` workflow ADJUSTMODE replaces for live tuning.
