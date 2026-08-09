---
title: Chat Commands
sidebar: home_sidebar
permalink: chat-commands.html
keywords: chat, commands, slash, slash command, channel
toc: true
---

QuickySitter has **no public chat commands**. All user-facing interaction (pose menus, adjust, swap, security) runs through `llDialog` popups opened by touching or sitting on the furniture, same as stock AVsitter 2. What does exist are two **owner-only maintenance channels**, documented below.

## `/5`: [QS]adjuster (owner only)

`[QS]adjuster` listens on fixed channel 5, filtered to the owner:

| Command | What it does |
|---------|--------------|
| `/5 targets` | Labels every assigned seat prim with floating text showing its `SET-SLOT` pair (link message 90298 to the sitA scripts). Handy for verifying prim-description seat pinning; see [SitTargets](sittargets.html). |
| `/5 helper` | Starts the classic `[HELPER]` flow for the slot-0 sitter. Requires someone to be seated. The bars are live handles; persisting still happens per pose via `[SAVE]` from a menu, see [Adjustment Workflow](adjustment-workflow.html). |
| `/5 adjust owner\|group\|all` | Sets the **Adjust ACL** level (same setting as `[SECURITY]` → `Adjust`) without opening the dialog. Works with or without `[QS]root-security` in the build; resets to owner-only when the furniture changes owner. |
| `/5 cleanup` | Build finalization: removes `[QS]adjuster` and the `[AV]helper` object from the prim (mirrors the stock AVsitter cleanup step), and broadcasts a finalize signal that creator-only plugins answer by removing themselves — with the Creator Edition's Animesh plugin present, the whole animesh kit (dummy bodies, `animeshconfig`, both scripts, rezzed dummies) goes too, no extra `[FINALIZE]` needed. Removing the adjuster is what switches authoring off: without it no `[HELPER]`/`[HELPER HUD]`/`[NEW]`/`[DUMP]`/`[SAVE]` exists on the piece, while personal pose offsets keep working. |

One more channel-5 command is handled by the rezzed `[AV]helper` stick itself, not by the adjuster (stock behavior, filtered to the current helper **controller**, not the owner): saying an avatar UUID (`/5 <uuid>`) moves the stick onto that avatar's position if they are within 10 m, the classic trick for copying positions from poseballs.

## `/88`: [QS]debug (owner only, optional)

The optional `[QS]debug` script is a Linkset-Data inspector for creators. On install it announces "ready on /88"; `/88 help` prints the command list:

| Command | What it does |
|---------|--------------|
| `/88 help` | Show the command list. |
| `/88 keys [pattern]` | List `qs:*` keys (optional regex pattern). |
| `/88 count <ch>` | Pose count for a channel. |
| `/88 meta <ch>` / `/88 cfg <ch>` / `/88 sitter <ch>` | Show the channel's `qs:meta` / `qs:cfg` / `qs:sitter` row. |
| `/88 pose <ch> <i>` | Show one `qs:p:<ch>:<i>` row, parsed into fields. |
| `/88 poses <ch>` | Dump all poses of a channel. |
| `/88 grep <text>` | Find all `qs:p:*` rows whose value contains `<text>`. |
| `/88 raw <key>` | Raw value of any LSD key. |
| `/88 mem` | LSD bytes used / free. |
| `/88 delch <ch>` | Delete all `qs:*:<ch>` keys (destructive). |
| `/88 nuke yes` | Wipe ALL Linkset Data (destructive; plain `nuke` just warns). |
| `/88 stress start` / `stop` / `status` / `speed` | hudproxy stress-test driver. |

See [Debug Flags](debug-flags.html) for the wider diagnostics story (the `VERBOSE` chat-verbosity ladder).

## Dialog channels (internal)

Menu dialogs listen on a random negative channel rolled per dialog and bound to the avatar the dialog was sent to. There are no fixed, guessable menu channels, and nothing in QuickySitter listens on channel 0 or 1. Plugin authors interact via link messages, not chat; see [LinkMessage Numbers](linkmessage-numbers.html).

## See also

- [Adjustment Workflow](adjustment-workflow.html): the `[HELPER]` and ADJUSTMODE flows the `/5` commands feed into.
- [Debug Flags](debug-flags.html): `VERBOSE` levels and what each script logs.
- [SitTargets](sittargets.html): what the `SET-SLOT` labels from `/5 targets` mean.
