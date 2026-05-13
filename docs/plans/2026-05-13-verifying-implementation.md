# verifying-implementation Implementation Plan

> **For agentic workers:** Steps use checkbox (`- [ ]`) syntax for tracking. Follow RED-GREEN-REFACTOR: run baseline scenario first (Task 1), then write the skill (Task 2), then validate (Task 3).

**Goal:** Create a `verifying-implementation` skill that teaches agents to cross-check a feature against Jira ACs, Figma anatomy/guidelines, and Design System — producing a structured gap report appended to the active plan file.

**Architecture:** Single `SKILL.md` file following the writing-skills TDD cycle. No code, no test files. "Tests" are subagent pressure scenarios run with and without the skill. Skill is ecosystem-agnostic — no references to other skills by name.

**Tech Stack:** Markdown only.

---

### Task 1 (RED): Baseline scenario — agent without skill

Run a subagent WITHOUT the skill and document what it does naturally when asked to verify a feature implementation.

**Files:**
- No files to create. Observation only.

- [ ] **Step 1: Dispatch baseline subagent**

Dispatch a `general-purpose` subagent with this exact prompt (do NOT mention the skill):

```
You are an agent helping a developer verify their implementation is complete.

Here is a fictional feature context to verify:

---
Jira ticket WEB-412: "Product card — media tag"

Acceptance criteria:
1. A tag (e.g. "New", "Sale") appears in the top-left corner of the product card media
2. Tag is only shown when a `tag` prop is provided
3. Tag uses the DS `Tag` component with `variant="promo"`
4. Out-of-stock (OOS) state: product card shows a greyed overlay; tag is hidden
5. Hover state: show a CTA button overlaid on the media

Jira comments:
- Designer (2026-04-10): "Hover CTA should only appear on desktop — no hover on mobile"
- PM (2026-04-11): "OOS overlay confirmed — but tag should remain visible even in OOS, overriding AC4"

Figma anatomy (described):
- Media area with tag positioned top-left
- OOS state: grey overlay on media, tag still visible
- Hover state: CTA "Add to bag" button centered over media, desktop only
- Guideline note: max 1 tag per card

Design System:
- `Tag` component supports variants: `promo`, `info`, `sale`
- `promo` uses color token `color-promo` (#FF4800), white text

Implementation (described):
- `ProductCard.tsx` renders `<Tag variant="promo">` in top-left — line 44
- Tag is hidden in OOS state (overlay hides it) — line 67
- Hover state not implemented
- No max-tag enforcement (no validation)
- DS `Tag` used with correct variant
---

Please verify this implementation against the spec. Tell me what's correct, what's missing, and what needs to be fixed.
```

- [ ] **Step 2: Document baseline failures**

After the subagent responds, record what it got wrong. Expected failure patterns:
- Skipped Jira comments — missed that PM overrode AC4 (tag should be visible in OOS, not hidden)
- Missed the desktop-only constraint on hover state
- Produced vague output ("hover state is missing") with no gap table structure
- No ✅/⚠️/❌ classification
- No consumer responsibility or learnings sections
- No mention of appending output to a plan file

Write the observed failures here before proceeding to Task 2:

```
Observed baseline failures:
[fill in after running subagent]
```

---

### Task 2 (GREEN): Create SKILL.md

**Files:**
- Create: `skills/verifying-implementation/SKILL.md`

- [ ] **Step 1: Create directory**

```bash
mkdir -p /path/to/repo/skills/verifying-implementation
```

- [ ] **Step 2: Write SKILL.md**

Create `skills/verifying-implementation/SKILL.md` with exactly this content:

````markdown
---
name: verifying-implementation
description: Use when a feature implementation is complete and needs to be checked against the original ticket, Figma design, and design system before closing or raising a PR
---

# Verifying Implementation

## Overview

Cross-check the delivered implementation against Jira ACs, Figma anatomy/guidelines, and DS component usage. Produces a structured gap report appended to the active plan file.

