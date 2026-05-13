# evaluating-pr-feedback — Design Spec

**Date:** 2026-05-12
**Status:** Draft

---

## Problem

When a PR receives a comment, the agent currently has two options: implement it (via `iterating-on-implementation`) or evaluate it technically (via `receiving-code-review`). Neither skill focuses on the triage decision itself: *is this comment worth acting on at all?*

Without an explicit triage step, agents either implement every comment blindly or engage in lengthy technical back-and-forth. What's missing is a fast, structured decision: score the comment, decide in one of two directions, and take the appropriate action cleanly.

---

## Scope

**In scope:**
- Evaluating a single PR comment (inline or review comment)
- Optionally reading the referenced code (file + lines) for context
- Scoring the comment on three dimensions to reach a decision
- If not worth acting: drafting a concise English reply for the human to post
- If worth acting: handing off to `iterating-on-implementation`

**Out of scope:**
- Posting to GitHub directly (the skill outputs to chat only)
- Handling multiple comments in batch
- Replacing `receiving-code-review` (different scope — that skill handles full review sessions with technical rigor)

---

## Relationship to Existing Skills

| Skill | When to use |
|---|---|
| `evaluating-pr-feedback` (new) | Triage a single comment: worth acting or not? |
| `receiving-code-review` | Receiving a full code review with multiple items, technical pushback, implementation order |
| `iterating-on-implementation` | Implementing in-scope changes after triage says "act" |

---

## Decision Criterion — Scoring Model

Each comment is scored on three dimensions (1–3 each):

### Gain (what improves if acted on)
| Score | Criterion |
|---|---|
| 3 | Correctness, security, or data integrity issue |
| 2 | UX, performance, or maintainability improvement |
| 1 | Style, cosmetic, or preference-based |

### Plausibility (does this scenario actually occur in practice?)
| Score | Criterion |
|---|---|
| 3 | Common real-world case, clearly plausible |
| 2 | Edge case — possible but uncommon |
| 1 | Hypothetical or theoretical — unlikely to occur in practice |

### Cost inverse (how much effort to implement? — higher cost = lower score)
| Score | Criterion |
|---|---|
| 3 | Trivial — under 30 minutes, low risk |
| 2 | Moderate — a few hours or multiple files |
| 1 | Complex — risky refactor, cross-cutting change, or requires spec |

### Decision threshold
- **Sum ≥ 7** → Act → invoke `iterating-on-implementation`
- **Sum < 7** → Don't act → draft rejection reply

---

## Process

```
Input:
  - PR comment text (required)
  - Code context: file path + line range (optional)

1. Read the comment and code context if provided
2. Score on Gain, Plausibility, Cost inverse (each 1–3)
3. Sum the scores:
   ≥ 7 → invoke iterating-on-implementation
   < 7 → draft rejection reply, output to chat

Output (rejection path):
  - Score breakdown (Gain: N, Plausibility: N, Cost: N, Total: N/9)
  - Draft reply in English: direct, brief, no apologies, reasoning included
  - Human copies/pastes — skill does NOT post to GitHub
```

---

## Rejection Reply Format

- Language: English
- Tone: direct, no apologies, no performative softening
- Length: 1–3 sentences
- Content: why this isn't worth acting on (plausibility, cost, or low gain)

**Example (low plausibility):**
> "Leaving this as-is. The scenario described requires [X condition] which doesn't occur in this codebase's usage patterns. The change would add complexity without meaningful benefit."

**Example (low gain, high cost):**
> "Not acting on this one. The improvement is stylistic and the refactor touches multiple files. The risk/reward ratio doesn't justify it."

---

## SKILL.md Design

**Frontmatter:**
```yaml
---
name: evaluating-pr-feedback
description: Use when receiving a PR comment that needs a triage decision — whether to act on it or dismiss it with a reply
---
```

**Sections:**
- Overview (core principle: score first, then decide)
- Input format (comment text + optional code context)
- Scoring table (Gain / Plausibility / Cost inverse)
- Decision threshold
- Rejection reply format and examples
- Handoff to `iterating-on-implementation`
- What this skill does NOT do (no GitHub posting, no multi-comment batch)

---

## Open Questions

None — all design decisions resolved during brainstorming session.
