---
title: Adjustment Workflow
sidebar: home_sidebar
permalink: adjustment-workflow.html
keywords: quickyhud, adjustmode, helper, adjuster, save, dump, workflow, pose
toc: true
---

There are two ways to author and fine-tune poses in-world: the **QuickyHUD-driven path** (the modern workflow, where the wearable HUD *is* your adjuster and a quiet `[DUMP]` hands you the result) and the classic **`[HELPER]` dialog**, the stock-AVsitter loop that is still here, lightly updated. Both write through the same LSD pose store, so you can move between them freely.

## QuickyHUD-driven adjustment (ADJUSTMODE)

The headline QS authoring path: adjust with the wearable QuickyHUD instead of the dialog helper bars.

**Entering it.** The operator clicks `[QUICKYHUD]` in the adjust menu (or flips ADJUSTMODE from the HUD's own settings). `[QS]adjuster` sends 90266 `"On"` to hudproxy (sitB only broadcasts the button click on 90100), and entering ADJUSTMODE **auto-attaches a HUD to the seated operator** (`ATTACH_FOR_ADJUST` 90274 → hudadmin), so there is no rummaging in inventory. The pose menu gains a `[DONE]` exit button.

**Adjusting.** Nudge position and rotation with the HUD's camera-relative X/Y/Z buttons at the selected step size. While **ADJUSTMODE is `On`**, every change is written straight into the **pose default** (the value *all* sitters get) instead of a per-avatar offset. The new default is persisted to LSD (`qs:p:<ch>:<i>`) immediately, so it survives script reset, rerez and region restart; only re-seeding from a changed AVpos notecard overwrites it. (With ADJUSTMODE `Off`, the same HUD nudges save a *personal* offset for the wearer only, via 90262 → `[QS]offset`.)

**Capturing: the quiet `[DUMP]`.** When the poses look right, `[DUMP]` from the ADJUSTMODE menu runs **quiet**: boot rebuilds the AVpos directives, posts them to the settings-service URL, and shouts only an upfront `[DUMP] Live view: <url>` line and a final done/failed line. The `COPY ABOVE/BELOW` banners are **suppressed from chat** and go only into the uploaded web content, so there is no per-line chat spam. The URL is the deliverable; open it and paste its contents over your `AVpos` notecard. (The classic `[HELPER]` `[DUMP]` stays loud, with full stock-style chat, for users without the HUD.)

**Leaving it.** `[DONE]` ends ADJUSTMODE (sitB broadcasts 90100 `[DONE]`; `[QS]adjuster` does the tear-down, sending 90266 `"Off"` to hudproxy) and drops you into the dialog adjust submenu.

See [HUD Integration](hud-integration.html) for the in-prim contract and [Personal Pose Offsets](personal-pose-offsets.html) for the offset store.

## The classic `[HELPER]` dialog (still available)

The stock-AVsitter authoring loop is still here and unchanged in feel: `[HELPER]` exposes `[NEW]`/`[SAVE]`/`[DUMP]`, and you adjust with the dialog arrows. It's exposed only when `[QS]adjuster` is present (sitB reads `qs:alive:adjuster`), the `[AV]helper` object is in the prim's inventory, and the clicker passes the **Adjust ACL**. Since 1.25 that ACL is owner-only by default but can be widened to `GROUP` or `ALL` in `[QS]root-security`'s `[SECURITY]` menu (published as `qs:sec:adjust`); the owner always passes.

What QS changed under the hood is small:

- **`[SAVE]` persists.** Stock `[SAVE]` only updated sitB's memory, so it was lost on reset unless you `[DUMP]`ed and pasted back. QS writes the new default to LSD (`qs:p:<ch>:<i>`), so it survives script reset, rerez and region restart (as long as the notecard's asset-key is unchanged). To capture LSD edits back into the notecard, use `[DUMP]`.
- **Live re-render + customs eviction.** `[SAVE]` emits 90263 (drop stale pose-specific customs) then 90301 (re-render the seated avatar on the new default); see the 90263 note below.
- **`[QUICKYHUD]`** in this menu is the entry point into the ADJUSTMODE path above.

### `[HELPER]` menu entries

