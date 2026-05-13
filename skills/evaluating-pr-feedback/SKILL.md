---
name: evaluating-pr-feedback
description: Use when receiving a PR comment that needs a triage decision — whether to act on it or dismiss it with a reply
---

# Evaluating PR Feedback

## Overview

Not every PR comment deserves action. This skill scores a comment on three dimensions and decides in one of two directions: draft a rejection reply (output to chat only), or hand off to `iterating-on-implementation`.

**Core principle:** Score first, then decide. Never implement blindly.

**Announce at start:** "I'm using evaluating-pr-feedback to triage this comment."

## Input

- **Comment text** (required) — the PR comment to evaluate
- **Code context** (optional) — file path + line range referenced by the comment. Read that code before scoring if provided.

## Scoring Model

Score the comment on three dimensions (1–3 each). Sum the scores.

### Gain — what improves if acted on?
| Score | Criterion |
|---|---|
| 3 | Correctness, security, or data integrity issue |
| 2 | UX, performance, or maintainability improvement |
| 1 | Style, cosmetic, or preference-based |

### Plausibility — does this scenario actually occur in practice?
| Score | Criterion |
|---|---|
| 3 | Common real-world case, clearly plausible |
| 2 | Edge case — possible but uncommon |
| 1 | Hypothetical or theoretical — unlikely to occur in this codebase |

### Cost inverse — how easy/cheap to implement? (3 = cheapest, 1 = most expensive)
| Score | Criterion |
|---|---|
| 3 | Trivial — under 30 min, low risk, 1–2 files |
| 2 | Moderate — a few hours or multiple files |
| 1 | Complex — risky refactor, cross-cutting, or requires new spec |

## Decision

Always show the score breakdown before acting on the decision:
```
Gain: N | Plausibility: N | Cost: N | Total: N/9
```

**Sum ≥ 7 → Worth acting.** Present the score breakdown and your conclusion to the human, then ask whether to proceed. Only invoke `iterating-on-implementation` if the human confirms.

**Sum < 7 → Don't act.** Draft a rejection reply and output it to chat. Do NOT post to GitHub.

## Rejection Reply Format

- Language: English
- Tone: direct, no apologies, no performative softening
- Length: 1–3 sentences
- Content: why it isn't worth acting on

**Example (low plausibility):**
> "Leaving this as-is. The scenario described requires [X] which doesn't occur in this codebase's usage patterns. The change would add complexity without meaningful benefit."

**Example (low gain + high cost):**
> "Not acting on this one. The improvement is stylistic and the refactor touches multiple files. The risk/reward ratio doesn't justify it."

## What This Skill Does NOT Do

- Does **not** post to GitHub — output goes to chat only; human decides whether to copy/paste
- Does **not** handle multiple comments in batch — evaluate one at a time
- Does **not** replace `receiving-code-review` — that skill is for full review sessions with multiple items and technical pushback
