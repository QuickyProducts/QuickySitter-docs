---
title: Adjustment Workflow
sidebar: home_sidebar
permalink: adjustment-workflow.html
keywords: quickyhud, adjustmode, helper, adjuster, save, dump, workflow, pose
toc: true
---

There are two ways to author and fine-tune poses in-world: the **QuickyHUD-driven path** — the modern workflow, where the wearable HUD *is* your adjuster and a quiet `[DUMP]` hands you the result — and the classic **`[HELPER]` dialog**, the stock-AVsitter loop that is still here, lightly updated. Both write through the same LSD pose store, so you can move between them freely.

## QuickyHUD-driven adjustment (ADJUSTMODE)

The headline QS authoring path: adjust with the wearable QuickyHUD instead of the dialog helper bars.

**Entering it.** The owner clicks `[QUICKYHUD]` in the adjust menu (or flips ADJUSTMODE from the HUD's own settings). sitB sends 90266 `"On"` to hudproxy, and entering ADJUSTMODE **auto-attaches a HUD to the seated operator** (`ATTACH_FOR_ADJUST` 90274 → hudadmin) — no rummaging in inventory. The pose menu gains a `[DONE]` exit button.

**Adjusting.** Nudge position and rotation with the HUD's camera-relative X/Y/Z buttons at the selected step size. While **ADJUSTMODE is `On`**, every change is written straight into the **pose default** — the value *all* sitters get — instead of a per-avatar offset. Edits are immediate, apply to everyone, and are permanent until a script reset or a fresh notecard. (With ADJUSTMODE `Off`, the same HUD nudges save a *personal* offset for the wearer only, via 90262 → `[QS]offset`.)

**Capturing — the quiet `[DUMP]`.** When the poses look right, `[DUMP]` from the ADJUSTMODE menu runs **quiet**: boot rebuilds the AVpos directives, posts them to the settings-service URL, and shouts only that URL plus the `COPY ABOVE/BELOW` markers — **no per-line chat spam**. The URL is the deliverable; open it and paste its contents over your `AVpos` notecard. (The classic `[HELPER]` `[DUMP]` stays loud — full stock-style chat — for users without the HUD.)

**Leaving it.** `[DONE]` ends ADJUSTMODE (sitB broadcasts 90100 `[DONE]`; `[QS]adjuster` does the tear-down, sending 90266 `"Off"` to hudproxy) and drops you into the dialog adjust submenu.

See [HUD Integration](hud-integration.html) for the in-prim contract and [Personal Pose Offsets](personal-pose-offsets.html) for the offset store.

## The classic `[HELPER]` dialog (still available)

The stock-AVsitter authoring loop is still here and unchanged in feel: `[HELPER]` exposes `[NEW]`/`[SAVE]`/`[DUMP]`/`[SITTARGET]`, and you adjust with the dialog arrows. It's exposed only when `[QS]adjuster` is present (sitB reads `qs:alive:adjuster`) and the clicker has creator-level permission.

What QS changed under the hood is small:

- **`[SAVE]` persists.** Stock `[SAVE]` only updated sitB's memory — lost on reset unless you `[DUMP]`ed and pasted back. QS writes the new default to LSD (`qs:p:<ch>:<i>`), so it survives script reset, rerez and region restart (as long as the notecard's asset-key is unchanged). To capture LSD edits back into the notecard, use `[DUMP]`.
- **Live re-render + customs eviction.** `[SAVE]` emits 90263 (drop stale pose-specific customs) then 90301 (re-render the seated avatar on the new default) — see the 90263 note below.
- **`[QUICKYHUD]`** in this menu is the entry point into the ADJUSTMODE path above.

### `[HELPER]` menu entries

| Entry | What it does |
|-------|--------------|
| `[NEW]` | Create a new pose entry with the next sequential name. You're prompted for the name in chat. |
| `[SAVE]` | Write `CURRENT_POSITION`/`CURRENT_ROTATION` to `qs:p:<ch>:<i>` as the new pose default. Also emits 90263 to clear stale customs. |
| `[DUMP]` | Trigger boot's dump cascade. From this path it runs **loud** (full chat output) and uploads. |
| `[SITTARGET]` | Adjust the sit-target itself, not the pose offset. |
| `[QUICKYHUD]` | Toggle QuickyHUD ADJUSTMODE (90266 `"On"`). Visible only when hudproxy is present. |
| `[DONE]` | Leave helper mode and return to the normal pose menu. |

## When to use which save

- **Reposition while sitting, no save:** touch → `[ADJUST]` → arrows. Session-only.
- **New default for this pose (creator):** ADJUSTMODE + HUD, or `[HELPER]` → `[SAVE]`. Writes the LSD pose default for everyone.
- **Personal offset, just you, this pose:** HUD nudge with ADJUSTMODE `Off`, or `[SAVE]` (gated on `[QS]offset` via `qs:offset:alive`) → `QSO:<short>:<slot>:<pose>`.
- **Personal offset, just you, all poses:** `[SAVE ALL]` → the magic `M#T!` fallback entry.
- **Add a new pose:** `[NEW]` (from the `[HELPER]` or ADJUSTMODE menu) → type a name → adjust → save.

## `[DUMP]` — captured changes back to notecard format

`[DUMP]` reads everything in `qs:cfg:*`, `qs:sitter:*`, `qs:p:*` and rebuilds it into AVpos directive syntax. `[QS]adjuster`'s contribution is one line — it sends 90098 to boot to kick the cascade (with a `"quiet"` marker on the ADJUSTMODE path); boot owns the rest. Output depends on the path:

- **Quiet (ADJUSTMODE path):** only the settings-service URL and the `COPY ABOVE/BELOW` banners are shouted to the owner. No per-line chat — the URL is the deliverable.
- **Loud (`[HELPER]` path):** streamed to local chat in chunks **and** uploaded; boot shouts the URL at the end. Stock-style, for users without the HUD.

See [Boot Sequence](boot-sequence.html).

## Why `[HELPER] [SAVE]` triggers 90263

In stock AVsitter, `[SAVE]` only updates the pose default in memory. The seated avatar is NOT repositioned live, so stale pose-specific `CUSTOMS` entries never get a chance to re-apply on top of the new default.

QS's sitB 90301 handler deliberately calls `send_anim_info(FALSE)` so the seated avatar reflects the new default immediately. But that routes through `apply_current_anim` in sitA, which adds `MY_CUSTOMS[pose_name]` on top of the new default. Result: visible "snap" by the old offset vector.

90263 is sent **before** 90301, so sitA processes the customs eviction ahead of the 90055 chain that re-applies the pose. The seated avatar lands on the helper-bar position; future re-sits start from the new default with no carry-over offset. `M#T!` (the all-poses personal offset) is preserved by design — it isn't tied to the saved pose name.

## See also

- [Personal Pose Offsets](personal-pose-offsets.html) — `[QS]offset`'s read/write model.
- [HUD Integration](hud-integration.html) — the in-prim QuickyHUD contract.
- [Boot Sequence](boot-sequence.html) — what `[DUMP]` does internally.
- [AVpos Reference](avpos-reference.html) — the on-disk format `[DUMP]` reconstructs.
