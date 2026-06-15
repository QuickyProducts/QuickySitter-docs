---
title: Adjustment Workflow ([HELPER])
sidebar: home_sidebar
permalink: adjustment-workflow.html
keywords: helper, adjuster, save, adjust, workflow, pose
toc: true
---

This page describes the `[HELPER]` workflow — the in-world authoring loop for creating and adjusting poses. The mechanism is similar to stock AVsitter 2; QS adds LSD-backed persistence so your `[SAVE]` clicks survive script resets.

## When to use what

- **Free repositioning while sitting** (no save, just visual): touch → `[ADJUST]` → use the up/down arrows. Position changes are session-only.
- **Save a new default for the current pose**: `[HELPER]` → `[SAVE]`. Pose default in LSD is updated.
- **Save a personal offset for yourself, this pose**: `[SAVE]` (gated on `[QS]offset` presence via the `qs:offset:alive` flag). Goes to `[QS]offset` as `QSO:<short>:<slot>:<pose>`. There is no separate `[ADJUSTER]` button.
- **Save a personal offset for yourself, all poses**: `[SAVE ALL]`. Writes the magic `M#T!` entry — fallback when no pose-specific offset is set.
- **Add a new pose entry**: `[HELPER]` → `[NEW]` → type name → adjust → `[SAVE]`.

## The `[HELPER]` menu

`[HELPER]` is exposed only when `[QS]adjuster` is present (sitB reads the `qs:alive:adjuster` flag) and the user has creator-level permission. The menu has the following entries (exact set depends on context — sit state, hudproxy presence, etc.):

| Entry | What it does |
|-------|--------------|
| `[NEW]` | Create a new pose entry with the next sequential name. You're prompted for the pose name in chat. |
| `[SAVE]` | Write the current `CURRENT_POSITION`/`CURRENT_ROTATION` back to `qs:p:<ch>:<i>` as the new pose default. Also emits 90263 to clear stale customs. |
| `[DUMP]` | Trigger boot's dump cascade. Output goes both to chat and to the AVsitter settings service URL. |
| `[CANCEL]` | Exit helper without saving the pending edit. |
| `[QUICKYHUD]` | Toggle QuickyHUD ADJUSTMODE. Sends 90266 `"On"` to hudproxy. Visible only when hudproxy is present. |
| `[SITTARGET]` | Enter sit-target adjustment mode (move the sit-target itself, not the pose offset). |
| `[DONE]` | Leave helper mode and return to the normal pose menu. |

## QS-specific: live LSD persistence

In stock AVsitter, `[SAVE]` updates the pose default in `[AV]sitB`'s memory. If the script is reset or the object re-rezzed without first running `[DUMP]` and copy-pasting back into the notecard, the change is lost.

In QuickySitter, `[SAVE]`:

1. Writes the new pose default to `qs:p:<ch>:<i>` (LSD).
2. Emits 90263 to `[QS]offset` to drop any pose-specific customs on this slot for any user. `M#T!` (all-poses fallback) is preserved.
3. Emits 90301 to `[QS]sitB` so the seated avatar re-renders with the new default immediately.

Result: your edit survives script reset, object rerez, region restart — as long as `qs:boot:asset` still matches the notecard's current asset-key. Re-saving the AVpos notecard reverts to its text (boot detects the new asset-key, re-seeds LSD). To capture in-world edits back into the notecard format, use `[DUMP]`.

## `[DUMP]` — captured changes back to the notecard format

`[DUMP]` reads everything in `qs:cfg:*`, `qs:sitter:*`, `qs:p:*` and formats it back into AVpos directive syntax. It outputs to two places:

1. **Local chat.** Streamed in chunks so it doesn't blow the chat throughput limit.
2. **HTTP upload.** Posted to the AVsitter settings service; boot llShouts the URL to the owner. Useful for large configs that would otherwise overflow chat.

`[QS]adjuster`'s contribution to this whole pipeline is one line — it sends 90098 to boot to kick the cascade. Boot owns the rest. See [Boot Sequence](boot-sequence.html).

## QuickyHUD-driven adjustment

When QuickyHUD is in the linkset (hudproxy + hudadmin), the user gets a wearable HUD with:

- **X+/Y+/Z+/X-/Y-/Z-** for nudge increments.
- **Save / Reset** buttons.
- A live offset display.

The HUD writes per-user offsets via 90262 directly to `[QS]offset`, with no menu round-trip. This is much faster than the dialog-driven approach for fine-tuning. See [HUD Integration](hud-integration.html).

When the HUD's ADJUSTMODE is `On`, the regular sitB pose menu gains a `[DONE]` button (alongside `[NEW]`/`[DUMP]`/`[SAVE]`), since the user is now in HUD-driven adjustment. Click `[DONE]` to leave HUD adjustment — it switches ADJUSTMODE off and opens the dialog adjust submenu.

## Why `[HELPER] [SAVE]` triggers 90263

In stock AVsitter, `[SAVE]` only updates the pose default in memory. The seated avatar is NOT repositioned live, so stale pose-specific `CUSTOMS` entries never get a chance to re-apply on top of the new default.

QS's sitB 90301 handler deliberately calls `send_anim_info(FALSE)` so the seated avatar reflects the new default immediately. But that routes through `apply_current_anim` in sitA, which adds `MY_CUSTOMS[pose_name]` on top of the new default. Result: visible "snap" by the old offset vector.

90263 is sent **before** 90301, so sitA processes the customs eviction ahead of the 90055 chain that re-applies the pose. The seated avatar lands on the helper-bar position; future re-sits start from the new default with no carry-over offset. `M#T!` (the all-poses personal offset) is preserved by design — it isn't tied to the saved pose name.

## See also

- [Personal Pose Offsets](personal-pose-offsets.html) — `[QS]offset`'s read/write model.
- [HUD Integration](hud-integration.html) — for the wearable-HUD alternative.
- [Boot Sequence](boot-sequence.html) — what `[DUMP]` does internally.
- [AVpos Reference](avpos-reference.html) — the on-disk format `[DUMP]` reconstructs.
