---
title: QuickySitter Pro Animesh Adjust Dummies
sidebar: home_sidebar
permalink: quickysitter-pro-animesh.html
redirect_from:
  - /quickyhud-animesh.html
keywords: animesh, dummy, adjust, partner, couples, group poses, solo setup, creator, remote authoring, standing
toc: true
---

Set up and adjust **couples and group poses without a second avatar**: the Animesh plugin rezzes a posable dummy onto any empty seat of the current pose. The dummy plays that seat's animation and sits at its pose position. You adjust it through the QuickyHUD exactly like a real partner, and the result saves through the normal adjust workflow. You can do all of it seated, or stand next to the piece and build it from outside (see [Remote authoring](#remote-authoring-building-without-sitting-down-128)).

> **Note:** This is a **creator tool**, delivered with QuickySitter Pro (Creator Edition) only, never with the Personal Edition. It is license-gated: on a piece without a valid creator license the plugin stays idle and the `[ANIMESH]` entry never appears. Finished customer furniture never receives it, and `[FINALIZE]` strips it off a piece before you sell.

## Quick start

1. Sit on the furniture and select the couples / group pose you want to work on.
2. Open `[ADJUST]` → `[ANIMESH]`. The seat list shows the **empty seats of the current pose**.
3. Pick a seat, then pick a body from the numbered list, and the dummy rezzes on that seat and starts the partner animation.
4. Enter ADJUSTMODE (the `[HELPER HUD]` button in the seat list does it in one click and attaches the HUD) and `SELECT` the dummy in the HUD picker. It behaves like any sitter target.
5. Adjust position/rotation; saving works as usual (`[SAVE]` / `[DUMP]` into the AVpos notecard).

Repeat per seat: one dummy per empty seat, several at once for group poses. `[OFF ALL]` removes every dummy in one click; standing up cleans them up automatically.

## Remote authoring: building without sitting down (1.28)

Everything above assumes you are sitting on the piece. You do not have to be. Stand within **5 m** of the furniture and type:

```
/5 animesh
```

The piece answers with its own pose menu and the session begins. From there the work is the same, with one difference that matters: you can walk around the piece and look at the scene from outside while you build it.

1. Pick the pose you want to build. The menu stays open, so you can change your mind.
2. `[ADJUST]` leads to the session's own page: `[ANIMESH]`, `[HELPER HUD]`, `[DUMP]` and `[DONE]`.
3. `[ANIMESH]` opens the seat list, exactly as when seated. Fill the seats with dummies.
4. Drive them with the HUD: the arrows move, `SELECT` picks which dummy they move, `MENU` returns to the pose menu, `RESET` puts one back. Every change is saved as the pose default straight away, so there is no separate save step.
5. `[DUMP]` rebuilds the `AVpos` once you are happy, and the link arrives in chat.
6. `[DONE]` ends the session and clears the dummies.

`[HELPER HUD]` hands you a HUD if you are not wearing one and takes it back when the session ends. The session also ends by itself if you sit down or leave the region, so nothing is left running behind you.

### Sitting or standing

The two modes divide the work rather than replacing each other:

- **Sit** to create a pose and to judge how it feels on a real body, which is the one thing a dummy cannot tell you.
- **Stand** to build and position a couple or a group, and to see the result the way a visitor will.

Switching is free. Sitting down during a session hands you straight to the seated tools; standing up and typing `/5 animesh` brings you back, and your dummies are still where you left them.

### Who may open a session

Remote authoring follows the **Adjust** access level in the `[SECURITY]` menu, the same setting that governs the seated adjust tools. On the default, `OWNER`, only you can open a session. Set it to `GROUP` or `ALL` and the people you have opened the piece to can build on it too, which is what you want when you build from a personal account while a store account owns the furniture.

On a piece without the security plugin the chat shortcut does the same job:

```
/5 adjust group
```

The level falls back to `OWNER` automatically when a piece changes hands, so a customer never inherits an opened setting.

## Reaching the menu while adjusting

The `[ADJUST]` menu is not reachable while ADJUSTMODE or helper mode is active, so the plugin offers three shortcuts:

