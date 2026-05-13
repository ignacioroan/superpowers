# evaluating-pr-feedback Skill — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create the `evaluating-pr-feedback` skill — a fast triage layer that scores a single PR comment and either drafts a rejection reply or hands off to `iterating-on-implementation`.

**Architecture:** One new skill directory with a single `SKILL.md`. The skill is pure process documentation: it defines a scoring model (Gain × Plausibility × Cost inverse, threshold ≥ 7), produces a chat-only draft reply on the rejection path, and delegates to `iterating-on-implementation` on the action path. No execution logic is introduced here.

**Tech Stack:** Markdown only. Delegates to existing skills: `iterating-on-implementation`, `systematic-debugging` (if needed for bug triage).

**Spec:** `docs/specs/2026-05-12-evaluating-pr-feedback-design.md`

---

## Task 1: Write SKILL.md

**Files:**
- Create: `skills/evaluating-pr-feedback/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p skills/evaluating-pr-feedback
```

- [ ] **Step 2: Write SKILL.md**

Create `skills/evaluating-pr-feedback/SKILL.md` with the following content:

````markdown
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

### Cost inverse — how much effort to implement?
| Score | Criterion |
|---|---|
| 3 | Trivial — under 30 min, low risk, 1–2 files |
| 2 | Moderate — a few hours or multiple files |
| 1 | Complex — risky refactor, cross-cutting, or requires new spec |

## Decision

**Sum ≥ 7 → Act.** Invoke `iterating-on-implementation`.

**Sum < 7 → Don't act.** Draft a rejection reply and output it to chat. Do NOT post to GitHub.

Show the score breakdown before the reply:
```
Gain: N | Plausibility: N | Cost: N | Total: N/9
```

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
````

- [ ] **Step 3: Verify frontmatter constraints**

Check:
- `name` uses only letters, numbers, hyphens ✓
- `description` starts with "Use when..." ✓
- `description` does NOT summarize workflow — only triggering conditions ✓
- Total frontmatter under 1024 characters

Run:
```bash
head -5 skills/evaluating-pr-feedback/SKILL.md
wc -w skills/evaluating-pr-feedback/SKILL.md
```

Expected: frontmatter looks clean, word count under 400.

---

## Task 2: Validate the skill with two test runs

Run two scenarios as subagents WITH the skill in context. No test files are persisted — this is a one-shot validation pass.

**Files:** none (validation only, no artifacts to keep)

- [ ] **Step 1: Run scenario A — comment that should be rejected (low score)**

Dispatch a fresh general-purpose subagent with this prompt (inline the full SKILL.md content so the subagent "has" the skill):

```
You have the following skill available:

[PASTE FULL CONTENT OF skills/evaluating-pr-feedback/SKILL.md]

---

A PR reviewer left this comment on your implementation:

Comment: "You should handle the case where the server returns a 418 status code (I'm a teapot). Add a specific error message for it."

Code context: `src/api/client.ts`, lines 45-67 — a standard HTTP error handler that maps 4xx/5xx codes to user-facing messages.

Use the skill to evaluate this comment. Show your scoring and output the result.
```

Expected: agent scores low on Plausibility (418 is hypothetical/never sent by real APIs), outputs score breakdown, drafts rejection reply. Does NOT invoke iterating-on-implementation.

Verify: Did the agent announce the skill? Did it show the score breakdown? Did it draft a reply in English? Did it correctly reject rather than implement?

- [ ] **Step 2: Run scenario B — comment that should be acted on (high score)**

Dispatch another subagent with this prompt:

```
You have the following skill available:

[PASTE FULL CONTENT OF skills/evaluating-pr-feedback/SKILL.md]

---

A PR reviewer left this comment on your implementation:

Comment: "The login function doesn't sanitize the email input before passing it to the database query. This could allow SQL injection."

Code context: `src/auth/login.ts`, lines 12-28 — a function that takes user-provided email and password and queries the database.

Use the skill to evaluate this comment. Show your scoring and output the result.
```

Expected: agent scores high on Gain (security = 3) and Plausibility (SQL injection is a real risk = 3), invokes `iterating-on-implementation`. Does NOT draft a rejection reply.

Verify: Did the agent correctly score ≥ 7? Did it invoke `iterating-on-implementation`?

- [ ] **Step 3: Fix any loopholes found**

If either subagent failed to follow the skill correctly (wrong decision, skipped score breakdown, posted to GitHub instead of chat), identify the gap in SKILL.md and fix it before proceeding.

If both passed: no changes needed.

---

## Task 3: Commit

**Files:** all of the above

- [ ] **Step 1: Confirm clean state**

```bash
git status
```

Expected: `skills/evaluating-pr-feedback/SKILL.md` as a new untracked file (plus spec if not yet committed).

- [ ] **Step 2: Commit spec and skill together**

```bash
git add docs/specs/2026-05-12-evaluating-pr-feedback-design.md
git add skills/evaluating-pr-feedback/SKILL.md
git commit -m "feat: add evaluating-pr-feedback skill"
```

- [ ] **Step 3: Verify commit**

```bash
git --no-pager log --oneline -3
git --no-pager show --stat HEAD
```

Expected: commit appears with both spec and skill file.
