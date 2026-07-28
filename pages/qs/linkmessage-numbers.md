---
title: LinkMessage Numbers
sidebar: home_sidebar
permalink: linkmessage-numbers.html
keywords: link message, linkmsg, reference, 90000, 90500
toc: true
---

QuickySitter uses link-message numbers in the range **90000 – 90500**, identical to stock AVsitter 2. Stock numbers are unchanged from a sender's perspective: a stock plugin's link-message traffic works as-is in QuickySitter furniture. (Script-**name** probes are another matter: see [Compatibility Matrix](compatibility-matrix.html).)

This page lists the **fork-specific numbers** QuickySitter adds (in stock-unused ranges) and notes the stock numbers whose handler script moved or whose semantics changed slightly. For the complete stock AVsitter 2 reference, see [`avsitter2_link_message_reference.md`](https://github.com/QuickyProducts/QuickySitter/blob/master/avstock/avsitter2_link_message_reference.md) in the QS repo (vendored copy of upstream).

## Stock numbers used unchanged

`90000`-`90014`, `90030` (SWAP), `90033`, `90045` (pose-played broadcast), `90050`/`90051` (menu pose pick), `90055`/`90056` (anim info), `90057` (helper move), `90060`/`90065`/`90070` (sit/unsit/permissions), `90075`/`90076` (oldschool helper), `90100`/`90101` (menu choice), `90150`-`90211`, `90220` (play pose without opening the menu; `[QS]prop` handles it alongside `90200`), `90230`, `90298`-`90300`, `90401`-`90500`. From a sender's perspective the contracts match stock.

## Stock numbers whose handler script moved

| Num | Stock home | QuickySitter home | Why |
|-----|-----------|-------------------|-----|
| `90020` | sent to scripts asking for `[DUMP]` | sent **from `[QS]boot`** instead of adjuster | `[DUMP]` ownership moved to boot |
| `90021` | handled by `[AV]adjuster` | handled by **`[QS]boot`** | same: boot owns the cascade |
| `90022` | handled by `[AV]adjuster` | handled by **`[QS]boot`** | same: boot owns the receiver |

Plugins still send `90022`/`90021` to `LINK_THIS` exactly like in stock; the listener just lives in a different script in the same prim.

## Stock numbers with subtler semantic changes

| Num | Change |
|-----|--------|
| `90301` | sitB's handler is stricter: it only refreshes the seated avatar when the saved pose is the one currently playing, compared by pose **name** (not by index) with a payload-shape check, and forwards pos/rot **directly from the 90301 payload** instead of re-reading LSD. Sender contract (`name\|pos\|rot\|`) unchanged. Stock plugins don't send 90301 (it was sitA→sitB internal), so this is invisible externally. |

## Stock numbers no longer routed

| Num | Status |
|-----|--------|
| `90302` ("sitA sends initial notecard settings to sitB") | Removed. sitB reads `qs:cfg:<ch>` from LSD directly in `state_entry`. Incoming 90302 is silently ignored. |
| `90020` → sitB | sitB is no longer a `[DUMP]` source (boot owns the dump). Incoming 90020 to sitB is silently ignored. |

## Fork-specific numbers

All in stock-unused ranges. A stock-AVsitter plugin sending or receiving in these ranges would have collided with something, but the stock reference shows these slots as unused.

### Boot self-check + reload (9002x)

| Num | Direction | Use |
|-----|-----------|-----|
| `90023` | `[QS]boot` → all | `QS_BOOT_RELOAD`. Emitted at the end of the seed cascade. `[QS]sitB` reloads its page state from LSD on receipt (the retired MENU_LIST is gone), and `[QS]sitA` reloads too, eliminating the manual-reset step after a notecard re-save. |
| `90024` | `[QS]boot` → all | `QS_BOOT_WIPE`. Broadcast **before** boot wipes the seeded LSD and resets, when a notecard re-save invalidates the seeded state. sitA/sitB flip their booted flag back to FALSE so pre-boot guards re-engage until `QS_BOOT_RELOAD` (90023) fires. |

See [Boot Sequence](boot-sequence.html).

### Boot self-check probe (9007x)

| Num | Direction | Use |
|-----|-----------|-----|
| `90077` | `[QS]boot` → `[QS]sitB` | Boot self-check probe ("is the menu pipeline present?"). One-shot from boot's `state_entry`. |
| `90078` | `[QS]sitB` → `[QS]boot` | Boot self-check reply. |

### Plugin presence (9007x – 9009x)

Plugin presence is **not** a link-message handshake. Each plugin writes a `qs:alive:<name>` LSD flag in its `state_entry` (the offset plugin uses the inverted name `qs:offset:alive`); menu builders read those flags on demand, never caching. The only presence-related link-message is the census re-stamp:

| Num | Direction | Use |
|-----|-----------|-----|
| `90079` | `[QS]boot` → all | `QS_ALIVE_CENSUS`. boot wipes every `qs:alive:*` flag and broadcasts this; surviving plugins re-stamp their flag in response, so a removed plugin drops out without an inventory probe. |
| `90093` | bidirectional | hudproxy presence probe (the live HUD-presence HELLO; `QSDUMP_HELLO` 90095 and `QS_SITB_HELLO` 90078 are also live, for the dump cascade and the boot self-check respectively). See [HUD Integration](hud-integration.html). |
| `90094` | `[QS]boot` → all plugins | QSDUMP probe: "if you're DUMP-capable, announce yourself now." |
| `90095` | DUMP plugin → `[QS]boot` | QSDUMP hello: "I respond to 90020 DUMP messages." |
| `90096` | plugin → `[QS]sitA` | QSALIVE count/version/caps probe (**not** a presence handshake). See [QSALIVE Discovery](qsalive-discovery.html). |
| `90097` | `[QS]sitA` (slot 0) → plugin | QSALIVE count/version/caps reply, plus one unsolicited boot-announce. |

### DUMP cascade ownership (9009x)

| Num | Direction | Use |
|-----|-----------|-----|
| `90098` | `[QS]adjuster` → `[QS]boot` | "Start dump for channel." Replaces stock adjuster-owned `[DUMP]`. `id` is a mode marker: `"quiet"` for the ADJUSTMODE live-view dump (URL only, chat banners suppressed), `""` (or `"loud"`) for the operator-visible `[DUMP]`. The boot self-check (90077/90078) never dumps. |
| `90099` | `[QS]boot` → self | Dump tick: self-trigger between dump-line iterations. |

### Quiet swap (9003x)

| Num | Direction | Use |
|-----|-----------|-----|
| `90031` | menu source → `[QS]sitA` | Quiet SWAP, like stock `90030` SWAP but suppresses the swap announcement. Fork addition; stock furniture has no 90031 sender. |

### Plug-and-play plugin registry (9021x)

| Num | Direction | Use |
|-----|-----------|-----|
| `90204` | `[QS]adjuster` → `[QS]root-security` | `QSADJ_ACL_SET`: apply an Adjust-ACL level from the `/5 adjust` chat command, `msg = "OWNER"` / `"GROUP"` / `"ALL"`. A stock-free slot inside the otherwise stock `90200` band. The adjuster sends it and writes `qs:sec:adjust` itself, both unconditionally, so the plugin's RAM state cannot go stale and revert the level on the next census. Since 1.26. |
| `90212` | plugin → `[QS]sitB` | QSPLUG_REGISTER: `msg = "<label>\|<click_chan>\|<scriptName>"`. Registers a runtime button into the `[OPTIONS]` top-level menu. sitB dedupes by `scriptName`. Click dispatch lands on `<click_chan>` with `msg = <label>`, `id = <controller-key>`. See [Options Menu Plugins](options-menu-plugins.html). |
| `90213` | plugin → `[QS]sitB` | QSADJ_REGISTER: `msg = "<label>\|<click_chan>\|<scriptName>\|<flags>"`. Like QSPLUG_REGISTER but the button lands in the `[ADJUST]` submenu instead of `[OPTIONS]`. `flags` bit 0 = owner-only (rendered per the Adjust ACL, like `[HELPER HUD]`). RAM registry, re-announced by the plugin on QSALIVE_REPLY (90097) so it survives a re-seed. |

### Personal pose offsets (9026x)

| Num | Direction | Use |
|-----|-----------|-----|
| `90214` | `[QS]adjuster` → `[QS]faces` | `QSFACE_PICK`: open the facial-anim picker for a sitter slot. `msg = "<slot>\|<controller>\|<sitterAv>"`. Prim-local (sent `LINK_THIS`, and the receiver refuses a cross-prim sender), because the slot indexes the receiving prim's own sitter list. Since 1.26. |
| `90215` | `[QS]adjuster` → creator-only plugins | `QS_FINALIZE`: broadcast by `/5 cleanup` just before the adjuster removes itself. Subscribers tear THEMSELVES down; the adjuster knows no plugin inventory names. Unheard when no such plugin is present. Since 1.26. |
| `90216` | plugin → `[QS]sitB` | `QSADJ_UNREGISTER`: drop a button registered via 90213. `msg = <scriptName>`, the same identity 90213 dedupes on. A plugin that deletes itself MUST send this first, otherwise its button outlives it and dispatches to a channel nobody listens on. Since 1.26. |
| `90238` | `[QS]sitB` → hudadmin | Click channel of the self-registered `💠 POSE HUD` entry (attach mode `menuplus`). Same handler as the 90510 button, so it toggles. Since 1.26. |
| `90260` | `[QS]offset` → `[QS]sitA` + hudproxy | "Mirror this RAM-tier personal offset." ZERO/ZERO is the delete sentinel. |
| `90261` | `[QS]sitA` + hudproxy → `[QS]offset` | "Push every RAM-tier cached offset for this (sitter, slot) pair to me." |
| `90262` | `[QS]sitA`, hudproxy, hudadmin, `[QS]debug` → `[QS]offset` | "Save this offset for (sitter, slot, pose)." Magic name `M#T!` is the all-poses fallback. |
| `90263` | `[QS]adjuster` → `[QS]sitA` + `[QS]offset` | "Drop stale customs after `[HELPER] [SAVE]`." |
| `90264` | hudadmin → `[QS]offset` | "Wipe ALL personal offsets." (hudproxy has no 90264 sender.) |
| `90265` | `[QS]offset` → all `[QS]sitA` | "Clear your RAM-tier mirror." Paired with 90264. |
| `90266` | `[QS]adjuster` + hudadmin → hudproxy | "Flip QuickyHUD ADJUSTMODE remotely": `"On"` / `"Off"`. |

See [Personal Pose Offsets](personal-pose-offsets.html).

### Retired presence HELLOs (9008x – 9009x)

These per-plugin HELLO broadcasts were the original (pre-0.9951) presence mechanism. They were **retired in 0.9951** and replaced by the `qs:alive:*` LSD flags + `QS_ALIVE_CENSUS` (90079) described above. The numbers are reserved and not reused; current scripts neither send nor listen for them.

| Num | Was | Replaced by |
|-----|-----|-------------|
| `90088` | `QS_OFFSET_HELLO`: offset presence | `qs:offset:alive` LSD flag (inverted name). |
| `90089` | `QS_PROP_HELLO`: prop presence (gated `[PROP]`) | `qs:alive:prop` LSD flag. |
| `90090` | `QS_FACES_HELLO`: faces presence (gated `[FACES]`/`[FACE]`) | `qs:alive:faces` LSD flag. |
| `90091` | `QS_ADJUSTER_HELLO`: adjuster presence (gated `[HELPER]`) | `qs:alive:adjuster` LSD flag. |
| `90092` | `QS_SELECT_HELLO`: select presence (gated select routing) | `qs:alive:select` LSD flag (sitB also keeps an `[AV]select` inventory fallback for stock-AVsitter compat). |

### Re-Sync and dynamic props (9027x – 9028x)

| Num | Direction | Use |
|-----|-----------|-----|
| `90271` | any in-prim source → `[QS]sitA` | SYNC-pose Re-Sync trigger. See [Re-Sync Protocol](resync-protocol.html). |
| `90280` | any in-prim source → `[QS]prop` | `QSPROP_ATTACH`: dynamic prop register + rez without notecard entry. See [HUD Integration](hud-integration.html). |

### QuickyHUD-internal numbers (9027x)

These live entirely inside the QuickyHUD scripts (hudproxy ↔ hudadmin, both on the furniture prim). Listed here for range completeness; a stock plugin never sees them.

| Num | Direction | Use |
|-----|-----------|-----|
| `90272` | hudproxy → hudadmin | `SELECT_OPEN_DIALOG`: render the SELECT (swap-target) picker. |
| `90273` | hudadmin → hudproxy | `SELECT_PICKED`: the chosen target token. |
| `90274` | hudproxy → hudadmin | `ATTACH_FOR_ADJUST`: ensure the operator has a HUD attached when ADJUSTMODE goes On. |
| `90275` | any source → hudproxy | `QSANIM_OCCUPANT`: animesh dummy occupant hook. See [Animesh Adjust Dummies](quickysitter-pro-animesh.html). |
| `90276` | hudproxy → linkset | `ADJUSTMODE_CHANGED`: the mode flipped, `msg = "On"` / `"Off"`. Fired from the single choke point every flip passes through. hudadmin uses the `"Off"` edge to retire HUDs that only existed for ADJUSTMODE. Nobody listens on stock AVsitter, so the broadcast is compat-neutral there. Since 1.26. |

## Compatibility summary

- **Stock plugin in QuickySitter furniture:** ✅ link-message traffic works unchanged; plugins relying on `[AV]sitA` script-name probes degrade. See [Compatibility Matrix](compatibility-matrix.html).
- **QuickySitter scripts in stock-AVsitter furniture:** ❌ doesn't work, because sitA/sitB expect `qs:cfg`/`qs:sitter`/`qs:p:*` LSD keys that boot writes during seed; stock furniture has no `[QS]boot`. This is intentional, not a goal of the fork.

See also: [Compatibility Matrix](compatibility-matrix.html), [LSD Keys](lsd-keys.html).
