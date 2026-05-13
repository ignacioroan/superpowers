---
name: verifying-implementation
description: Use when a feature implementation is complete and needs to be checked against the original ticket, Figma design, and design system before closing or raising a PR
---

# Verifying Implementation

## Overview

Cross-check the delivered implementation against Jira ACs, Figma anatomy/guidelines, and DS component usage. Produces a structured gap report appended to the active plan file.

**Core principle:** No verification without all three sources. No output without a plan file or explicit user confirmation that none exists.

**Announce at start:** "I'm using verifying-implementation to check this feature."

## Prerequisites — stop and ask if any source is missing

Before doing anything, confirm all three are in context:

| Source | What you need |
|---|---|
| **Jira** | Ticket URL — read ACs, description, and ALL comments |
| **Figma** | At minimum: anatomy node + guidelines node. Branch nodes if applicable |
| **Design System (DS)** | Storybook URL or component source — variants, tokens, documented states |

If any source is missing → **stop, list what's missing, ask the user to provide it.**

## Process

1. **Find the plan file.** Search `docs/plans/`, `docs/`, and the repo root for a plan file matching the feature or ticket name. If none found → inform the user and ask for instructions before continuing.
2. **Read Jira.** Extract each AC line by line. Note comments that override or refine the original spec — comments take precedence over ACs when they conflict.
3. **Read Figma.** For each relevant node: anatomy labels, states (default, OOS, hover, empty, mobile), guidelines (max items, copy rules, responsive notes).
4. **Read DS.** Confirm which components are used, their variants, token values for color/spacing referenced in Figma.
5. **Read the implementation.** Grep and view source files. Read tests to understand what's already covered.
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
