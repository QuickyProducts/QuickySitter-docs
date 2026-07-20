---
title: Chat Commands
sidebar: home_sidebar
permalink: chat-commands.html
keywords: chat, commands, slash, slash command, channel
toc: true
---

QuickySitter inherits the user-facing chat commands from stock AVsitter 2 (the `/1 …` family below behaves the same). It also adds its own owner-only diagnostics channel: `/88`, served by the optional `[QS]debug` script; see [Debug Flags](debug-flags.html). This page is a quick-reference; for the conceptual tutorial see the [upstream AVsitter chat commands page](https://avsitter.github.io/avsitter2_home.html).

## Public commands (chat channel 0)

Issued by anyone with access (typically the owner or sitter), spoken in local chat.

| Command | What it does |
|---------|--------------|
| `/1 menu` | Open the pose menu for the speaker. |
| `/1 reset` | Reset the prim (creator only). |
| `/1 set <n>` | Change to sit-target set `n` (creator only). |

Public commands run through `[QS]sitA`'s `listen` event on the configured chat channel.

## Helper-bar shortcuts

When in `[HELPER]` adjustment mode, the chat input is gated to a helper-bar channel. Type values directly:

| Input | Effect |
|-------|--------|
| `<x>,<y>,<z>` | Set POS offset to that exact vector (overrides arrow nudges). |
| `<rot_x>,<rot_y>,<rot_z>` | Set ROT offset to Euler degrees vector. |
| `save` | Same as clicking `[SAVE]`. |
| `cancel` | Exit helper without saving. |

These shortcuts are documented in the upstream adjuster tutorial.

## Listen channel layout

Each script listens on a channel derived from its slot and purpose. Plugin authors don't usually need to know these (link-messages are the public API), but for debugging:

| Script | Listen channel | Purpose |
|--------|----------------|---------|
| `[QS]sitA` slot N | Sit-channel derived from `llGetOwner` and slot | Per-sitter dialog menus, public commands. |
| `[QS]adjuster` | Helper-bar channel | Adjustment values from chat. |
| `[QS]select` (optional) | Cross-furniture select channel | Multi-furniture routing. Only present when the optional `[QS]select` plugin is installed. |
| `[QS]debug` (optional) | `/88` (owner-only) | LSD inspector + stress-test commands. See [Debug Flags](debug-flags.html). |

## Dialog interaction

Most user interaction goes through `llDialog` blue-popup menus, not chat. The `[HELPER]` menu is the main entry point; from there:

- `[NEW]`: create a new pose stub.
- `[SAVE]`: commit current adjustments to LSD.
- `[SAVE ALL]`: set the `M#T!` all-poses fallback for this user/slot. See [Personal Pose Offsets](personal-pose-offsets.html).
- `[DUMP]`: kick the `90098` cascade in boot, then chat or upload the LSD-formatted AVpos text.
- `[CANCEL]`: exit without saving.
- `[STOP HELP]`: leave helper mode entirely.

## RLV commands (with `[AV]root-RLV`)

When the stock `[AV]root-RLV` plugin is in the prim, additional RLV-style commands become available. These are documented in the upstream AVsitter RLV docs.

## See also

- [Adjustment Workflow](adjustment-workflow.html): full `[HELPER]` menu walkthrough.
- [Submenus](submenus.html): menu navigation model.
- [LinkMessage Numbers](linkmessage-numbers.html): what each menu click sends internally.
