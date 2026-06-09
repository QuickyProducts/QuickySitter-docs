---
title: Contributing
sidebar: home_sidebar
permalink: contributing.html
keywords: contributing, PR, pull request, workflow
toc: true
---

QuickySitter is open-source under MPL 2.0. Contributions are welcome via pull requests against [QuickyProducts/QuickySitter](https://github.com/QuickyProducts/QuickySitter).

This page covers practical conventions specific to QS. For LSL coding norms in general, the [Second Life LSL Portal](http://wiki.secondlife.com/wiki/LSL_Portal) is the canonical reference.

## Workflow

1. Fork the repo on GitHub, or create a feature branch if you have push access.
2. Make your changes, following the conventions below.
3. Bump touched scripts' versions by 0.00001 — all to the same locked number (see [Version Bump Convention](version-bump.html)).
4. Test in-world — at minimum, follow the relevant scenarios in [`qs/test/TESTPLAN.md`](https://github.com/QuickyProducts/QuickySitter/blob/master/qs/test/TESTPLAN.md).
5. Open a PR with a description of what changed and why.
6. Update `qs/PROTOCOL.md` and/or `qs/STORAGE.md` in the same PR if protocol behavior or state layout changes.

## Coding conventions

### LSL gotchas to follow

These are project-wide conventions worth re-stating because they've each caused regressions in the past:

- **`llParseString2List` over `llParseStringKeepNulls`.** KeepNulls keeps trailing empty fields, which several QS handlers misinterpret as valid (empty) data and crash on. Trim empties unless you have a clear reason not to.
- **No `llOwnerSay` in hot paths.** A passive `llOwnerSay` log on a script that runs in N sitter slots multiplies by N on furniture-heavy regions and floods the owner's chat. Use on-demand dialog reports or gate behind a `bDebug` flag.
- **Reserved identifiers.** `state` is a keyword. So are all type names (`integer`, `string`, `list`, `vector`, `rotation`, `float`). Use Hungarian prefixes (`sKey`, `iState`) or descriptive names (`lsd_key`).
- **LSL has no sequence point in `&&`.** `(var = f()) != "" && other(var)` reads a stale `var` in the second operand. Read, then check; don't assign in conditions.
- **LSD return-code literals.** `LINKSETDATA_MEMFULL` and friends are viewer-dependent constants; not portable. Use literal int values with an inline comment. Only `LINKSETDATA_OK` and `LINKSETDATA_RESET` are guaranteed.
- **No script-name inventory probes.** Don't add `llGetInventoryType("[QS]sitA")` checks. Use [QSALIVE](qsalive-discovery.html) (90096/90097) instead — names are not stable across forks.
- **AVpos `MENU` vs `TOMENU`.** `MENU` is a section marker only; `TOMENU` is the clickable button that opens the submenu. A top-level menu without a matching `TOMENU` doesn't render.

### Language for written output

Code, comments, file content, commit messages, PR descriptions — all English. The repo's working language is English regardless of where contributors live. Touch-as-you-migrate; no sweep PRs converting in-place comments.

### Comments

Default to writing **no** comments. The exception is hidden constraints, subtle invariants, workarounds for specific bugs, or behavior that would surprise a reader. Don't explain WHAT the code does — well-named identifiers handle that. Don't reference the current task ("used by X", "added for the Y flow", "handles issue #123") — that belongs in the PR description and rots as the code evolves.

A comment that would still be true and useful in 2 years is the bar.

### Update announcements in PRs

When a PR touches script versions, the PR description starts with the file + bump list:

```
**Update:** [QS]sitA 0.999 → 0.99901, [QS]boot 0.999 → 0.99901
```

This makes review queues easy to skim for "which scripts changed." Touched scripts share the same new locked number.

## Documentation

Documentation lives in two places:

1. **`qs/PROTOCOL.md`** and **`qs/STORAGE.md`** in the script repo — canonical for protocol and state details. Same PR as the code.
2. **This site** ([QuickyProducts/QuickySitter-docs](https://github.com/QuickyProducts/QuickySitter-docs)) — user-facing reference, derived from the in-repo design docs.

If your PR changes a link-message contract, update PROTOCOL.md in the same PR. Updating this docs site can come in a follow-up; the in-repo doc is what reviewers and other contributors read first.

## Licensing

By contributing you agree your contributions are licensed under MPL 2.0 (code) or CC-BY-SA 4.0 (documentation), matching the rest of the project. The boilerplate license header at the top of each `[QS]*.lsl` carries the AVsitter Contributors copyright notice — keep it intact when forking a stock script.

## See also

- [Repo Structure](repo-structure.html) — what lives where in the source tree.
- [Version Bump Convention](version-bump.html) — the +0.00001 locked-version rule.
- [Debug Flags](debug-flags.html) — how to instrument new code temporarily.
