---
title: '[AV]camera'
sidebar: home_sidebar
permalink: plugin-camera.html
keywords: camera, plugin, view, llSetLinkCamera
toc: true
---

The camera plugin in QuickySitter is **stock `[AV]camera` from AVsitter 2**, not forked. Stock camera's only name-bound code (`get_number_of_scripts` via `main_script = "[AV]sitA"`) is dead code (never called anywhere in the file), and all working paths are protocol-based and script-name-agnostic.

No `[QS]camera` is planned. The `camera_script` literal in `[QS]boot` stays as legitimate AVsitter-protocol surface, because boot hardcodes the name only for the DUMP cascade (sending 90020 to ask camera to dump its CAMERA lines).

## Notecard syntax

Camera presets are declared with `CAMERA` directives in `AVpos`, one per line. The format is:

```
CAMERA <trigger>|<eye_position>|<focus_position>
```

`<trigger>` is the pose name (or `DEFAULT` for the un-posed camera). Both positions are vectors relative to the prim's pivot.

```
CAMERA DEFAULT|<-0.09605, -2.75508, 1.23718>|<-0.07217, -1.81931, 0.88538>
CAMERA Sit1|<0.12466, -1.65931, 3.34216>|<0.12805, -1.20604, 2.45079>
```

Full reference in the [upstream AVcamera page](https://avsitter.github.io/avsitter2_camera.html).

## Link messages used

| Num | Direction | Use |
|-----|-----------|-----|
| `90020` | `[QS]boot` → `[AV]camera` | DUMP request. Boot sends this hardcoded for camera (no QSDUMP announce). |
| `90021` | `[AV]camera` → `[QS]boot` | DUMP complete echo. |
| `90022` | `[AV]camera` → `[QS]boot` | One dump line. |
| `90174` | `[QS]adjuster` → `[AV]camera` | Add CAMERA line at runtime. |
| `90230` | various → `[AV]camera` | Set camera by name (also sends a 90005 menu-reopen echo to the controller). |
| `90231` | various → `[AV]camera` | Set camera by name, identical to 90230 but **without** the 90005 menu-reopen echo. |

These are all stock AVsitter numbers used with stock semantics. Neither 90230 nor 90231 is a clear/reset: a camera reset is the message string `"RESET"` sent on either number (it restores the last camera and drops the by-button lock).

## Should `[QS]camera` ever be forked?

Only if a use case appears that needs script-name-independent gating or QSDUMP-style discovery. Currently no plans:

- Camera doesn't have a menu button that would need gating (camera switches happen via button-bound 90230 messages from pose entries).
- Two hardcoded `"[AV]camera"` name-bindings remain: `camera_script` in `[QS]boot` (for the DUMP cascade) and the same literal in `[QS]adjuster` (which sends the 90174 / 90230 / 90231 messages to the camera). Both are confined to literals that are trivial to update if needed.

If you're building a third-party camera plugin, follow the QSALIVE adoption pattern from [QSALIVE Discovery](qsalive-discovery.html) so it can detect QS at runtime without locking to a specific sitter-script name.

## See also

- [Upstream AVcamera documentation](https://avsitter.github.io/avsitter2_camera.html).
- [LinkMessage Numbers](linkmessage-numbers.html): full link-message map.
- [Boot Sequence § QSDUMP](boot-sequence.html#qsdump-plugin-announce-for-the-dump-cascade): why camera stays hardcoded.
