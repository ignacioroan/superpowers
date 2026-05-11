# iterating-on-implementation — Design Spec

**Date:** 2026-05-11  
**Status:** Draft  

---

## Problem

The current Superpowers workflow covers the full development lifecycle:
`brainstorming → spec → writing-plans → plan → execution → finishing-a-development-branch`

There is no skill for what happens **after** implementation, when the user or agent discovers:
- Bugs or edge cases not covered by the original plan
- Small functional adjustments after seeing the implementation in action
- Minor additions that stay within the scope of the existing plan

Today, agents face a gap: either they go through a full `brainstorming` session (overkill for small changes), or they improvise without structure (risky). The `receiving-code-review` skill handles code-review feedback, but that leaves uncovered the broader set of post-implementation iteration scenarios.

---

## Proposed Solution

A new skill: **`iterating-on-implementation`**

A lightweight orchestration skill that:
1. Loads and reads the existing spec and plan for context
2. Asks the minimum questions needed to understand the change
3. Validates the change fits within the original plan's scope
4. Appends a structured `## Iteration N` section to the existing plan
5. Delegates execution to existing skills (no reinventing)

---

## Scope Boundaries

### Use this skill when:
- Post-implementation fix, adjustment, or small addition
- The change affects only components already implemented in the current plan
- No new architectural decisions are required

### Do NOT use this skill — redirect instead:
| Situation | Redirect to |
|---|---|
| Change requires new components or architecture | `brainstorming` |
| Change motivated by code review feedback | `receiving-code-review` |
| Change requires a substantially new design | `brainstorming` |

---

## Process Flow

```
1. Locate context
   └─ Read existing spec + plan (ask user for paths if not obvious)

2. Understand the change
   └─ Ask minimum questions: what to change, success criteria, what NOT to touch

3. Scope check
   └─ Does this stay within the original plan's components?
      ├─ Yes → continue
      └─ No  → redirect to brainstorming, stop here

4. Write Iteration section
   └─ Append "## Iteration N: [title]" to the existing plan
      with Trigger, Scope, Goal, and task steps (same format as writing-plans)

5. Execute
   └─ Delegate to subagent-driven-development (recommended) or executing-plans

6. Verify
   └─ Invoke verification-before-completion before declaring done
```

---

## Iteration Section Format

The section appended to the plan follows the same conventions as `writing-plans`:

```markdown
---

## Iteration 1: Fix case-sensitivity in email auth

**Trigger:** Login fails when email contains uppercase letters
**Scope:** `src/auth/` — existing module, no new files
**Goal:** All email comparisons are case-insensitive; existing auth tests pass

### Task I1: Fix email normalization

**Files:**
- Modify: `src/auth/login.ts`
- Test: `tests/auth/login.test.ts`

- [ ] **Step 1: Write the failing test**

```typescript
it('accepts uppercase email', async () => {
  const result = await login('User@Example.com', 'password');
  expect(result.success).toBe(true);
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npm test -- auth/login`
Expected: FAIL

- [ ] **Step 3: Normalize email to lowercase on input**

```typescript
const normalizedEmail = email.toLowerCase();
```

- [ ] **Step 4: Run tests to verify they pass**

Run: `npm test -- auth/login`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add src/auth/login.ts tests/auth/login.test.ts
git commit -m "fix: normalize email to lowercase on login"
```
```

---

## Skills Orchestrated (no reinvention)

| Situation | Skill invoked |
|---|---|
| Change involves a bug | `systematic-debugging` first |
| Implementing the changes | `subagent-driven-development` (recommended) or `executing-plans` |
| Before declaring done | `verification-before-completion` |
| After completion (optional) | `requesting-code-review` |

---

## What This Skill Does NOT Do

- Does **not** create new specs — scope-expanding changes go to `brainstorming`
- Does **not** reimplement execution logic — delegates to existing skills
- Does **not** handle code review feedback — that's `receiving-code-review`
- Does **not** replace the `finishing-a-development-branch` flow if the iteration is substantial

---

## SKILL.md Design

**Frontmatter:**
```yaml
---
name: iterating-on-implementation
description: Use when post-implementation changes are needed — fixes, adjustments, or small additions within the scope of an already-implemented plan, without requiring a new spec or full brainstorming session
---
```

**Sections:**
- Overview (core principle, what it does and doesn't do)
- When to use / When NOT to use (scope check table)
- Process (numbered steps with decision points)
- Iteration section format (reference template)
- Skills invoked

---

## Open Questions

None — all design decisions resolved during brainstorming session.
