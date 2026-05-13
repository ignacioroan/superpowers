# verifying-implementation — Design Spec

**Date:** 2026-05-13
**Status:** Draft

---

## Problem

When a feature is implemented, an agent verifying it against the original spec typically:

- Goes directly to one source (usually Figma or code) without reading all three
- Skips Jira comments that contain design decisions made after the original ticket was written
- Produces vague output ("looks good", "seems fine") with no actionable gaps
- Loses the output to the chat log — no durable record attached to the plan

The root failure is the absence of a structured, multi-source comparison process with a mandatory output location. Without it, agents do partial checks and produce unreliable results.

---

## Scope

**In scope:**
- Verifying a single feature implementation against Jira ACs, Figma anatomy/guidelines, and the Design System
- Stopping before starting if any of the three sources is missing — asking the user to provide it
- Producing a structured gap report (✅ / ⚠️ / ❌) appended to the active plan file
- Including consumer responsibility notes (items that depend on correct upstream data)
- Including agent/skill learnings (patterns observed during verification that should inform future work)

**Out of scope:**
- Verifying multiple features in a single run — one ticket at a time
- Acting on gaps found (verification produces a report; acting on gaps is a separate concern)
- Posting to Jira, Figma, or any external tool

---

## Prerequisites — the three required sources

Before any verification work starts, confirm all three are available:

| Source | What is needed |
|---|---|
| **Jira** | Ticket URL — ACs, description, and ALL comments |
| **Figma** | At minimum: anatomy node + guidelines node; branch nodes if design is in progress |
| **Design System (DS)** | Storybook URL or component source — variants, tokens, documented states |

If any source is missing → stop, list what's missing, ask the user to provide it before proceeding.

---

## Process

1. **Find the plan file** — search `docs/plans/`, `docs/`, and the repo root for a plan file matching the feature or ticket. If none found → inform the user and ask for instructions before continuing.
2. **Read Jira** — extract each AC line by line; note comments that override or refine the original spec; note linked Figma/DS refs.
3. **Read Figma** — for each relevant node: anatomy labels, states (default, out-of-stock/OOS, hover, empty, mobile), guidelines (max items, copy rules, responsive notes).
4. **Read DS** — confirm which components are used, their variants, token values for color/spacing referenced in Figma.
5. **Read the implementation** — grep and view source files; read tests to understand what's already covered.
6. **Compare systematically** — for each element from Jira ACs and Figma anatomy, produce a row in the gap table.
7. **Write the output section** — append to the plan file.

---

## Gap Table Format

For each element identified from Jira and Figma:

| Element | Expected (Jira / Figma) | Implemented | Status | Action |
|---|---|---|---|---|
| Tag position | Top-left of media | `Media.tsx:122` | ✅ | — |
| Max 3 features | Design guideline | Not enforced | ⚠️ | Consumer responsibility — document |
| OOS price color | `text-disabled` | `text-secondary` | ⚠️ | Known trade-off — DS token fails WCAG AA |
| Hover state | Overlay + CTA | Not implemented | ❌ | Needs implementation |

**Status definitions:**
- ✅ — matches spec exactly
- ⚠️ — partial, known trade-off, or consumer responsibility (correct by design, but requires correct upstream data or documented caveats)
- ❌ — missing or incorrect — actionable gap

---

## Output Section Format

Appended to the plan file:

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

Items that are implemented correctly but require correct data or usage from the consumer:
- ...

### Agent / skill learnings

Patterns or recurring gaps observed during this verification that should inform future agents or skill improvements:
- ...
```

---

## SKILL.md Design

**Frontmatter:**
```yaml
---
name: verifying-implementation
description: Use when a feature implementation is complete and needs to be checked against the original ticket, Figma design, and design system before closing or raising a PR
---
```

**Sections:**
- Overview (core principle: no verification without all three sources; no output without a plan file)
- Prerequisites table (Jira / Figma / DS) with stop-and-ask rule
- Process (numbered steps: find plan → read sources → read code → compare → write output)
- Gap table format (element / expected / implemented / status / action)
- Output section format (markdown template to append)
- Common Mistakes

**Tone and style:**
- Agnostic to any specific skill ecosystem — no references to sibling skills in this repo
- References to "execution skill" or "gap-fixing workflow" should remain generic
- Suitable to be moved to a different skill repository without modification

---

## Common Mistakes to Cover

| Mistake | Correction |
|---|---|
| Starting without all 3 sources | Stop. List what's missing. Ask. Never verify blind. |
| Skipping Jira comments | Comments often contain design decisions that override the original spec. Read all of them. |
| Reporting only ❌ gaps | ⚠️ items (trade-offs, consumer responsibility) are equally important to document |
| Skipping the plan file check | If there's no plan file, output to chat and ask — don't silently drop the report |
| Reading only anatomy, not guidelines | Guidelines contain max-item rules, copy constraints, and mobile behavior |
| Marking a trade-off as ✅ | If it differs from spec for any reason, it's ⚠️ — document why |

---

## Open Questions

None — design decisions resolved based on session context from 2026-05-13.
