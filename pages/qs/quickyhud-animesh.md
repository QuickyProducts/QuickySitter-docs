---
title: QuickyHUD Animesh Adjust Dummies
sidebar: home_sidebar
permalink: quickyhud-animesh.html
keywords: animesh, dummy, adjust, partner, couples, group poses, solo setup, creator
toc: true
---

Set up and adjust **couples and group poses without a second avatar**: the Animesh plugin rezzes a posable dummy onto any empty seat of the current pose. The dummy plays that seat's animation and sits at its pose position. You adjust it through the QuickyHUD exactly like a real partner, and the result saves through the normal adjust workflow.

> **Note:** This is a **creator tool**, delivered with the QuickyHUD/QuickySitter creator bundle only. It is license-gated: on a piece without a valid creator license the plugin stays idle and the `[ANIMESH]` entry never appears. Finished customer furniture never receives it, and `[FINALIZE]` strips it off a piece before you sell.

## Quick start

1. Sit on the furniture and select the couples / group pose you want to work on.
2. Open `[ADJUST]` → `[ANIMESH]`. The seat list shows the **empty seats of the current pose**.
3. Pick a seat, then pick a body from the numbered list, and the dummy rezzes on that seat and starts the partner animation.
4. Enter ADJUSTMODE (the `[QUICKYHUD]` button in the seat list does it in one click and attaches the HUD) and `SELECT` the dummy in the HUD picker. It behaves like any sitter target.
5. Adjust position/rotation; saving works as usual (`[SAVE]` / `[DUMP]` into the AVpos notecard).

Repeat per seat: one dummy per empty seat, several at once for group poses. `[OFF ALL]` removes every dummy in one click; standing up cleans them up automatically.

## Reaching the menu while adjusting

The `[ADJUST]` menu is not reachable while ADJUSTMODE or helper mode is active, so the plugin offers three shortcuts:

- **Click a dummy**: opens its body picker directly (change or `[REMOVE]` the body). Use **right-click → Touch**: a plain left-click often misses animesh objects (their click target is a static bounding box).
- **`[QUICKYHUD]`** in the seat list: flips ADJUSTMODE on and attaches the HUD for you.
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

Dummies always yield to people:

- Someone **sits down** or **swaps** onto a dummy's seat → the dummy is removed with a short chat note, and the avatar lands on the seat normally.
- Swapping an avatar **with** a dummy (trading places) is not supported, so the dummy simply yields.
- The operator standing up removes all dummies.

A **Re-Sync** restarts the dummies together with the real sitters, so SYNC poses stay in phase.

## [FINALIZE]

`[FINALIZE]` in the seat list (owner-only, with a confirm dialog) strips the complete animesh kit off the piece: all `[QS]dummy` bodies, the `animeshconfig` notecard and both plugin scripts. Run it on a finished piece **before you sell it**, and the customer copy then carries no trace of the tool.

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
