---
title: Re-Sync Protocol (LinkMsg 90271)
sidebar: home_sidebar
permalink: resync-protocol.html
keywords: resync, sync, 90271, drift, sync poses
toc: true
---

Multi-avatar SYNC poses (loops with shared timing, such as cuddles and dances) drift between viewers over time. The most common cause: a viewer culls and re-acquires an avatar (camera zoom, region crossing, draw-distance change). On re-acquisition the viewer restarts the looped animation locally at `t=0`, while other viewers keep their original timeline.

QuickySitter exposes a single LinkMsg that any in-prim script can send to force every sitter slot to re-phase its main pose loop in the same Sim frame.

## The trigger

| Num   | Direction                                | `msg` | `id` | Meaning |
|-------|------------------------------------------|-------|------|---------|
| 90271 | hudproxy / any in-prim source → all `[QS]sitA` slots | `""` | `""` | "Every SYNC-pose sitter, do one Stop+Start cycle on your main anim now." |

That's the entire integration. Sending the LinkMsg is one line:

```lsl
llMessageLinked(LINK_SET, 90271, "", "");
```

Handled by current `[QS]sitA` (release 1.25; scripts carry per-script versions between releases, uniform only at a release stamp).

## Safe to broadcast

`[QS]sitA` applies the trigger only when it makes sense: a SYNC pose is playing (no `P:` prefix), a sitter is seated, and animation permission is held. Anything else (an empty sitter, a solo `P:` pose, a permission-pending window) is silently ignored. Broadcasting 90271 is therefore always harmless.

*When* and *how often* to send is entirely the sender's decision. QuickyHUD's SYNC button is one such sender; any in-prim script can be another.

## See also

- [QSALIVE Discovery](qsalive-discovery.html): presence protocol that lets hudproxy detect QS before sending 90271.
- [HUD Integration](hud-integration.html): full hudproxy ↔ adjuster handshake.
- [LinkMessage Numbers](linkmessage-numbers.html): the complete fork link-message map.
