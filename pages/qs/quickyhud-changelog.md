---
title: QuickySitter Pro Changelog
sidebar: home_sidebar
permalink: quickyhud-changelog.html
keywords: changelog, releases, fixes, features, quickysitter pro, quickyhud
toc: true
---

Customer-facing changes only. Each entry is tagged **Fix** (bug fix), **Feature** (new), or **Base** (groundwork shipped ahead of a separate feature). Routine internal/technical changes aren't listed. Newest version on top.

## Version 1.26

- **Feature**: The `[QUICKYHUD]` entry in the furniture's `[ADJUST]` menu is now called `[HELPER HUD]`: it is the HUD counterpart of `[HELPER]`, and the name now says what it does. Requires the matching QuickySitter update.
- **Feature**: ADJUSTMODE is now entered in one place only: the furniture's `[ADJUST]` menu (`[HELPER HUD]` entry). The "Switch ADJUSTMODE" entry in the HUD settings dialog is gone; it dated from the HUD's early days as a stand-alone AVsitter plugin and was a second, separately guarded door into the same mode. Leaving ADJUSTMODE works as before via `[DONE]`/`[ADJUST OFF]` in the pose menu, and it still switches off automatically on stand-up. On plain AVsitter furniture (without QuickySitter) ADJUSTMODE is no longer reachable.
- **Feature**: The "Quicky HUD" furniture menu button is now a toggle: press it while you are already wearing the HUD and the HUD detaches; press it again and it comes back. Until now the button could only attach, and removing the HUD needed right-click > Detach. The pose menu re-opens by itself once the HUD has been attached or removed.
- **Feature**: The menu button that hands out the HUD can now be labelled `POSE HUD` instead of `Quicky-HUD`, and the label may carry decoration around it, for example `ADJUST 💠 POSE HUD|90510` for the `[ADJUST]` dialog or `BUTTON 💠 Pose HUD|90510|POSE HUD` for the main button strip. All that matters is that one of the two names appears somewhere in it; upper or lower case makes no difference. Both names stay valid, existing furniture needs no change.
- **Fix**: Leaving ADJUSTMODE (`[DONE]` or automatically on stand-up) now also removes the HUD that `[HELPER HUD]` auto-attached for the adjustment ("menu" attach mode): it no longer lingers in personal-adjust mode nobody asked for. A HUD you started yourself via the menu button stays on, exactly as before.
- **Fix**: Seat swapping no longer silently removes the HUD in "menu" attach mode: sitters who were wearing one before the swap get it re-attached automatically, everyone else stays HUD-free as before.
- **Fix**: If another script in the furniture clears its stored settings, the Quicky HUD now recovers by itself instead of stopping for good. You are told once that it happened. Pose positions and personal adjustments that were stored there are still lost and need re-saving.

## Version 1.25

*The creator product is now called **QuickySitter Pro** (formerly "QuickyHUD Creator"). The HUD itself keeps its name and is one tool in the kit among others. From this release the kit and the QuickySitter engine share one version number: QuickySitter jumps from 1.04 to 1.25 to meet the kit.*

- **Feature**: Fresh builds start sittable: installing into an empty prim now also delivers a starter `AVpos` notecard (one seat, built-in sit animation), so the furniture works immediately. A repair run heals a piece whose `AVpos` was accidentally deleted the same way. An existing `AVpos` is never touched: the starter is only delivered when no notecard exists at all.
- **Fix**: Update safety for notecards: updates now treat every notecard in the furniture as strictly off-limits. An update push can never remove or replace your `AVpos` (all poses and positions) or any other configuration notecard.
- **Fix**: The `RESERVE` setting from the `hudconfig` notecard (storage bytes kept free for your own scripts) is now actually honored when personal pose offsets are saved; it was silently ignored before. Existing furniture is converted automatically.
- **Feature**: Showcase mode for the Animesh dummies (owner only): switch `[SHOWCASE]` on in the seat list and the staged dummies stay put when you get up, which turns a staged pose into a store display. They also survive a visitor sitting down or swapping seats: the avatar visibly shares the seat instead of the display being torn down. Note that updating or resetting the furniture clears a staged display, so re-stage after an update.
- **Feature**: Bundled QuickySitter update, prop scale & worn fit: the new `[QS]objectadjust` companion script goes into your prop and makes it resizable from the editor, or fittable directly on the body for attachment props, and QuickySitter's `[SAVE]` persists it. End users can fine-tune a rezzed prop by touch (±1/5/10 % menu, `[RESTORE]`).
- **Fix**: Bundled QuickySitter update: the new "Adjust" access level lets chosen non-owners use the adjust tools (build from your personal account while a store account owns the furniture), the `[DUMP]` settings link works again on QuickySitter's own service, and its web page now shows the classic AVsitter layout.

## Version 1.24

- **Feature**: Animesh partner-dummy plugin is here (creator tool): set up couples / group poses without a second avatar. `[ADJUST]` → `[ANIMESH]` rezzes a posable dummy onto any empty seat of the current pose; position it through the HUD like a real partner. Click a dummy to change or remove it, even while adjusting.
- **Feature**: Eight standard dummy bodies included (Female / Male in S / M / L plus two Kabuki), auto-detected by name: drop any `[QS]dummy…` body into the furniture and it appears in the picker. The animeshconfig notecard is only needed for your own extra bodies.
- **Feature**: Animesh ships with the bundle: fresh installs, push installs and routine updates deliver the plugin and bodies automatically (creator builds only: finished customer furniture never receives it).
- **Feature**: `[FINALIZE]` strips the complete animesh kit (bodies, notecard, both scripts) off a finished furniture before you sell it.
- **Fix**: A repair update now refreshes the whole installation; previously it pushed only the missing pieces and silently skipped pending updates for everything already present (e.g. the HUD scripts).
- **Fix**: The installer no longer throws a script error when `[AV]sitA` is already deactivated (ItsNOTMine plate migration).

## Version 1.22

- **Base**: Foundation for the upcoming standalone Animesh plugin (solo setup of couples / group poses, no second avatar): 1.22 ships the QuickyHUD/QuickySitter integration base the plugin builds on, chiefly the hudproxy hook. The plugin itself is delivered separately later.
- **Fix**: Bundled QuickySitter update: the first sit after the furniture had been idle a while now plays the proper animation right away (in rare cases it could show the default pose until you re-sat).

## Version 1.21

- **Fix**: The HUD now works when QuickySitter is installed in a child prim, not only in the root/main prim.
