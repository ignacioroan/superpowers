---
name: implementation-verifying
description: Use when a feature implementation is complete and needs to be checked against the original ticket, Figma design, and design system before closing or raising a PR
---

# Verifying Implementation

## Overview

Cross-check the delivered implementation against Jira ACs, Figma anatomy/guidelines, and DS component usage. Produces a structured gap report appended to the active plan file.

**Core principle:** No verification without all three sources. No output without a plan file or explicit user confirmation that none exists.

**Announce at start:** "I'm using implementation-verifying to check this feature."

## Prerequisites — stop and ask if any source is missing

Before doing anything, confirm all three are in context:

| Source | What you need |
|---|---|
| **Jira** | Ticket URL — read ACs, description, and ALL comments |
| **Figma** | At minimum: anatomy node + guidelines node. Branch nodes if applicable |
| **Design System (DS)** | Storybook URL or component source — variants, tokens, documented states |
| **Running UI** | A dev server / component explorer must be running so the implementation can be rendered |

If any source is missing → **stop, list what's missing, ask the user to provide it.**

## Process

1. **Find the plan file.** Search `docs/plans/`, `docs/`, and the repo root for a plan file matching the feature or ticket name. If none found → inform the user and ask for instructions before continuing.
2. **Read Jira.** Extract each AC line by line. Note comments that override or refine the original spec — comments take precedence over ACs when they conflict.
3. **Read Figma.** For each relevant node: anatomy labels, states (default, OOS, hover, empty, mobile), guidelines (max items, copy rules, responsive notes).
4. **Read DS.** Confirm which components are used, their variants, token values for color/spacing referenced in Figma.
5. **Read the implementation.** Grep and view source files. Read tests to understand what's already covered.
5.5. **Render and verify visually.** Run the `visual-verification` skill across viewports and states. Rows in the gap table MUST include findings from the rendered comparison, not only from reading design nodes. (A read-only check is how rendered layout defects slip through.)
6. **Compare systematically.** For each element from Jira ACs, Figma anatomy, and Figma guidelines, produce a row in the gap table.
7. **Write the output section.** Append to the plan file using the format below. If the user confirmed there is no plan file (Step 1), output the section to chat instead.

## Gap Table

For each element identified from Jira and Figma:

| Element | Expected (Jira / Figma) | Implemented | Status | Action |
|---|---|---|---|---|
| Tag position | Top-left of media | `Media.tsx:44` | ✅ | — |
| OOS tag visibility | Visible (PM override, comment 2026-04-11) | Hidden by overlay | ❌ | Fix: tag must render above OOS overlay |
| Hover CTA | Desktop only (Designer comment 2026-04-10) | Not implemented | ❌ | Needs implementation; mobile excluded |
| Max 1 tag | Figma guideline | Not enforced | ⚠️ | Consumer responsibility — document |

**Status:**
- ✅ — matches spec exactly
- ⚠️ — partial, known trade-off, or consumer responsibility (correct by design — document why)
- ❌ — missing or incorrect — actionable gap

- **Reuse-before-rebuild:** for every hand-rolled common primitive (surface/card, button, link, list/carousel, image, input), a row confirming no component-library component covers it. The concrete catalog is the consuming project's agent guide; the check is mandatory here.
- **Layout primitive:** a row confirming the block/page container uses the project's shared layout primitive (grid/gutters/rhythm), not a bare flex element.

## Accessibility checklist (run before sign-off)

- **No empty semantic wrappers.** Every `<figure>`, `<figcaption>`, `<section>`, or landmark either has content or is conditionally rendered.
- **Decorative images carry empty alt at the component level** (driven by a variant/role prop), not only in fixtures; meaningful images carry real alt.
- **Controls have accessible names**; heading hierarchy is correct and the heading element is configurable; regions are labelled (`aria-labelledby`/`aria-label`) where the pattern needs it.

## Output Section Format

Append to the plan file:

```markdown
---

## Verification: [Feature / Ticket]

**Date:** YYYY-MM-DD
**Sources:** [Jira URL] · [Figma node URLs] · [DS Storybook URL]

### Gap report

| Element | Expected | Implemented | Status | Action |
|---|---|---|---|---|
...

### Consumer responsibility notes

Items implemented correctly but requiring correct data or usage from the consumer:
- ...

### Agent / skill learnings

Patterns observed during verification that should inform future agents or skill improvements.
Only populate if something was genuinely surprising — e.g. a Jira comment that overrode a core AC, a DS token that failed WCAG, a mobile constraint buried in guidelines.
- ...
```

## Common Mistakes

| Mistake | Correction |
|---|---|
| Starting without all 3 sources | Stop. List what's missing. Ask. Never verify blind. |
| Skipping Jira comments | Comments override ACs when they conflict. Read all of them. |
| Reporting only ❌ gaps | ⚠️ items (trade-offs, consumer responsibility) are equally important to document |
| Marking max-item constraints as ❌ | Unenforced design guidelines are ⚠️ consumer responsibility — not bugs |
| Skipping the plan file check | Stop and ask for instructions first (Step 1). Only output to chat after the user confirms none exists. |
| Reading only anatomy, not guidelines | Guidelines contain max-item rules, copy constraints, and mobile behavior |
| Marking a trade-off as ✅ | If it differs from spec for any reason, it's ⚠️ — document why |
