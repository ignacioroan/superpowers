---
name: visual-verification
description: Use when UI work is implemented and you need to confirm the rendered output matches the design — captures a screenshot matrix across viewports and states from a running dev server / component explorer and compares against the reference, before claiming a UI change is done
---

# Visual Verification

## Overview

**A spec you read is not the output you ship. Render it and look.**

Reading a design source tells you what *should* render; it does not tell you what
*does*. This skill renders the implementation and captures a screenshot matrix
(viewports × states), compares it to the design reference and expected behavior,
and reports defects with root causes. Single responsibility: it renders and
compares; it does not do the full design-source cross-check (that is
`implementation-verifying`, which calls this skill).

**Reading CSS properties and reasoning about their defaults is not verification.**
The cascade — rules from other stylesheets, media queries, inherited values, and
real content — can override what any single file implies. A `display: flex` in
one file tells you nothing about `flex-wrap`, `flex-basis`, or breakpoint
overrides set elsewhere. Only rendering the actual page at the actual viewport
shows what ships.

**Announce at start:** "I'm using visual-verification to check the rendered UI."

## Process

1. **Pick a render surface** (tool-agnostic): a component-explorer story, an app
   route, or a harness, served by a running dev server. Confirm the server is up.
2. **Pick a capture tool** by what's available — the contract is "capture N
   viewports × M states to a known directory", not a specific tool. Any of: a
   preview/automation MCP, a headless-browser skill, or a connected browser.
3. **Viewport matrix:** at minimum mobile / tablet / desktop, plus every
   breakpoint the design defines.
4. **State matrix:** the meaningful states — empty, min, max, with/without each
   optional element, long-content, and edge/error states.
5. **Capture** the full matrix to a known directory.
6. **Compare** each shot to the design reference frame and to expected behavior.
   When layout looks wrong, **inspect computed styles** to find the root cause —
   see `failure-modes.md`. Do not guess.
7. **Report** findings: defect, viewport/state, suspected root cause, fix
   direction. Feed fixes back, re-capture, confirm fixed.

## Red flags (you are not done)

- You concluded "matches the design" without rendering it.
- You read the CSS, saw a property value, and inferred the rendered layout from
  that — without checking what other stylesheets, media queries, or cascade rules
  actually produce at that breakpoint.
- You captured one viewport / one state.
- You saw a layout defect and guessed the cause without inspecting computed styles.

## Reference

`failure-modes.md` — universal rendered-layout failure modes.