**Core principle:** No verification without all three sources. No output without a plan file.

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
6. **Compare systematically.** For each element from Jira ACs and Figma anatomy, produce a row in the gap table.
7. **Write the output section.** Append to the plan file using the format below.

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

Patterns observed during verification that should inform future agents or skill improvements:
- ...
```

## Common Mistakes

| Mistake | Correction |
|---|---|
| Starting without all 3 sources | Stop. List what's missing. Ask. Never verify blind. |
| Skipping Jira comments | Comments override ACs when they conflict. Read all of them. |
| Reporting only ❌ gaps | ⚠️ items (trade-offs, consumer responsibility) are equally important to document |
| Skipping the plan file check | If there's no plan, output to chat and ask — don't silently drop the report |
| Reading only anatomy, not guidelines | Guidelines contain max-item rules, copy constraints, and mobile behavior |
| Marking a trade-off as ✅ | If it differs from spec for any reason, it's ⚠️ — document why |
````

- [ ] **Step 3: Verify file created correctly**

```bash
head -5 skills/verifying-implementation/SKILL.md
wc -w skills/verifying-implementation/SKILL.md
```

Expected: frontmatter present, word count under 500.

---

### Task 3 (REFACTOR/validate): Agent WITH skill

Run the same scenario with the skill available and verify the baseline failures are now addressed.

**Files:**
- No files to create. Observation only.

- [ ] **Step 1: Dispatch validation subagent**

Dispatch a `general-purpose` subagent with this prompt:

```
You have access to the verifying-implementation skill at:
skills/verifying-implementation/SKILL.md

Read and follow it exactly.

---
Jira ticket WEB-412: "Product card — media tag"

Acceptance criteria:
1. A tag (e.g. "New", "Sale") appears in the top-left corner of the product card media
2. Tag is only shown when a `tag` prop is provided
3. Tag uses the DS `Tag` component with `variant="promo"`
4. Out-of-stock (OOS) state: product card shows a greyed overlay; tag is hidden
5. Hover state: show a CTA button overlaid on the media

Jira comments:
- Designer (2026-04-10): "Hover CTA should only appear on desktop — no hover on mobile"
- PM (2026-04-11): "OOS overlay confirmed — but tag should remain visible even in OOS, overriding AC4"

Figma anatomy (described):
- Media area with tag positioned top-left
- OOS state: grey overlay on media, tag still visible
- Hover state: CTA "Add to bag" button centered over media, desktop only
- Guideline note: max 1 tag per card

Design System:
- `Tag` component supports variants: `promo`, `info`, `sale`
- `promo` uses color token `color-promo` (#FF4800), white text

Implementation (described):
- `ProductCard.tsx` renders `<Tag variant="promo">` in top-left — line 44
- Tag is hidden in OOS state (overlay hides it) — line 67
- Hover state not implemented
- No max-tag enforcement (no validation)
- DS `Tag` used with correct variant

Note: there is no plan file in this scenario. Handle this as the skill instructs.
---

Verify this implementation against the spec.
```

- [ ] **Step 2: Verify the skill addressed the baseline failures**

Check that the subagent now:
- Read Jira comments and caught the PM override (AC4 overridden — tag must be visible in OOS) → ❌
- Caught the desktop-only constraint on hover → ❌ with mobile caveat
- Produced a ✅/⚠️/❌ gap table
- Included consumer responsibility (max 1 tag)
- Included agent/skill learnings section
- Handled the missing plan file gracefully (asked for instructions)

If any baseline failure reappears → add an explicit counter to Common Mistakes and re-run.

- [ ] **Step 3: Commit spec + skill**

```bash
git add docs/specs/2026-05-13-verifying-implementation-design.md \
        skills/verifying-implementation/SKILL.md
git commit -m "feat: add verifying-implementation skill"
```
