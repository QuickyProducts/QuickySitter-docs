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

- **LinkSet Data storage** — pose data, channel settings and the prop database live in LSD instead of script memory, so complex furniture stays stable past Mono's 64 KB cap. The practical limit is the linkset's shared **128 KiB LSD pool** (poses, props, settings and personal offsets all draw from it): room for roughly **1,700 poses** with typical entry sizes — and the pool is per furniture, not per slot, so a multi-sitter holds far more than stock AVsitter's ~200 poses per script-heap ever could.
- **HUD addon API** — QuickyHUD attaches as a seamless adjustment addon over the standard LinkMsg surface; removable at any time without side effects.
- **SYNC re-sync trigger** — LinkMsg `90271` restarts every sitter's main loop in the same Sim frame so couple poses re-phase on demand.
- **Module discovery via presence protocol** — plugins announce themselves on `90096`/`90097` instead of script-name inventory probes; scripts can be renamed without breaking third-party plugins.
- **Workload distribution** — responsibilities are split across more focused scripts to keep heap pressure low.

## Where to start

- **[Getting Started]({{ site.baseurl }}/getting-started.html)** — install, basic setup, migration from AVsitter.
- **[AVpos Reference]({{ site.baseurl }}/avpos-reference.html)** — AVpos format, sit targets, menu structure, the adjustment workflow.
- **[QSALIVE Discovery]({{ site.baseurl }}/qsalive-discovery.html)** — what QS adds on top of AVsitter (HUD, Re-Sync, LSD, presence protocol).
- **[LinkMessage Numbers]({{ site.baseurl }}/linkmessage-numbers.html)** — link message numbers, LSD keys, compatibility matrix.

## Source & License

QuickySitter LSL scripts are available on [GitHub]({{ site.script_github }}) under the [Mozilla Public License 2.0](https://www.mozilla.org/en-US/MPL/2.0/). See the [Changelog]({{ site.script_github }}/blob/master/CHANGELOG.md) for customer-facing changes.

This documentation site re-uses portions of the [AVsitter documentation](https://avsitter.github.io) by Avcode Technologies under the [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/). QuickySitter additions are also CC-BY-SA 4.0.

QuickySitter&trade; is not affiliated with or sponsored by Linden Research or the AVsitter&trade; project.
