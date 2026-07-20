---
title: QuickyHUD Changelog
sidebar: home_sidebar
permalink: quickyhud-changelog.html
keywords: changelog, releases, fixes, features, quickyhud
toc: true
---

Customer-facing changes only. Each entry is tagged **Fix** (bug fix), **Feature** (new), or **Base** (groundwork shipped ahead of a separate feature). Routine internal/technical changes aren't listed. Newest version on top.

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
