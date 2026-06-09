---
title: Re-Sync Protocol (LinkMsg 90271)
sidebar: home_sidebar
permalink: resync-protocol.html
keywords: resync, sync, 90271, drift, sync poses
toc: true
---

Multi-avatar SYNC poses (loops with shared timing — cuddles, dances) drift between viewers over time. The most common cause: a viewer culls and re-acquires an avatar (camera zoom, region crossing, draw-distance change). On re-acquisition the viewer restarts the looped animation locally at `t=0`, while other viewers keep their original timeline.

QuickySitter exposes a single LinkMsg that any in-prim script can send to force every sitter slot to re-phase its main pose loop in the same Sim frame.

## The trigger

| Num   | Direction                                | `msg` | `id` | Meaning |
|-------|------------------------------------------|-------|------|---------|
| 90271 | hudproxy / any in-prim source → all `[QS]sitA` slots | `""` | `""` | "Every SYNC-pose sitter, do one Stop+Start cycle on your main anim now." |

That's the entire integration. Sending the LinkMsg is one line:

```lsl
llMessageLinked(LINK_SET, 90271, "", "");
```

Handled by current `[QS]sitA` (the receiver shipped during the unified-version line; all shipped scripts now report `0.999`).

## Mechanism

On receipt, each `[QS]sitA` instance whose current pose is a SYNC pose (name not prefixed `P:`) and whose sitter is alive runs:

```lsl
llStopAnimation(CURRENT_ANIMATION_FILENAME);
llSleep(0.05);
llStartAnimation(CURRENT_ANIMATION_FILENAME);
```

The 50 ms sleep is just long enough to cross a Sim-frame boundary so Stop and Start aren't coalesced into a no-op (Sim runs at ~45 Hz / 22 ms per frame), short enough that most viewers' next render frame falls outside the gap. Stop+Start is the only mechanism that actually re-phases a running loop on the viewer side — the viewer determines loop phase locally at the `Start` event.

POSE-type poses (prefixed `P:`) are solo-by-convention and don't need re-sync — `do_resync_tick` no-ops on them.

## Gating conditions

`do_resync_tick()` returns early unless **all** of the following hold:

- current pose is a SYNC pose (no `P:` prefix)
- `PERMISSION_TRIGGER_ANIMATION` is granted
- sitter is alive (`llGetAgentSize != ZERO_VECTOR`)
- `CURRENT_ANIMATION_FILENAME` is non-empty

These guards mean a 90271 burst on an empty sitter, a solo P:-pose, or during a permission-pending window is harmless.

## Policy lives on the sender side

`[QS]sitA` deliberately knows nothing about *when* to re-sync. It just executes the trigger when asked. The sender (typically [hudproxy](hud-integration.html) in QuickyHUD setups) decides:

- **Auto vs manual** (the user's HUD setting)
- **Tick interval** (e.g., every 30 s, or only on user-noticed drift)
- **Per-furniture overrides** (HUD might disable Re-Sync for solo furnitures, or for furnitures marked solo by the creator)

This split came after several iterations (sitA 0.16 – 0.21) tried to own auto-tick scheduling inside sitA itself: a wall-clock-aligned 30 s timer, a notecard `RESYNC OFF` directive, and a dummy-animation refresh trick. All three were abandoned — the dummy-anim trick refreshes skeleton state but not loop phase (architecturally cannot do what it was meant to do), and the auto-tick approach competed with the natural sequence timer in sitA in ways that didn't add value over a HUD-driven trigger.

## What hudproxy must do

When the user clicks the SYNC button on the HUD, hudproxy sends one LinkMsg in the furniture's linkset:

```lsl
llMessageLinked(LINK_SET, 90271, "", "");
```

Auto-tick (if hudproxy implements it) is a `llSetTimerEvent` loop on hudproxy's side that fires the same LinkMsg every N seconds. Nothing about that policy reaches sitA.

## Multi-sitter timing

All `[QS]sitA` slots in the linkset receive 90271 in the same Sim frame (`LINK_SET` broadcast). Each runs its own Stop+Sleep+Start, with the Sim processing them sequentially within the frame. The resulting viewer-side restarts arrive within one Sim frame of each other — close enough that drift between sitters is corrected to within ~50 ms.

## See also

- [QSALIVE Discovery](qsalive-discovery.html) — presence protocol that lets hudproxy detect QS before sending 90271.
- [HUD Integration](hud-integration.html) — full hudproxy ↔ adjuster handshake.
- [LinkMessage Numbers](linkmessage-numbers.html) — the complete fork link-message map.
