---
title: QuickyHUD Changelog
sidebar: home_sidebar
permalink: quickyhud-changelog.html
keywords: changelog, releases, fixes, features, quickyhud
toc: true
---

Customer-facing changes only. Each entry is tagged **Fix** (bug fix), **Feature** (new), or **Base** (groundwork shipped ahead of a separate feature). Routine internal/technical changes aren't listed. Newest version on top.

## Version 1.25

*The creator product is now called **QuickySitter Pro** (formerly "QuickyHUD Creator"). The HUD itself keeps its name and is one tool in the kit among others. From this release the kit and the QuickySitter engine share one version number: QuickySitter jumps from 1.04 to 1.25 to meet the kit.*

- **Feature**: Showcase mode for the Animesh dummies (owner only): switch `[SHOWCASE]` on in the seat list and the staged dummies stay put when you get up, which turns a staged pose into a store display. They also survive a visitor sitting down or swapping seats — the avatar visibly shares the seat instead of the display being torn down. Note that updating or resetting the furniture clears a staged display, so re-stage after an update.
- **Feature**: Prop scale & worn fit: the new `[QS]propadjust` companion script goes into your prop and makes it resizable from the editor, or fittable directly on the body for attachment props, and QuickySitter's `[SAVE]` persists it. End users can fine-tune a rezzed prop by touch (±1/5/10 % menu, `[RESTORE]`).
- **Feature**: New creator tool `scripttime-probe`: it measures the script time of any object by UUID from a distance.
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
