---
title: Creator Utilities
sidebar: home_sidebar
permalink: creator-utilities.html
keywords: utilities, tools, shifter, AVpos-shifter, generator, MLP, migration
toc: true
---

Utilities are scripts you drop into a prim while building and remove before you hand the build out. They are not part of the running furniture: no sitter depends on them, and none of them shows up in any menu.

QuickySitter forks one of the AVsitter utilities and ships it as `[QS]AVpos-shifter`. The rest of the AVsitter Utilities box still works, with one important exception covered below.

## The one rule that applies to all of them

**On QuickySitter the AVpos notecard is a seed, not the live store.**

Positions you save with `[SAVE]`, and poses you add with `[NEW]`, are written into the furniture's own storage. The notecard is never rewritten. It is read once, when its asset key changes, and everything in storage is replaced from it at that moment.

Two consequences, and both of them bite:

1. Any utility that reads the AVpos notecard sees whatever was in it the last time you saved it, not what the furniture is actually playing.
2. Saving an edited AVpos notecard **discards every position you saved through the menus** and re-seeds from the notecard text.

So before you point any notecard-reading tool at a piece you have been adjusting, bring the notecard up to date:

1. `[ADJUST]` > `[HELPER]` > `[DUMP]`
2. Paste the dump into the `AVpos` notecard and save it.
3. Now run the tool.

Skipping step 1 and 2 is the single most common way to lose a build's worth of adjustment work. See [Adjustment Workflow](adjustment-workflow.html) for where the data actually lives.

## [QS]AVpos-shifter

Moves every pose and prop position in an AVpos notecard at once. Owner only.

| Command | What it does |
|---------|--------------|
| `/5 <0,0,1.5>` | Shifts all positions by that offset. |
| `/6 <0,0,180>` | Rotates all positions by those degrees, about the prim center. |
| `/5 <0,0,1.5>,<0,0,180>` | Both at once. Note the comma between the two vectors. |
| touch a child prim | Rebases the whole notecard onto that prim, keeping the poses where they are in the world. Useful when you relink furniture and the poses need to follow a different prim. |

The converted notecard comes out twice: printed in local chat between two cut markers, and uploaded to a web page whose link is given at the end. Paste the result back into `AVpos` and save it.

Drop the script into the prim that holds the `AVpos` notecard. It does **not** have to be the root prim, which matters on QuickySitter builds that keep their scripts in a child prim.

### What is different from the AVsitter original

- **The settings link works.** The original posts to `avsitter.com`, which no longer accepts QuickySitter output. The fork posts to the QuickySitter dump service, the same one `[DUMP]` uses. Append `&raw=1` to the link if you want the byte-exact stream instead of the grouped layout.
- **It warns you when the notecard may be stale.** Whenever it detects QuickySitter storage in the linkset it repeats the DUMP-first rule above before converting.
- **It does not delete itself after a run.** The original removed itself every single time, so shifting twice cost two copies and a run that went wrong cost one for nothing. The fork stays until you finalize the build with `/5 cleanup`, which removes it along with `[QS]adjuster` and the other build tools. If you used it in a prim with no `[QS]adjuster` in it, delete it by hand.
- **It stays quiet on chat that is not meant for it.** Channel 5 is shared with the `[QS]adjuster` commands and with the `[AV]helper` stick. The original answered all of them with "You didn't enter a vector!".
- **It never deletes itself over placement.** The original removed itself when it found no `AVpos` beside it or when it was not in the root prim.

## AVsitter utilities that still work

`AVpos-generator` and `Anim-perm-checker` only walk the prim's inventory, and `MLP-converter` only reads MLP notecards. None of them touches AVpos data, so all three behave exactly as they do on stock AVsitter.

## Missing-anim-finder: read this first

`Missing-anim-finder` decides which animations are unused by reading the AVpos notecard alone, then offers to **delete** them.

Poses you added with `[NEW]` never reach the notecard. On a piece built through the menus, their animations look unused, and one click on YES deletes them from the prim.

Either run `[HELPER]` `[DUMP]` and refresh the notecard first, as described above, or answer NO to the delete prompt and go through its list by hand. The "used in the notecard but not found in inventory" half of its report is safe either way.

## Noob-detector

Posts to the same retired `avsitter.com` endpoint the original shifter used, so its web link is dead. Nothing in the QuickySitter workflow needs it.

## See also

- [Adjustment Workflow](adjustment-workflow.html): where saved positions actually live, and what `[DUMP]` produces.
- [Chat Commands](chat-commands.html): the other `/5` commands, including `/5 cleanup`.
- [Migration from AVsitter](migration.html): what carries over from an existing AVsitter build.
- [AVsitter Utilities](avsitter2_utilities.html): the upstream documentation for the unforked tools.
