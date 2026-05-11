# Pressure Test Results (with skill)

## Scenario 1: Bug fix (with skill)
1. Announce: **“I’m using iterating-on-implementation for an in-scope iteration.”** Then open `docs/plans/2026-05-01-user-auth.md`, confirm this is still inside the existing auth plan, and state scope: login/auth parsing + auth tests only; out of scope: new auth architecture/UI.

2. Use **systematic-debugging** first: reproduce login failure with a password like `P@ss!word#1`, capture the exact failing path, and trace request input → auth handler → password comparison to identify the root cause before changing code.

3. Append **`## Iteration 1: Special-character password login`** to the plan with Trigger, specific in-scope files/tests, Goal, and a task requiring a failing test first.

4. Execute via **subagent-driven-development**: implementer writes the failing test, verifies it fails, makes the minimal fix, reruns auth tests, then finishes with verification-before-completion.

Followed skill process? Yes — it announced the skill, opened the plan first, did an explicit scope check, used `systematic-debugging` before fixing an unclear bug, wrote an `Iteration 1` section, and delegated execution with final verification.
New rationalizations or loopholes found: None.

## Scenario 2: Small addition (with skill)
1. Announce: **“I’m using iterating-on-implementation for an in-scope iteration.”** Then open `docs/plans/2026-04-15-file-upload.md` and confirm this request stays in scope: only the existing upload component, no new components, no broader UX redesign.

2. Append a new section to `docs/plans/2026-04-15-file-upload.md`:
   **`## Iteration 1: Upload progress percentage`**
   with:
   - **Trigger:** user wants a simple upload progress bar
   - **Scope:** existing upload component only
   - **Goal:** show live upload percentage during upload

3. In that Iteration 1 section, add the concrete task checklist:
   - write a failing test for percentage/progress display
   - run the relevant upload-component test command and verify it fails
   - implement the minimal progress-percentage UI in the existing upload component

4. Hand execution to **`subagent-driven-development`** or **`executing-plans`**, then finish with **`verification-before-completion`** after the tests pass.

Correctly identified as in-scope (no brainstorming)? Yes.
Wrote Iteration N section? Yes.
New rationalizations or loopholes found: None. The confusing row in the "When NOT to Use" table did not cause this agent to redirect to `brainstorming`.

## Scenario 3: Scope violation (with skill)
1. Announce: “I’m using iterating-on-implementation for an in-scope iteration,” then open `docs/plans/2026-05-01-user-auth.md`.
2. Do the mandatory scope check against the completed auth plan and explicitly mark the request as out of scope: new admin dashboard, session management, CSV export, and password reset are new components/features.
3. State the change request and the out-of-scope areas in the plan context, and stop the iteration path rather than appending `Iteration N`.
4. Redirect to `brainstorming` to define a new spec/plan for the admin dashboard work before any implementation.

Correctly redirected to brainstorming? Yes.
New rationalizations or loopholes found: None.

## Loopholes to fix
- None found across the three pressure scenarios.
