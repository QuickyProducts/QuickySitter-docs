---
title:
sidebar: home_sidebar
permalink: index.html
toc: false
---

# QuickySitter&trade;

Welcome to the documentation for QuickySitter&trade; — a furniture pose system for Second Life<sup>&reg;</sup>, written in LSL and built as a **fork of [AVsitter&trade; 2](https://github.com/AVsitter/AVsitter)**.

## What is QuickySitter

QuickySitter keeps full compatibility with stock AVsitter 2 (notecard format, MENU/POSE/PROP syntax, plugin LinkMsg contracts) and adds:

- **LinkSet Data storage** — pose defaults and channel settings live in LSD instead of script memory, so complex furniture stays stable past Mono's 64 KB cap.
- **HUD addon API** — QuickyHUD attaches as a seamless adjustment addon over the standard LinkMsg surface; removable at any time without side effects.
- **SYNC re-sync trigger** — LinkMsg `90271` restarts every sitter's main loop in the same Sim frame so couple poses re-phase on demand.
- **Module discovery via presence protocol** — plugins announce themselves on `90096`/`90097` instead of script-name inventory probes; scripts can be renamed without breaking third-party plugins.
- **Workload distribution** — responsibilities are split across more focused scripts to keep heap pressure low.

## Where to start

- **[Getting Started](/getting-started.html)** — install, basic setup, migration from AVsitter.
- **[Core System](/core-system.html)** — AVpos format, sit targets, menu structure, the adjustment workflow.
- **[QuickySitter Extensions](/qs-extensions.html)** — what QS adds on top of AVsitter (HUD, Re-Sync, LSD, QSALIVE).
- **[Reference](/reference.html)** — link message numbers, LSD keys, compatibility matrix.

## Source & License

QuickySitter LSL scripts are available on [GitHub]({{ site.script_github }}) under the [Mozilla Public License 2.0](https://www.mozilla.org/en-US/MPL/2.0/).

This documentation site re-uses portions of the [AVsitter documentation](https://avsitter.github.io) by Avcode Technologies under the [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/). QuickySitter additions are also CC-BY-SA 4.0.

QuickySitter&trade; is not affiliated with or sponsored by Linden Research or the AVsitter&trade; project.
