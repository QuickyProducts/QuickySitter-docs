---
title: QuickyHUD — User Manual
sidebar: home_sidebar
permalink: quickyhud-manual.html
keywords: quicky hud, pose hud, manual, help, adjustmode, sync
toc: true
---

QuickyHUD is a tool designed to simplify and speed up positioning on QuickySitter furniture. It allows creators and users to quickly fine-tune avatar positions directly while sitting, without navigating complex menus.

## HUD quick overview

The HUD provides quick access to the most important positioning functions for AVsitter adjustments. The control map below labels every button — the numbers match the list that follows.

<figure class="hud-map" style="max-width:640px;margin:1.5rem auto">
<svg width="100%" viewBox="0 0 760 430" role="img" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<title>QuickyHUD pose-mode control map</title>
<desc>The HUD in pose mode with numbered callouts pointing to each control.</desc>
<defs><marker id="hudarrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs>
<image href="images/QuickyHUD.png" xlink:href="images/QuickyHUD.png" x="118" y="24" width="525" height="383"/>
<g stroke="#5b6b7e" stroke-width="1.2" fill="none">
<line x1="52" y1="64" x2="156" y2="64" marker-end="url(#hudarrow)"/>
<line x1="52" y1="199" x2="156" y2="199" marker-end="url(#hudarrow)"/>
<line x1="52" y1="369" x2="156" y2="369" marker-end="url(#hudarrow)"/>
<line x1="262" y1="406" x2="316" y2="300" marker-end="url(#hudarrow)"/>
<line x1="258" y1="48" x2="312" y2="96" marker-end="url(#hudarrow)"/>
<line x1="460" y1="24" x2="503" y2="42" marker-end="url(#hudarrow)"/>
<line x1="600" y1="24" x2="560" y2="42" marker-end="url(#hudarrow)"/>
<line x1="703" y1="119" x2="523" y2="119" marker-end="url(#hudarrow)"/>
<line x1="703" y1="216" x2="545" y2="216" marker-end="url(#hudarrow)"/>
<line x1="703" y1="300" x2="528" y2="300" marker-end="url(#hudarrow)"/>
</g>
<g font-family="Arial,Helvetica,sans-serif" font-size="13" text-anchor="middle">
<circle cx="401" cy="182" r="11" fill="#1f80c0"/><text x="401" y="182" fill="#fff" dominant-baseline="central">1</text>
<circle cx="40" cy="64" r="12" fill="#1f80c0"/><text x="40" y="64" fill="#fff" dominant-baseline="central">3</text>
<circle cx="40" cy="199" r="12" fill="#1f80c0"/><text x="40" y="199" fill="#fff" dominant-baseline="central">6</text>
<circle cx="40" cy="369" r="12" fill="#1f80c0"/><text x="40" y="369" fill="#fff" dominant-baseline="central">4</text>
<circle cx="250" cy="415" r="12" fill="#1f80c0"/><text x="250" y="415" fill="#fff" dominant-baseline="central">2</text>
<circle cx="250" cy="40" r="12" fill="#1f80c0"/><text x="250" y="40" fill="#fff" dominant-baseline="central">11</text>
<circle cx="455" cy="14" r="12" fill="#1f80c0"/><text x="455" y="14" fill="#fff" dominant-baseline="central">9</text>
<circle cx="610" cy="14" r="12" fill="#1f80c0"/><text x="610" y="14" fill="#fff" dominant-baseline="central">10</text>
<circle cx="715" cy="119" r="12" fill="#1f80c0"/><text x="715" y="119" fill="#fff" dominant-baseline="central">7</text>
<circle cx="715" cy="216" r="12" fill="#1f80c0"/><text x="715" y="216" fill="#fff" dominant-baseline="central">5</text>
<circle cx="715" cy="300" r="12" fill="#1f80c0"/><text x="715" y="300" fill="#fff" dominant-baseline="central">8</text>
</g>
</svg>
<figcaption>QuickyHUD in pose mode</figcaption>
</figure>

1. **Menu** — opens the furniture's pose menu (the glowing bolt in the center).
2. **Select sitter** — choose whom you adjust: normally only sitters in your own pose group; in ADJUSTMODE you can select any sitter.
3. **Sync** — realigns drifted animations.
4. **Swap seat** — switch seat: direct with two seats, via a picker with more.
5. **Move X / Y** — shift forward/back and left/right in three step sizes (0.01 / 0.05 / 0.1 m).
6. **Move up / down (Z)** — raise and lower height in the same three steps.
7. **Position ↔ rotation** — switch the arrows to rotation control (RX/RY/RZ) and back.
8. **Settings** — opens RESET (any sitter), plus ADJUSTMODE, AUTOSYNC and CLEAR (owner only).
9. **HUD size +/−** — scale the on-screen HUD larger or smaller.
10. **Minimize** — collapse the HUD to the bolt icon; tap it again to restore.
11. **Help** — links to this manual, the Marketplace page and the support group.

> **Personal vs. permanent:** Move and rotate (5–7, including Z) normally adjust your *personal offset* — saved per avatar and per pose, and restored the next time you sit. In **ADJUSTMODE** they instead edit the pose's *default values* (the AVsitter pose data, not offsets) — the way a creator re-adjusts the base poses for everyone.

**Inside the Settings menu (8):** RESET (reset the selected target to its default pose) is available to any sitter; the ADJUSTMODE toggle, AUTOSYNC (off / 60 / 120 / 180 s) and CLEAR offset storage are owner only. The texture/design picker sets the HUD look.

**Automatic / background:** offsets are saved and restored per avatar and pose, the HUD attaches on sit (or from a menu button in menu mode), and with RLV active the hover height is set to 0 on attach and restored on detach.

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

## See also

- [Re-Sync Protocol](resync-protocol.html) — the LinkMsg 90271 trigger SYNC uses.
- [HUD Integration](hud-integration.html) — the in-prim hudproxy/hudadmin contract.
- [Personal Pose Offsets](personal-pose-offsets.html) — how offsets are stored and persisted.
- [Adjustment Workflow](adjustment-workflow.html) — the creator-side `[HELPER]` workflow ADJUSTMODE replaces for live tuning.
