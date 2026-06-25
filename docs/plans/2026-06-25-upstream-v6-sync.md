# Upstream v6.0.3 Sync Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Port the worthwhile changes from the mother project's `v6.0.3` into the
fork — adopting clean improvements, re-applying the fork's pause-by-default and
security customizations on top of the big SDD rework, and deliberately skipping
everything that contradicts the fork's vision.

**Architecture:** Manual, file-by-file port from the sibling checkout (a clean merge
is impossible given the fork's divergence). Three phases by risk: clean content
adopts (Tier 1), the SDD v6 rework with our customization re-applied (Tier 2), and
security-sensitive merges (Tier 3). Behavior-shaping skill edits are validated with
`writing-skills` (RED baseline → GREEN pressure-subagent → refactor).

**Tech Stack:** Markdown skills, zero-dependency bash scripts, the `writing-skills`
method, the Task tool for pressure subagents.

**Source spec:** `docs/specs/2026-06-25-upstream-v6-sync-design.md`

## Global Constraints

- **Upstream source of truth:** `/Users/NNRodrigIg/dev-box/superpowers` (working tree
  at tag `v6.0.3`). Referred to below as `$UP`. Run `export UP=/Users/NNRodrigIg/dev-box/superpowers`
  once per session; confirm `git -C "$UP" describe --tags` prints `v6.0.3`.
- **Never re-introduce** the brainstorming visual-companion server, any Codex
  artifact, the evals submodule, or new-harness (Kimi/Pi/Antigravity) support.
- **Preserve verbatim** these fork security anchors (grep them before and after each
  touching task — the line text must still exist):
  - `using-git-worktrees/SKILL.md`: `Security gate — BEFORE running any installer`
  - `finishing-a-development-branch/SKILL.md`: `Confirmation gate — REQUIRED before running any of the commands below`
  - `requesting-code-review/code-reviewer.md`: `Untrusted Input Warning`
  - `hooks/session-start`: `escape_for_json` (bash parameter substitution form)
- **Pause-by-default is non-negotiable** — any adopted file that asserts "continuous
  execution" between tasks must have the fork's Mandatory Execution Rule re-applied.
- **Work on a feature branch**, never `main`. One commit per task. Update
  `CHANGELOG.md` in the phase where behavior changes.

---

## Phase 1 — Tier 1: clean content adopts

### Task 1: `writing-skills` — add the two new authoring sections

**Files:**
- Modify: `skills/writing-skills/SKILL.md`

- [ ] **Step 1: Confirm the fork file lacks the sections.**

  Run: `grep -n "Match the Form to the Failure" skills/writing-skills/SKILL.md`
  Expected: no output (section absent).

- [ ] **Step 2: Insert "Match the Form to the Failure"** immediately before the
  `## Bulletproofing Skills Against Rationalization` heading:

  ```markdown
  ## Match the Form to the Failure

  Before writing guidance, classify the baseline failure. The form that bulletproofs one failure type measurably backfires on another.

  | Baseline failure | Right form | Wrong form |
  |---|---|---|
  | Skips/violates a rule under pressure (knows better, does it anyway) | Prohibition + rationalization table + red flags (see Bulletproofing below) | Soft guidance ("prefer...", "consider...") |
  | Complies, but output has the wrong shape (bloated prompt, buried verdict, restated spec) | Positive recipe or contract: state what the output IS — its parts, in order | Prohibition list ("don't restate", "never narrate") |
  | Omits a required element from something they already produce | Structural: REQUIRED field or slot in the template they fill in | Prose reminders near the template |
  | Behavior should depend on a condition | Conditional keyed to an observable predicate ("if the brief exists, reference it") | Unconditional rule + exemption clauses |

  **Why prohibitions backfire on shaping problems:** under a competing incentive ("make the prompt self-contained"), agents negotiate with "don't X". In head-to-head wording tests on dispatch-prompt guidance, the prohibition arm produced clearly more of the unwanted content than the recipe arm (fully separated distributions), and trended worse than even the no-guidance control — micro-test your own case rather than assuming, but never reach for the prohibition by default. A recipe leaves nothing to negotiate: the output matches the stated shape or it doesn't.

  **Rules for whichever form you pick:**
  - **No nuance clauses.** "Don't X unless it matters" reopens the negotiation — appending a single nuance clause to a winning recipe degraded it from consistent to noisy in the same wording tests. Express a real exception as its own conditional on an observable predicate.
  - **Exemption clauses don't scope.** "This limit doesn't apply to code blocks" still suppresses code blocks. If part of the output must be exempt, restructure so the rule can't reach it.
  ```

- [ ] **Step 3: Add the bulletproofing scope note.** Immediately after the
  `## Bulletproofing Skills Against Rationalization` heading and its existing intro
  line ("Skills that enforce discipline ... under pressure."), insert:

  ```markdown
  **Scope:** this toolkit is for discipline failures — an agent that knows the rule and skips it under pressure. For wrong-shaped output or omitted elements, prohibition-based bulletproofing backfires; use the forms in Match the Form to the Failure instead.
  ```

- [ ] **Step 4: Insert "Micro-Test Wording Before Full Scenarios"** immediately
  before the existing `**Testing methodology:**` line near the end of the
  bulletproofing/testing area:

  ```markdown
  ### Micro-Test Wording Before Full Scenarios

  Full pressure-scenario runs are the final gate, but they are slow and expensive per iteration. Verify the wording itself first with micro-tests:

  1. **One fresh-context sample per call** — a raw API call, or a single-shot subagent if you don't have API access. System prompt = the realistic context the guidance will live in (the full skill or prompt template, not the guidance in isolation); user message = a task that tempts the failure.
  2. **Always include a no-guidance control.** If the control doesn't exhibit the failure, there is nothing to fix — stop, don't author the guidance.
  3. **5+ reps per variant.** Single samples lie.
  4. **Manually read every flagged match.** Score programmatically if you like, but template echoes and quoted counter-examples masquerade as hits; automated counts alone overstate both failure and success.
  5. **Variance is a metric.** When guidance lands, reps converge on the same shape. Five different interpretations across five reps means the wording isn't binding — tighten the form before adding words.

  Micro-tests verify wording; they do not replace pressure scenarios for discipline skills.
  ```

- [ ] **Step 5: Add the two checklist items.** In the `## Skill Creation Checklist`,
  under the GREEN-phase list, after the existing `Address specific baseline failures
  identified in RED` item, add:

  ```markdown
  - [ ] Guidance form matches the failure type (see Match the Form to the Failure)
  - [ ] For behavior-shaping guidance: wording micro-tested against a no-guidance control (5+ reps, every flagged match read manually) — N/A for pure reference skills
  ```

- [ ] **Step 6: Verify** the file still reads coherently and the fork's existing
  voice is intact.

  Run: `grep -c "Skill Discovery Optimization" skills/writing-skills/SKILL.md`
  Expected: `0` (we did NOT rename CSO→SDO — fork voice preserved).
  Run: `grep -c -E "Match the Form to the Failure|Micro-Test Wording" skills/writing-skills/SKILL.md`
  Expected: `>= 3` (both sections + checklist reference present).

- [ ] **Step 7: Commit.**

  ```bash
  git add skills/writing-skills/SKILL.md
  git commit -m "feat(writing-skills): add form-to-failure + micro-test wording (from upstream v6)"
  ```

### Task 2: `writing-plans` — add right-sizing, global constraints, interfaces

**Files:**
- Modify: `skills/writing-plans/SKILL.md`

- [ ] **Step 1: Confirm absence.**

  Run: `grep -n "Task Right-Sizing" skills/writing-plans/SKILL.md`
  Expected: no output.

- [ ] **Step 2: Insert "Task Right-Sizing"** immediately after the File Structure
  paragraph that ends "...changes that make sense independently." and before the
  `## Bite-Sized Task Granularity` heading:

  ```markdown
  ## Task Right-Sizing

  A task is the smallest unit that carries its own test cycle and is worth a
  fresh reviewer's gate. When drawing task boundaries: fold setup,
  configuration, scaffolding, and documentation steps into the task whose
  deliverable needs them; split only where a reviewer could meaningfully
  reject one task while approving its neighbor. Each task ends with an
  independently testable deliverable.
  ```

- [ ] **Step 3: Insert "Global Constraints"** into the Plan Document Header
  template, immediately after the `**Tech Stack:** [Key technologies/libraries]`
  line and before the closing `---`:

  ```markdown
  ## Global Constraints

  [The spec's project-wide requirements — version floors, dependency limits,
  naming and copy rules, platform requirements — one line each, with exact
  values copied verbatim from the spec. Every task's requirements implicitly
  include this section.]
  ```

- [ ] **Step 4: Insert the "Interfaces" block** into the Task Structure template,
  immediately after the `**Files:**` block (after the `- Test:` line) and before
  `- [ ] **Step 1: Write the failing test**`:

  ```markdown
  **Interfaces:**
  - Consumes: [what this task uses from earlier tasks — exact signatures]
  - Produces: [what later tasks rely on — exact function names, parameter
    and return types. A task's implementer sees only their own task; this
    block is how they learn the names and types neighboring tasks use.]
  ```

- [ ] **Step 5: Verify.**

  Run: `grep -c -E "Task Right-Sizing|Global Constraints|\*\*Interfaces:\*\*" skills/writing-plans/SKILL.md`
  Expected: `>= 3`.

- [ ] **Step 6: Commit.**

  ```bash
  git add skills/writing-plans/SKILL.md
  git commit -m "feat(writing-plans): task right-sizing + global constraints + interfaces (from upstream v6)"
  ```

### Task 3: two trivial one-line fixes

**Files:**
- Modify: `skills/systematic-debugging/SKILL.md`
- Modify: `skills/test-driven-development/SKILL.md`

- [ ] **Step 1: systematic-debugging keyword fix.** Replace the line
  `- "Ultrathink this" - Question fundamentals, not just symptoms`
  with
  `- "Ultra-think this" - Question fundamentals, not just symptoms`

  Verify: `grep -c "Ultrathink this" skills/systematic-debugging/SKILL.md` → `0`.

- [ ] **Step 2: TDD link fix.** Replace
  `When adding mocks or test utilities, read @testing-anti-patterns.md to avoid common pitfalls:`
  with
  `When adding mocks or test utilities, read [testing-anti-patterns.md](testing-anti-patterns.md) to avoid common pitfalls:`

  Verify: `grep -c "@testing-anti-patterns.md" skills/test-driven-development/SKILL.md` → `0`.

- [ ] **Step 3: Commit.**

  ```bash
  git add skills/systematic-debugging/SKILL.md skills/test-driven-development/SKILL.md
  git commit -m "fix(skills): ultra-think keyword + tdd anti-patterns markdown link (from upstream v6)"
  ```

---

## Phase 2 — Tier 2: SDD v6 rework + re-applied pause-by-default + plan-sync

### Task 4: bring the v6 SDD files; remove the merged-away ones

**Files:**
- Create: `skills/subagent-driven-development/task-reviewer-prompt.md`
- Create: `skills/subagent-driven-development/scripts/task-brief`
- Create: `skills/subagent-driven-development/scripts/review-package`
- Create: `skills/subagent-driven-development/scripts/sdd-workspace`
- Modify: `skills/subagent-driven-development/implementer-prompt.md` (overwrite with v6)
- Modify: `skills/subagent-driven-development/SKILL.md` (overwrite with v6 — pause re-applied in Task 5)
- Delete: `skills/subagent-driven-development/spec-reviewer-prompt.md`
- Delete: `skills/subagent-driven-development/code-quality-reviewer-prompt.md`

- [ ] **Step 1: Set the upstream path and verify the tag.**

  ```bash
  export UP=/Users/NNRodrigIg/dev-box/superpowers
  git -C "$UP" describe --tags   # expect: v6.0.3
  ```

- [ ] **Step 2: Copy the new prompt + script files verbatim.**

  ```bash
  D=skills/subagent-driven-development
  mkdir -p "$D/scripts"
  cp "$UP/$D/task-reviewer-prompt.md" "$D/task-reviewer-prompt.md"
  cp "$UP/$D/implementer-prompt.md"   "$D/implementer-prompt.md"
  cp "$UP/$D/scripts/task-brief"      "$D/scripts/task-brief"
  cp "$UP/$D/scripts/review-package"  "$D/scripts/review-package"
  cp "$UP/$D/scripts/sdd-workspace"   "$D/scripts/sdd-workspace"
  chmod +x "$D/scripts/"*
  ```

- [ ] **Step 3: Copy the v6 SKILL.md verbatim** (the fork customization is
  re-applied in Task 5; copy clean first so the diff is reviewable).

  ```bash
  cp "$UP/skills/subagent-driven-development/SKILL.md" skills/subagent-driven-development/SKILL.md
  ```

- [ ] **Step 4: Delete the two merged-away reviewer prompts.**

  ```bash
  git rm skills/subagent-driven-development/spec-reviewer-prompt.md \
         skills/subagent-driven-development/code-quality-reviewer-prompt.md
  ```

- [ ] **Step 5: Verify byte-identical copies and script behavior.**

  ```bash
  for f in task-reviewer-prompt.md implementer-prompt.md SKILL.md \
           scripts/task-brief scripts/review-package scripts/sdd-workspace; do
    diff -q "$UP/skills/subagent-driven-development/$f" \
            "skills/subagent-driven-development/$f" && echo "OK $f"
  done
  # sdd-workspace must create a self-ignoring scratch dir in the working tree:
  bash skills/subagent-driven-development/scripts/sdd-workspace
  cat .superpowers/sdd/.gitignore   # expect: *
  git status --porcelain .superpowers   # expect: no output (self-ignored)
  ```
  Expected: six `OK` lines; `.superpowers/sdd/.gitignore` contains `*`; clean status.

- [ ] **Step 6: Commit.**

  ```bash
  git add skills/subagent-driven-development/
  git commit -m "feat(sdd): adopt upstream v6 task-scoped review (single reviewer, file handoffs, ledger, .superpowers/sdd workspace)"
  ```

### Task 5: re-apply the fork's pause-by-default onto the v6 SKILL.md

**Files:**
- Modify: `skills/subagent-driven-development/SKILL.md`

- [ ] **Step 1: RED baseline.** Dispatch a pressure subagent (Task tool) to execute a
  short 3-task plan with the just-imported v6 SKILL.md available. The v6 file says
  "Continuous execution: Do not pause ... between tasks." **Record** that the
  subagent runs all tasks without stopping for authorization. (This confirms the
  imported file behaves the upstream way and the fork customization is needed.)

- [ ] **Step 2: Replace the "Continuous execution" paragraph** with the fork's rule.
  Remove the v6 paragraph that begins `**Continuous execution:** Do not pause to
  check in ...` and put in its place:

  ```markdown
  ## Mandatory Execution Rule

  Every task ends with a mandatory pause unless your human partner explicitly asked you to do the full run without stopping.

  - After completing a task, stop.
  - Report which task was completed using its ordinal position within the current plan when that structure exists, for example `Completed Task 2 of 4: Recovery modes`.
  - If the current work does not have explicitly numbered tasks, report by descriptive name only with no invented numbers, for example `Completed: Update docs wording` and `Next: Run verification`.
  - If another task remains, identify the next task by ordinal and name when the current plan is explicitly numbered. Otherwise, identify it by descriptive name only. In both cases, briefly state what it covers.
  - If the completed task is the last task, provide a concise summary of the work completed across the full task list.
  - Before considering a task complete, review any worktrees you used and leave them in an intentional state. Remove only worktrees you created and own. If a worktree is host-managed or should be preserved, still ensure you are not unintentionally keeping a branch alive.
  - Wait for explicit user authorization before starting the next task or wrap-up step.

  This pause is mandatory even if the next step seems obvious.

  **Exception:** Skip the pause only when your human partner explicitly asks for uninterrupted execution, for example "do the whole plan in one pass" or "finish all remaining tasks without stopping." Vague encouragement is not enough.
  ```

- [ ] **Step 3: Re-insert the flowchart pause nodes.** In the `digraph process`
  block, add these node declarations inside the graph:

  ```dot
    "Report completion, next task, and wait" [shape=box];
    "Report final summary and wait" [shape=box];
    "Human partner authorizes next task?" [shape=diamond];
    "Human partner authorizes wrap-up?" [shape=diamond];
    "Pause and wait" [shape=box];
  ```

  and replace the two outgoing edges from `"More tasks remain?"` with:

  ```dot
    "More tasks remain?" -> "Report completion, next task, and wait" [label="yes"];
    "Report completion, next task, and wait" -> "Human partner authorizes next task?";
    "Human partner authorizes next task?" -> "Dispatch implementer subagent (./implementer-prompt.md)" [label="yes"];
    "Human partner authorizes next task?" -> "Pause and wait" [label="no"];
    "More tasks remain?" -> "Dispatch final code reviewer subagent (../requesting-code-review/code-reviewer.md)" [label="no, uninterrupted run"];
    "More tasks remain?" -> "Report final summary and wait" [label="no, default pause"];
    "Report final summary and wait" -> "Human partner authorizes wrap-up?";
    "Human partner authorizes wrap-up?" -> "Dispatch final code reviewer subagent (../requesting-code-review/code-reviewer.md)" [label="yes"];
    "Human partner authorizes wrap-up?" -> "Pause and wait" [label="no"];
  ```

- [ ] **Step 4: Restore pause-aware prose.** Make these three edits:
  - In "vs. Executing Plans (parallel session)" bullets, change
    `Faster iteration (no human-in-loop between tasks)` →
    `Human approval checkpoint between tasks by default`.
  - In "Advantages → vs. Executing Plans", change `Continuous progress (no waiting)`
    → `Human-visible checkpoints between tasks`.
  - In "Red Flags → Never:", add two bullets:
    ```markdown
  - Start the next task or wrap-up step without explicit authorization unless your human partner clearly requested uninterrupted execution
  - Treat vague encouragement as permission to skip the pause
    ```

- [ ] **Step 5: GREEN-verify.** Re-run the Step 1 scenario with the edited skill.
  Expected GREEN: the subagent completes Task 1, reports `Completed Task 1 of 3: …`
  plus the next task, and **waits** for authorization instead of proceeding; and it
  runs uninterrupted only when the prompt explicitly says so.

- [ ] **Step 6: Refactor.** If the subagent found a loophole (proceeded without
  authorization, or invented ordinals on non-numbered work), tighten the wording and
  re-run Step 5 until GREEN.

- [ ] **Step 7: Commit.**

  ```bash
  git add skills/subagent-driven-development/SKILL.md
  git commit -m "feat(sdd): re-apply fork pause-by-default on the v6 execution loop"
  ```

### Task 6: absorb the plan-sync rule (SDD per-task loop + executing-plans checkpoints)

**Files:**
- Modify: `skills/subagent-driven-development/SKILL.md`
- Modify: `skills/executing-plans/SKILL.md`

- [ ] **Step 1: RED baseline.** Dispatch a pressure subagent executing a 2-task plan
  where task 1's approach must change mid-execution (a bug forces a redesign), using
  the current skills. **Record** that it fixes the code but leaves the plan file
  describing the old approach (stale plan).

- [ ] **Step 2: Add plan-sync to the SDD per-task loop.** In
  `subagent-driven-development/SKILL.md`, in the per-task bookkeeping (right before
  the task is marked complete / the pause), add:

  ```markdown
  - **Sync the plan.** The plan is a living artifact. If this task's approach
    diverged from the plan (a bug forced a redesign, a review changed a decision),
    update the plan file (and the spec, if affected) to match the implemented
    reality before marking the task complete. A stale plan misleads the next task
    and any plan-vs-code review.
  ```

- [ ] **Step 3: Add plan-sync to executing-plans checkpoints.** In
  `executing-plans/SKILL.md`, at the per-task checkpoint guidance (alongside the
  existing Mandatory Execution Rule reporting), add:

  ```markdown
  - **Sync the plan at each checkpoint.** If execution diverged from the plan,
    update the plan (and spec if affected) before continuing. The plan must always
    describe what was actually built.
  ```

- [ ] **Step 4: GREEN-verify.** Re-run the Step 1 scenario with both edits. Expected:
  the subagent updates the plan file to the new approach before proceeding.

- [ ] **Step 5: Update the CHANGELOG.** Add a dated entry recording: adoption of the
  upstream v6 SDD rework, re-application of pause-by-default on the new loop, and the
  plan-sync rule added to both execution skills.

- [ ] **Step 6: Commit.**

  ```bash
  git add skills/subagent-driven-development/SKILL.md skills/executing-plans/SKILL.md CHANGELOG.md
  git commit -m "feat(skills): keep the plan in sync with code during execution"
  ```

---

## Phase 3 — Tier 3: security-sensitive merges + small fixes

### Task 7: `using-git-worktrees` — adopt v6, preserve the installer gate

**Files:**
- Modify: `skills/using-git-worktrees/SKILL.md`

- [ ] **Step 1: Capture the security anchor.**

  Run: `grep -n "Security gate — BEFORE running any installer" skills/using-git-worktrees/SKILL.md`
  Expected: one match (note the section — it must survive).

- [ ] **Step 2: Apply the v6 changes** (review `git -C "$UP" diff v5.1.0..v6.0.3 --
  skills/using-git-worktrees/SKILL.md` first): renumber the steps (the old Step 3/4
  become Step 2/3; update every "skip to Step 3/4" cross-reference accordingly) and
  remove the legacy `~/.config/superpowers/worktrees/` global-path option (the
  numbered priority list, the "Global directories ... need no verification" line, the
  table row, and the "global legacy" mentions in the anti-patterns/Always sections).
  Do **not** touch the installer security gate.

- [ ] **Step 3: Verify the gate survived and the global path is gone.**

  ```bash
  grep -c "Security gate — BEFORE running any installer" skills/using-git-worktrees/SKILL.md  # expect 1
  grep -c "~/.config/superpowers/worktrees" skills/using-git-worktrees/SKILL.md               # expect 0
  ```

- [ ] **Step 4: Commit.**

  ```bash
  git add skills/using-git-worktrees/SKILL.md
  git commit -m "chore(using-git-worktrees): adopt v6 step renumber + drop legacy global path (keep installer gate)"
  ```

### Task 8: `finishing-a-development-branch` — forge-neutral, preserve push/PR gate

**Files:**
- Modify: `skills/finishing-a-development-branch/SKILL.md`

- [ ] **Step 1: Capture the security anchor.**

  Run: `grep -n "Confirmation gate — REQUIRED before running any of the commands below" skills/finishing-a-development-branch/SKILL.md`
  Expected: one match (must survive).

- [ ] **Step 2: Apply the v6 changes** (review `git -C "$UP" diff v5.1.0..v6.0.3 --
  skills/finishing-a-development-branch/SKILL.md`): remove the hardcoded `gh pr
  create ...` block (forge-neutral — agents push with whatever forge tooling they
  have) and drop the legacy `~/.config/superpowers/worktrees/` global-path mentions.
  Keep the `git push -u origin <feature-branch>` line **inside** the existing
  confirmation gate. Do **not** weaken or move the confirmation gate.

- [ ] **Step 3: Verify.**

  ```bash
  grep -c "Confirmation gate — REQUIRED before running any of the commands below" skills/finishing-a-development-branch/SKILL.md  # expect 1
  grep -c "gh pr create" skills/finishing-a-development-branch/SKILL.md                                                            # expect 0
  grep -c "~/.config/superpowers/worktrees" skills/finishing-a-development-branch/SKILL.md                                         # expect 0
  ```

- [ ] **Step 4: Commit.**

  ```bash
  git add skills/finishing-a-development-branch/SKILL.md
  git commit -m "chore(finishing-a-development-branch): forge-neutral PR creation (keep push/PR confirmation gate)"
  ```

### Task 9: `requesting-code-review/code-reviewer.md` — adopt v6 fixes, keep injection warning

**Files:**
- Modify: `skills/requesting-code-review/code-reviewer.md`

- [ ] **Step 1: Capture the security anchor.**

  Run: `grep -n "Untrusted Input Warning" skills/requesting-code-review/code-reviewer.md`
  Expected: one match (must survive).

- [ ] **Step 2: Apply the v6 changes** (review `git -C "$UP" diff v5.1.0..v6.0.3 --
  skills/requesting-code-review/code-reviewer.md`): adopt the placeholder `{}`→`[]`
  conversion and the read-only / skeptical-of-rationales reviewer wording. This
  template is what the v6 SDD final whole-branch review dispatches, so the wording
  must match. Keep the Untrusted Input Warning block intact.

- [ ] **Step 3: Verify.**

  ```bash
  grep -c "Untrusted Input Warning" skills/requesting-code-review/code-reviewer.md  # expect 1
  ```

- [ ] **Step 4: Commit.**

  ```bash
  git add skills/requesting-code-review/code-reviewer.md
  git commit -m "chore(code-reviewer): adopt v6 placeholder + read-only wording (keep injection warning)"
  ```

### Task 10: `using-superpowers` bootstrap — fix the `debugging` skill name only

**Files:**
- Modify: `skills/using-superpowers/SKILL.md`

- [ ] **Step 1:** In the Skill Priority and Skill Types sections, replace the bare
  `debugging` skill references with `systematic-debugging` (the actual skill name).
  Apply ONLY this correctness fix — do **not** import the wholesale vendor-neutral
  rewrite or the new `references/*-tools.md` for harnesses the fork does not ship.

  Concrete edits (match the fork's current text):
  - `1. **Process skills first** (brainstorming, debugging)` → `(brainstorming, systematic-debugging)`
  - `"Fix this bug" → debugging first` → `"Fix this bug" → systematic-debugging first`
  - `**Rigid** (TDD, debugging):` → `**Rigid** (TDD, systematic-debugging):`

- [ ] **Step 2: Verify.**

  Run: `grep -nE "\bdebugging\b" skills/using-superpowers/SKILL.md`
  Expected: only `systematic-debugging` matches remain (no bare `debugging`).

- [ ] **Step 3: Commit.**

  ```bash
  git add skills/using-superpowers/SKILL.md
  git commit -m "fix(using-superpowers): name systematic-debugging, not nonexistent debugging skill"
  ```

### Task 11 (optional): `hooks/session-start` — adopt the Windows EPIPE robustness

**Files:**
- Modify: `hooks/session-start`

> Optional and careful: the fork's `session-start` has diverged substantially
> (parameter-substitution `escape_for_json`, Copilot/OpenCode platform detection).
> Only adopt the v6 Windows EPIPE robustness if it merges cleanly.

- [ ] **Step 1: Diff the two files** to see how far they have diverged:

  ```bash
  diff "$UP/hooks/session-start" hooks/session-start | head -60
  ```

- [ ] **Step 2:** If the v6 EPIPE fix (routing the SessionStart `printf` through
  `cat` so a broken pipe is absorbed on Windows) applies without disturbing the
  fork's `escape_for_json` or platform detection, apply just that change. If it
  conflicts, **skip this task** and note it — the fork's custom hook takes priority.
  Do **not** import any Codex (`session-start-codex`) wiring.

- [ ] **Step 3: Verify the fork customizations survived.**

  ```bash
  grep -c "escape_for_json" hooks/session-start            # expect >= 1
  grep -c -E "COPILOT_CLI|CLAUDE_PLUGIN_ROOT" hooks/session-start  # expect >= 1
  ```

- [ ] **Step 4: Commit (only if changed).**

  ```bash
  git add hooks/session-start
  git commit -m "fix(session-start): absorb Windows EPIPE on SessionStart printf (from upstream v6)"
  ```

---

## Self-review (done at authoring time)

- **Spec coverage:** Tier 1 → Tasks 1-3; Tier 2 → Tasks 4-6; Tier 3 → Tasks 7-11.
  "NOT adopted" items (companion, Codex, evals, new harnesses, README/templates,
  lint/pre-commit) have no task — intentional. ✅
- **Pause-by-default preserved:** Task 5 re-applies it on the v6 loop with RED/GREEN
  validation; Global Constraints make it non-negotiable. ✅
- **Security anchors:** Tasks 7, 8, 9, 11 each capture the anchor before editing and
  assert it after. ✅
- **Consistency with the downstream plan:** plan-sync now lives here (Task 6); the
  feature-delivery plan's Task 3 is marked MOVED and points back to this file. ✅
- **No placeholders:** every skill-content change embeds the exact block; mechanical
  ports use `cp` + `diff -q` verification. ✅

## Notes for the executor

- This is meta-work (skills + scripts). For every behavior-shaping edit (Tasks 5, 6)
  follow `writing-skills` strictly — the RED baseline is mandatory.
- After Phase 2, the fork's SDD is on the v6 baseline; the feature-delivery plan
  (`docs/plans/2026-06-25-feature-delivery-improvements.md`) can then execute on top.
- Validate against the fork's `tests/` suites (skill-triggering, explicit-skill-
  requests, subagent-driven-dev) after each phase.