| Entry | What it does |
|-------|--------------|
| `[NEW]` | Create a new entry. Opens a type picker (`[POSE]`/`[SYNC]`/`[SUBMENU]`/...); for a pose you pick the animation(s), confirm with `[DONE]`, then type the name in a text box. Names are not auto-generated sequentially. |
| `[SAVE]` | Write `CURRENT_POSITION`/`CURRENT_ROTATION` to `qs:p:<ch>:<i>` as the new pose default. Also emits 90263 to clear stale customs. |
| `[DUMP]` | Trigger boot's dump cascade. From this path it runs **loud** (full chat output) and uploads. |
| `[QUICKYHUD]` | Toggle QuickyHUD ADJUSTMODE (90266 `"On"`). Visible only when hudproxy is present. |
| `[DONE]` | Leave helper mode and return to the normal pose menu. |

## When to use which save

- **Reposition while sitting, no save:** touch → `[ADJUST]` → arrows. Session-only.
- **New default for this pose (creator):** ADJUSTMODE + HUD, or `[HELPER]` → `[SAVE]`. Writes the LSD pose default for everyone.
- **Personal offset, just you, this pose:** HUD nudge with ADJUSTMODE `Off`, or `[SAVE]` (gated on `[QS]offset` via `qs:offset:alive`) → `QSO:<short>:<slot>:<pose>`.
- **Personal offset, just you, all poses:** `[OFFSET ALL]` (the sitter-dialog button) → the magic `M#T!` fallback entry. (`[SAVE ALL]` is the HUD-side label for the same thing.)
- **Add a new pose:** `[NEW]` (from the `[HELPER]` or ADJUSTMODE menu) → pick the type and animation → `[DONE]` → type a name → adjust → save.

## `[DUMP]`: captured changes back to notecard format

`[DUMP]` reads everything in `qs:cfg:*`, `qs:sitter:*`, `qs:p:*` and rebuilds it into AVpos directive syntax. `[QS]adjuster`'s contribution is one line: it sends 90098 to boot to kick the cascade (with a `"quiet"` marker on the ADJUSTMODE path); boot owns the rest. Output depends on the path:

- **Quiet (ADJUSTMODE path):** only the settings-service URL (an upfront `[DUMP] Live view: <url>` line) and a final done/failed line are shouted to the owner. The `COPY ABOVE/BELOW` banners are suppressed from chat and appear only in the uploaded content, because the URL is the deliverable.
- **Loud (`[HELPER]` path):** streamed to local chat in chunks **and** uploaded; boot shouts the URL at the end. Stock-style, for users without the HUD.

See [Boot Sequence](boot-sequence.html).

## Why `[HELPER] [SAVE]` triggers 90263

In stock AVsitter, `[SAVE]` only updates the pose default in memory. The seated avatar is NOT repositioned live, so a stale pose-specific personal offset (measured against the *old* default) never gets dropped and would re-apply on top of the new default the next time you sit.

QS's sitB 90301 handler repositions the seated avatar on the new default immediately, but it does so by forwarding the new pos/rot **straight from the 90301 payload** to sitA via 90055. It deliberately does **not** re-read the value through `send_anim_info()`: that would race with adjuster's `qs_save_pose_offset` LSD write and (worse) key off `ANIM_INDEX`, which can point at a stale slot or be `-1`, snapping the avatar back to the previous position. Note that sitA has no in-RAM `MY_CUSTOMS` map: personal offsets live in the `QSO:*` LSD tier plus a volatile `RAM_OVERFLOW` mirror.

90263 is sent **before** 90301, so sitA drops the now-stale pose-specific offset ahead of the 90055 chain that re-applies the pose. The seated avatar lands on the helper-bar position; future re-sits start from the new default with no carry-over offset. `M#T!` (the all-poses personal offset) is preserved by design, because it isn't tied to the saved pose name.

## See also

- [Personal Pose Offsets](personal-pose-offsets.html): `[QS]offset`'s read/write model.
- [HUD Integration](hud-integration.html): the in-prim QuickyHUD contract.
- [Boot Sequence](boot-sequence.html): what `[DUMP]` does internally.
- [AVpos Reference](avpos-reference.html): the on-disk format `[DUMP]` reconstructs.
