---
title: Options Menu Plugins (LinkMsg 90212)
sidebar: home_sidebar
permalink: options-menu-plugins.html
keywords: qsplug_register, 90212, options menu, plug-and-play, plugin button, plugin registration
toc: true
---

QuickySitter exposes a public extension point that lets a third-party plugin add a button to the furniture's pose menu **without editing any of the fork's core scripts**. The plugin announces its button at runtime via one link-message; `[QS]sitB` renders it under a new top-level `[OPTIONS]` entry and dispatches clicks straight back to the plugin's chosen channel.

This is the **integration** half of the fork's plug-and-play story. The **discovery** half — "is QuickySitter even present here?", "how many sitter slots?" — is [QSALIVE](qsalive-discovery.html). The two protocols complement each other (see [Relationship to QSALIVE](#relationship-to-qsalive) below); most plugins with UI use both.

## When you need this

You're writing a plugin script (e.g. a texture rotator, a wearable picker, a notification gateway) and you want users to reach it from the furniture's pose menu. Before QSPLUG_REGISTER you had to either:

- ask a creator to patch `[QS]sitA`/`[QS]sitB` and add a hard-coded button (high friction, doesn't scale, breaks on fork upgrade), or
- hijack an existing channel like `90100` and hope your label doesn't collide with anyone else's (fragile, name-dependent).

QSPLUG_REGISTER replaces both. Your plugin announces once, sitB caches your label, the user sees `[OPTIONS]` in the pose menu, clicks lead straight back to you.

## The handshake

| Num    | Direction              | `msg`                                | `id`              | Meaning |
|--------|------------------------|--------------------------------------|-------------------|---------|
| 90212  | plugin → `[QS]sitB`    | `<label>\|<click_chan>\|<scriptName>` | `""`              | "Add this button to the `[OPTIONS]` menu." |
| `<click_chan>` | `[QS]sitB` → plugin | `<label>`                       | `<controller-key>` | "User clicked your button." Fired when the avatar picks the button in the `[OPTIONS]` dialog. |

## Announce payload

Pipe-delimited. **Use `llParseString2List`, not `llParseStringKeepNulls`** — empty trailing fields would otherwise sneak garbage into the parser. (sitB itself parses this way; doing the same in your plugin keeps you on the safe side if you ever round-trip the payload.)

| Field | Content |
|-------|---------|
| 0 | Button label as it appears in the dialog (e.g. `[MYPLUGIN]`). Convention is bracket-wrapped uppercase for visual parity with built-in buttons, but anything llDialog accepts works. |
| 1 | Click channel — the LinkMessage `num` sitB fires when the user picks your button. Pick a free number outside the fork-reserved ranges: 90212–90229 is one free band, 90232–90259 is another. Document your pick in your plugin's README. |
| 2 | `llGetScriptName()` of the announcing script. Used as the dedupe key — a re-announce on plugin reset / inventory change overwrites the existing registry slot instead of appending a duplicate. |

## What sitB does with this

sitB caches registrations in a strided-3 RAM list `QSPLUG_REGISTRY = [label, click_chan, scriptName, ...]`. The animation menu builder shows an `[OPTIONS]` top-level button in the pose menu **only when the registry is non-empty** — a furniture without any plug-and-play plugins installed looks identical to furniture from before this feature existed.

Clicking `[OPTIONS]` opens a dedicated dialog listing every registered label. The dialog automatically pages with `[<<]`/`[>>]` when more than ~10 plugins are installed (the exact cap is 11 labels per page when no paging is needed, 9 when paging is on, because sitB always shows `[BACK]` and reserves room for the nav buttons).

On click, sitB fires:

```
llMessageLinked(LINK_SET, <your_click_chan>, <label>, <controller-key>);
```

`<controller-key>` is the avatar who picked the button in the dialog. There's no adjuster hop, no extra round-trip through sitA — your plugin handles the event directly in its own `link_message`.

## Adoption pattern for plugin authors

Minimal plugin, fully working:

```lsl
integer QSPLUG_REGISTER = 90212;
integer QSALIVE_REPLY   = 90097;
integer MY_CLICK_CHAN   = 90234;   // pick a free channel, document it
string  MY_LABEL        = "[MYPLUGIN]";

register_button()
{
    llMessageLinked(LINK_SET, QSPLUG_REGISTER,
        MY_LABEL + "|" + (string)MY_CLICK_CHAN + "|" + llGetScriptName(),
        "");
}

default
{
    state_entry()  { register_button(); }
    on_rez(integer p) { register_button(); }

    changed(integer c)
    {
        if (c & CHANGED_INVENTORY) register_button();
    }

    link_message(integer sender, integer num, string msg, key id)
    {
        // sitA broadcasts QSALIVE_REPLY unsolicited on its state_entry.
        // If sitA reset, sitB likely reloaded too and the registry is
        // empty — re-announce. Idempotent if sitB still has us.
        if (num == QSALIVE_REPLY)
        {
            register_button();
            return;
        }
        if (num == MY_CLICK_CHAN)
        {
            // id = controller key, msg = our label
            // ... your handler ...
            return;
        }
    }
}
```

The fork ships a copy of this script — verbatim including its comments — as [`qs/examples/[QS]plugin-example.lsl`](https://github.com/QuickyProducts/QuickySitter/blob/master/qs/examples/%5BQS%5Dplugin-example.lsl) in the main repo. Multi-copy and rename to exercise paging and dedupe.

## Re-announce: the four triggers

Plug-and-play registry survives sitB resets, plugin resets, inventory changes, and renames — but only because the plugin re-announces in the right moments. The reference pattern covers all four:

| Event | Why you re-announce |
|-------|---------------------|
| `state_entry` | Plugin just (re-)started. Initial registration. |
| `on_rez` | Furniture rezzed somewhere new. Cheaper than checking whether sitB lost our entry. |
| `changed(CHANGED_INVENTORY)` | Our own script may have been renamed by the creator dragging it in inventory. `scriptName` changed → dedupe key changed → sitB's old entry is now a ghost. Cheapest fix: re-announce; sitB's dedupe loop reconciles. |
| `link_message(num == 90097)` | sitA's unsolicited QSALIVE broadcast. Means sitA just (re-)booted — sitB likely went through `QS_BOOT_RELOAD` and the registry is empty. Re-announce; idempotent if sitB already has us. |

Skip any of these and your button stays correct in the *common* case but vanishes in the *edge* case. The full reference plugin (above) handles all four; copy-paste freely.

## Relationship to QSALIVE

QSPLUG_REGISTER and [QSALIVE](qsalive-discovery.html) are sibling protocols at different layers of the same plug-and-play architecture:

| | **QSALIVE** (90096/90097) | **QSPLUG_REGISTER** (90212) |
|---|---|---|
| **Layer** | Discovery — "is the host present?" | Integration — "I want a UI slot." |
| **Direction** | bidirectional (plugin probes, sitA replies) | unidirectional (plugin → sitB) |
| **Receiver** | sitA slot 0 | sitB (per-slot, but registry is shared) |
| **Statefulness** | stateless — each probe is fresh | stateful — sitB caches the registry |
| **Trigger** | plugin probes; sitA broadcasts unsolicited on state_entry | plugin announces on state_entry / on_rez / inventory change / 90097-receipt |
| **Payload** | `product\|ver\|sitter_count\|caps_CSV` | `label\|click_chan\|scriptName` |
| **Use case** | "Should I activate at all? Fall back to legacy AVsitter?" | "Add my button to the menu." |

A plugin **with UI** uses both: QSALIVE to detect QuickySitter (and fall back to legacy AVsitter if not present); QSPLUG_REGISTER to claim its menu slot. A plugin **without UI** (logger, state mirror, analytics) only needs QSALIVE.

The most important cross-wiring is: **listen to 90097 and re-announce on it**. sitA's unsolicited 90097 broadcast is the cheapest possible "the world just rebooted" signal — sitB likely went through its own `QS_BOOT_RELOAD` cascade and dropped your entry. The boilerplate above does this in one line.

## Limits and v1 scope

- **`[OPTIONS]` menu only.** The pose menu's main button strip (`[ADJUST]`, `[NEW]`, `[DUMP]`) and the `[ADJUST]` submenu are not Plugin-Registry targets in v1. Plugin authors who want buttons there must still patch sitB directly — or use the legacy notecard `ADJUST` line, which is functionally identical to a registry entry but boot-time and scoped to the ADJUST submenu (see [Adjustment Workflow](adjustment-workflow.html#adjust_menu)).
- **No active staleness probe in v1.** If a plugin script crashes silently between announces, its label stays in the registry until sitB resets (which re-issues a 90097 broadcast, prompting all surviving plugins to re-announce). `CHANGED_INVENTORY` in the plugin → re-announce is the recommended path; script removal is not detected actively. v2 may add a probe channel mirroring the [HUDPROXY 90093 pattern](hud-integration.html#hudproxy-presence).
- **Order = announce order.** First plugin to register gets the first slot in the `[OPTIONS]` dialog. No priority field in v1.
- **Click `id` is the controller key only.** sitA's legacy `<controller>|<sitter>` composite (used by the 90101 ADJUST_MENU dispatch when `AMENU & 4` is unset) is not emulated. Plugins that need the sitter key can query via QSALIVE or read the `qs:sitter:<ch>` LSD keys.

## FAQ

**Why does `[OPTIONS]` only show up sometimes?**
The button is gated on `QSPLUG_REGISTRY` being non-empty. If no plug-and-play plugin has registered yet (slow plugin state_entry, plugin not present, plugin crashed), the button is hidden — the pose menu looks as it did before this feature existed.

**My plugin's label disappeared after I renamed the script in inventory.**
Expected: the dedupe key is `llGetScriptName()`, which just changed. The old entry is still in the registry under the *old* name; the new name has no entry yet. Adding `changed(CHANGED_INVENTORY) { register_button(); }` (in the boilerplate above) fixes it — sitB will see the new name as a fresh registration. The old ghost entry gets flushed on the next sitB / sitA reset.

**Two of my plugins want the same button name.**
Won't work — sitB dedupes by `scriptName`, so two scripts with different names both register, but they both display their `<label>`. If both pick `[FOO]`, the dialog shows `[FOO]` twice and clicking either one is ambiguous from the user's side. Use distinct labels.

**Can I have multiple buttons per plugin?**
Not from a single script. Workaround: drop the plugin script twice with different filenames and different `MY_LABEL` / `MY_CLICK_CHAN` constants. Each instance registers independently.

**What if two plugins pick the same `click_chan`?**
Both will receive every click for that channel. Plugin authors must coordinate channel picks (or just be lucky in the 48-slot free band). The fork doesn't validate against duplicates — it just dispatches to whatever channel you announced.

## See also

- [QSALIVE Discovery](qsalive-discovery.html) — presence handshake, the discovery counterpart.
- [LinkMessage Numbers](linkmessage-numbers.html) — full channel map including free ranges for click channels.
- [HUD Integration](hud-integration.html) — for an example of the active-probe pattern v2 might adopt.
- [`qs/examples/[QS]plugin-example.lsl`](https://github.com/QuickyProducts/QuickySitter/tree/master/qs/examples) — the reference plugin, ready to drop in.
- [`qs/PROTOCOL.md § QSPLUG_REGISTER`](https://github.com/QuickyProducts/QuickySitter/blob/master/qs/PROTOCOL.md) — the in-repo spec (this page mirrors it).