- **Click a dummy**: opens its body picker directly (change or `[REMOVE]` the body). Use **right-click → Touch**: a plain left-click often misses animesh objects (their click target is a static bounding box).
- **`[HELPER HUD]`** in the seat list: flips ADJUSTMODE on and attaches the HUD for you.
- **Auto-offer**: entering ADJUSTMODE alone on a multi-seat piece with no dummies out opens the seat list by itself after a moment.

## The dummy bodies

Bodies are discovered **by name**: every object in the furniture inventory whose name starts with `[QS]dummy` appears in the body picker. The label is the rest of the name; the picker sorts in inventory (alphabetical) order.

The standard set (heights are mesh-measured):

| Body | Height |
|---|---|
| Female S / M / L | 168 cm / 187 cm / 224 cm |
| Male S / M / L | 175 cm / 194 cm / 233 cm |
| xKabuki Female / Male | 182 cm / 223 cm |

(The `x` in the Kabuki names is deliberate: it sorts them after the sized set.)

**Adding your own body**, two ways:

- Name it `[QS]dummyYourLabel` and drop it into the furniture, and it is discovered automatically. The label must not contain a `|` character.
- Keep its own name and declare it in the **`animeshconfig`** notecard: `ANIMESH <object name> | <description>` (one per line, up to 10 entries, description up to 48 characters). The notecard is only for your own extras, and it is never overwritten by updates.

Updates refresh the standard `[QS]dummy` set automatically; bodies you added yourself are left untouched.

**Requirements per body:** copy-enabled, **modify for the next owner**, the **"Animated Mesh"** feature flag set, and the `[QS]animeshReceiver` script inside. Pose animations do **not** need to be pre-loaded: if the body lacks one, the furniture hands over its own copy on demand, which is why the furniture's pose animations must be **copy**.

## Around real avatars

By default, dummies yield to people:

- Someone **sits down** or **swaps** onto a dummy's seat → the dummy is removed with a short chat note, and the avatar lands on the seat normally.
- Swapping an avatar **with** a dummy (trading places) is not supported, so the dummy simply yields.
- The operator standing up removes all dummies.

A **Re-Sync** restarts the dummies together with the real sitters, so SYNC poses stay in phase.

### Showcase mode is gone (1.28)

Earlier versions carried a `[SHOWCASE]` toggle that kept staged dummies on their seats through stand-up, for photos and vendor displays. It existed to work around the fact that building a display meant sitting on it and then getting off. Remote authoring builds the same display without anyone sitting down in the first place, so the toggle has been removed and the yield-to-people behaviour above always applies. Set a display piece up with `/5 animesh` and finalize it as usual.

## [FINALIZE]

`[FINALIZE]` in the seat list (owner-only, with a confirm dialog) strips the complete animesh kit off the piece: all `[QS]dummy` bodies, the `animeshconfig` notecard and all three plugin scripts. Run it on a finished piece **before you sell it**, and the customer copy then carries no trace of the tool.

## Limits

- Dummies play **body animation only**: no facial (Bento face) animation; the dummy heads carry no face rigging.
- Rezzing needs free **land impact**; on a full parcel the plugin reports the failed rez and skips that dummy.
- Left-click selection often misses animesh, so use right-click → Touch.

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| No `[ANIMESH]` entry in `[ADJUST]` | Plugin not installed on this piece, or no valid creator license (the plugin idles), or you are not seated. |
| Body rezzes but does not animate | The body's **"Animated Mesh"** flag is missing (silent no-op), the pose animation is not **copy**, or the body is **no-modify** so the animation hand-over is rejected. Watch local chat for the exact message. |
| A seat is missing from the list | The seat is occupied, or the current pose has no entry for that slot in the AVpos notecard. |
| Dummy sits at the wrong spot | The slot's `{pose}` position line is missing or unadjusted in AVpos. Adjust the dummy and save. |
| Boot line says `N bodies (D discovered + C config)` | Informational: how many bodies the picker can offer (discovered by name + notecard entries), not how many are rezzed. |
| `/5 animesh` does nothing | You are further than 5 m from the piece, the plugin is not on it, or the Adjust access level does not include you (see [Who may open a session](#who-may-open-a-session)). Typing it again while a session runs simply reopens the menu. |
| The HUD does not move a standing dummy | Press `SELECT` and pick the dummy first: with nobody seated there is no default target to fall back on. |
