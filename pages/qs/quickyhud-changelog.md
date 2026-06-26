---
title: QuickyHUD Changelog
sidebar: home_sidebar
permalink: quickyhud-changelog.html
keywords: changelog, releases, fixes, features, quickyhud
toc: true
---

Customer-facing changes only — each entry is tagged **Fix** (bug fix), **Feature** (new), or **Base** (groundwork shipped ahead of a separate feature). Routine internal/technical changes aren't listed. Newest version on top.

## Version 1.22

- **Base** — Foundation for the upcoming standalone Animesh plugin (solo setup of couples / group poses, no second avatar): 1.22 ships the QuickyHUD/QuickySitter integration base the plugin builds on — chiefly the hudproxy hook. The plugin itself is delivered separately later.
- **Fix** — Bundled QuickySitter update: the first sit after the furniture had been idle a while now plays the proper animation right away (in rare cases it could show the default pose until you re-sat).

## Version 1.21

- **Fix** — The HUD now works when QuickySitter is installed in a child prim, not only in the root/main prim.
