# Baseline Test Results

## Scenario 1: Bug fix after implementation

### Agent response (verbatim)
1. Reproduce it exactly: run the existing auth tests or app login flow with a password like `P@ss!word#1` and confirm the failure is real and consistent.  
2. Read the relevant plan/task notes and inspect the auth login path + recent changes (`git diff`/last commits) to see where passwords are parsed, transmitted, and compared.  
3. Add a failing regression test for “login succeeds with special characters in password” and watch it fail for the expected reason.  
4. Trace the password end-to-end (request payload → parsing → auth service → password check) to find the root cause before changing code.

### Observed failures
- First move was reproduction/debugging work, not an explicit iteration step for reopening completed implementation work.
- It read the existing plan/task notes, but did not create any structured iteration section or follow-up addendum in the plan.
- It did not do an explicit scope check before moving into debugging.
- It did not delegate to an execution skill; it kept the whole flow as ad hoc next steps.

### Rationalizations/shortcuts used (verbatim)
- "Read the relevant plan/task notes and inspect the auth login path + recent changes (`git diff`/last commits)"
- "Trace the password end-to-end ... to find the root cause before changing code."

## Scenario 2: Small functional addition

### Agent response (verbatim)
1. Treat it as a **new behavior change**, not “just tweak code”: reread `docs/plans/2026-04-15-file-upload.md` plus the existing upload component/tests to anchor on current behavior.

2. Do a **brief brainstorming pass** for this add-on: confirm scope is “existing upload component only, simple percentage/progress bar,” then present a tiny design and get approval.

3. Because brainstorming says implementation must flow through planning, create a **small follow-up plan/addendum** for the progress indicator work.

4. Then use **TDD**: write a failing test for upload progress percentage display, run it to see it fail, implement the minimal change, and rerun targeted tests.

### Observed failures
- First move was rereading the old plan and component/tests; it did not establish a dedicated iteration workflow for post-implementation changes.
- It did do a scope check, but only informally inside a brainstorming pass rather than with a concrete iteration section.
- It wanted to "present a tiny design and get approval" instead of acting autonomously on a small in-scope iteration.
- It mentioned creating a follow-up plan/addendum, but not a structured iteration section tied to the completed plan.
- It did not delegate to an execution skill; it named process skills (brainstorming, TDD) but no execution handoff.

### Rationalizations/shortcuts used (verbatim)
- "Treat it as a **new behavior change**, not ‘just tweak code’"
- "Do a **brief brainstorming pass**"
- "Because brainstorming says implementation must flow through planning"
- "present a tiny design and get approval"

## Patterns across scenarios
- Agents naturally fall back to generic implementation/debugging habits (reproduce, inspect code, write test) rather than a distinct post-implementation iteration workflow.
- Reading the old plan may happen, but structured iteration bookkeeping does not happen automatically.
- Scope checking is inconsistent: one scenario skipped it, the other did it informally.
- Agents do not naturally delegate iteration work to an execution-oriented skill.
- Small follow-up requests can trigger over-planning, approval-seeking, or re-running earlier process skills instead of a lightweight, explicit iteration path.
