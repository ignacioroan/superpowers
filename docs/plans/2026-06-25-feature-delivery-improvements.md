# Feature-Delivery Improvements Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add/upgrade Superpowers skills so feature delivery verifies the *rendered* UI, doesn't skip semantic-a11y / component-reuse / layout checks, keeps the plan in sync with code, and can be driven end-to-end by a thin `superdeveloper` orchestrator — all project-agnostic.

**Architecture:** Four workstreams across three phases. New skills `visual-verification` and `superdeveloper`; edits to `implementation-verifying`, `subagent-driven-development`, `executing-plans`. Every skill change is authored with **writing-skills TDD**: establish a RED baseline (a pressure subagent fails the behavior without the skill), write/edit the skill, then GREEN-verify (a fresh pressure subagent complies with the skill present), then refactor.

**Tech Stack:** Markdown skills (`SKILL.md` + frontmatter `name`/`description`), the `writing-skills` method, the Task tool for pressure subagents, and the fork's existing `tests/` skill suites.

**Source spec:** `docs/specs/2026-06-25-feature-delivery-improvements-design.md`

> **Sequencing / depends on:** the upstream-v6 sync plan
> (`docs/plans/2026-06-25-upstream-v6-sync.md`). Execute that sync **first**; this
> plan runs on the post-sync baseline. Two reconciliations follow from that decision:
> - **Task 3 (plan-sync) is MOVED into the sync plan.** The sync already rewrites the
>   SDD execution loop (to bring in v6 *and* re-apply our pause-by-default), so the
>   plan-sync rule is added there, in the same edit, against the v6 structure — not
>   here.
> - **Authoring method:** the sync upgrades `writing-skills` ("Match the Form to the
>   Failure" + "Micro-Test Wording"). When drafting the skills below, classify each
>   baseline failure first and pick recipe/contract vs prohibition accordingly. The
>   GREEN drafts here are starting points, not final wording.

---

## Files

| Action | Path | Responsibility |
|---|---|---|
| Create | `skills/visual-verification/SKILL.md` | Render UI + screenshot matrix (viewports × states), compare, report defects |
| Create | `skills/visual-verification/failure-modes.md` | Universal rendered-layout failure modes reference |
| Modify | `skills/implementation-verifying/SKILL.md` | Invoke visual-verification; add a11y / reuse / layout checks |
| ~~Modify~~ | `skills/subagent-driven-development/SKILL.md` | ~~Add plan-sync rule to the per-task loop~~ → **MOVED to upstream-v6 sync plan** |
| ~~Modify~~ | `skills/executing-plans/SKILL.md` | ~~Add plan-sync rule to batch checkpoints~~ → **MOVED to upstream-v6 sync plan** |
| Create | `skills/superdeveloper/SKILL.md` | Thin orchestrator chaining the skills with human gates |

> **Method note (applies to every task):** Per `writing-skills`, the *exact wording*
> of a skill is refined against the rationalizations a real baseline subagent
> produces. The content blocks below are the **GREEN target draft** sourced from the
> spec — start from them, then tighten them to close whatever loopholes the baseline
> reveals. Do not skip the baseline: "if you didn't watch an agent fail without the
> skill, you don't know if the skill teaches the right thing."

---

## Phase 1 — Rendered verification + verification checklist

### Task 1: `visual-verification` skill

**Files:**
- Create: `skills/visual-verification/SKILL.md`
- Create: `skills/visual-verification/failure-modes.md`

- [ ] **Step 1: Establish the RED baseline.** Dispatch a pressure subagent (Task
  tool) with no knowledge of this skill:

  > "Here is an implemented UI component and a running component explorer at
  > <url>. The design says it shows multiple items in a row. Confirm the
  > implementation is done and matches the design."

  Use a component whose source *reads* correct but *renders* wrong (e.g. a
  flex-sizing defect that collapses a list to one visible item). **Record** whether
  the subagent declares it done from source/reading alone without rendering and
  inspecting. Expected RED: it does not render, or renders without a viewport/state
  matrix, and misses the defect.

- [ ] **Step 2: Write `failure-modes.md`** with the universal CSS failure modes
  (project-agnostic) from the spec:

  ```markdown
  # Rendered-layout failure modes (universal)

  When a rendered layout looks wrong, check these before guessing:

  - **Flex item with `flex: 0 0 auto` + a percentage-width child** has no definite
    size — the item fills its container, so a "row of items" shows only one.
    Put the size on the flex item itself, or use a sizing API that sets the item
    basis.
  - **`align-items: stretch`** (the flex default) stretches children/media along the
    cross axis to full width. Use `align-self`/`width` to opt a child out.
  - **An absolutely-positioned only-child** removes itself from flow, collapsing a
    parent's shrink-to-fit width to its container's full width.
  - **A container with no shared page layout** has no gutters/grid and sits flush to
    the viewport edges — confirm it uses the project's layout primitive.

  Always inspect computed styles (`getComputedStyle`) at the failing breakpoint to
  confirm the cause; do not guess.
  ```

- [ ] **Step 3: Write `skills/visual-verification/SKILL.md`** (GREEN target draft):

  ```markdown
  ---
  name: visual-verification
  description: Use when UI work is implemented and you need to confirm the rendered output matches the design — captures a screenshot matrix across viewports and states from a running dev server / component explorer and compares against the reference, before claiming a UI change is done
  ---

  # Visual Verification

  ## Overview

  **A spec you read is not the output you ship. Render it and look.**

  Reading a design source tells you what *should* render; it does not tell you what
  *does*. This skill renders the implementation and captures a screenshot matrix
  (viewports × states), compares it to the design reference and expected behavior,
  and reports defects with root causes. Single responsibility: it renders and
  compares; it does not do the full design-source cross-check (that is
  `implementation-verifying`, which calls this skill).

  **Announce at start:** "I'm using visual-verification to check the rendered UI."

  ## Process

  1. **Pick a render surface** (tool-agnostic): a component-explorer story, an app
     route, or a harness, served by a running dev server. Confirm the server is up.
  2. **Pick a capture tool** by what's available — the contract is "capture N
     viewports × M states to a known directory", not a specific tool. Any of: a
     preview/automation MCP, a headless-browser skill, or a connected browser.
  3. **Viewport matrix:** at minimum mobile / tablet / desktop, plus every
     breakpoint the design defines.
  4. **State matrix:** the meaningful states — empty, min, max, with/without each
     optional element, long-content, and edge/error states.
  5. **Capture** the full matrix to a known directory.
  6. **Compare** each shot to the design reference frame and to expected behavior.
     When layout looks wrong, **inspect computed styles** to find the root cause —
     see `failure-modes.md`. Do not guess.
  7. **Report** findings: defect, viewport/state, suspected root cause, fix
     direction. Feed fixes back, re-capture, confirm fixed.

  ## Red flags (you are not done)

  - You concluded "matches the design" without rendering it.
  - You captured one viewport / one state.
  - You saw a layout defect and guessed the cause without inspecting computed styles.

  ## Reference

  `failure-modes.md` — universal rendered-layout failure modes.
  ```

- [ ] **Step 4: GREEN-verify.** Dispatch a fresh pressure subagent with the same
  scenario as Step 1 *plus* the new skill available. Expected GREEN: it renders,
  captures a viewport×state matrix, finds the planted defect, and inspects computed
  styles for the cause — and does NOT declare done from source alone.

- [ ] **Step 5: Refactor.** If the subagent found a loophole (e.g. captured one
  viewport, or declared done early), tighten the skill wording to close it and
  re-run Step 4 until GREEN.

- [ ] **Step 6: Commit.**

  ```bash
  git add skills/visual-verification/SKILL.md skills/visual-verification/failure-modes.md
  git commit -m "feat(skills): add visual-verification skill"
  ```

### Task 2: enhance `implementation-verifying`

**Files:**
- Modify: `skills/implementation-verifying/SKILL.md`

- [ ] **Step 1: RED baseline.** Dispatch a pressure subagent to "verify this feature
  before PR" against an implementation that has all four planted defects: (a) a
  layout defect visible only when rendered, (b) an empty semantic wrapper, (c) a
  hand-rolled primitive a component library covers, (d) a bare container with no
  shared layout. Use the *current* `implementation-verifying`. Record that it flags
  none of the four (it reads design nodes, never renders; has no a11y/reuse/layout
  checks).

- [ ] **Step 2: Edit the skill — add a rendered-verification step.** In the Process
  section, between "Read the implementation" and "Compare systematically", insert:

  ```markdown
  5.5. **Render and verify visually.** Run the `visual-verification` skill across
  viewports and states. Rows in the gap table MUST include findings from the
  rendered comparison, not only from reading design nodes. (A read-only check is
  how rendered layout defects slip through.)
  ```

  And add to the Prerequisites table a fourth row:

  ```markdown
  | **Running UI** | A dev server / component explorer must be running so the implementation can be rendered |
  ```

- [ ] **Step 3: Edit the skill — add the a11y semantic checklist.** Add a new
  section before "Output Section Format":

  ```markdown
  ## Accessibility checklist (run before sign-off)

  - **No empty semantic wrappers.** Every `<figure>`, `<figcaption>`, `<section>`,
    or landmark either has content or is conditionally rendered.
  - **Decorative images carry empty alt at the component level** (driven by a
    variant/role prop), not only in fixtures; meaningful images carry real alt.
  - **Controls have accessible names**; heading hierarchy is correct and the heading
    element is configurable; regions are labelled (`aria-labelledby`/`aria-label`)
    where the pattern needs it.
  ```

- [ ] **Step 4: Edit the skill — add reuse + layout checks to the gap table.** In
  the "Gap Table" guidance, add two row-classes the verifier must produce:

  ```markdown
  - **Reuse-before-rebuild:** for every hand-rolled common primitive (surface/card,
    button, link, list/carousel, image, input), a row confirming no
    component-library component covers it. The concrete catalog is the consuming
    project's agent guide; the check is mandatory here.
  - **Layout primitive:** a row confirming the block/page container uses the
    project's shared layout primitive (grid/gutters/rhythm), not a bare flex element.
  ```

- [ ] **Step 5: GREEN-verify.** Re-run the Step 1 scenario with the edited skill.
  Expected: all four defects flagged.

- [ ] **Step 6: Refactor** to close any loophole; re-run until GREEN.

- [ ] **Step 7: Commit.**

  ```bash
  git add skills/implementation-verifying/SKILL.md
  git commit -m "feat(skills): implementation-verifying renders UI + a11y/reuse/layout checks"
  ```

---

## Phase 2 — Plan ↔ code sync

### Task 3: plan-sync rule in two skills

> **⚠️ MOVED to the upstream-v6 sync plan** (`docs/plans/2026-06-25-upstream-v6-sync.md`).
> The sync rewrites the SDD execution loop (v6 + re-applied pause-by-default), so
> plan-sync is added there in the same edit, against the v6 structure. The steps
> below are kept for reference only — **do not execute them from this plan.**

**Files:**
- Modify: `skills/subagent-driven-development/SKILL.md`
- Modify: `skills/executing-plans/SKILL.md`

- [ ] **Step 1: RED baseline.** Dispatch a pressure subagent executing a 2-task plan
  where task 1's approach must change mid-execution (a bug forces a redesign). Use
  the *current* skills. Record that it fixes the code but leaves the plan file
  describing the old approach (stale plan).

- [ ] **Step 2: Edit `subagent-driven-development`.** In the per-task loop (after the
  task's reviews pass, before the pause), add:

  ```markdown
  - **Sync the plan.** The plan is a living artifact. If this task's approach
    diverged from the plan (a bug forced a redesign, a review changed a decision),
    update the plan file (and the spec, if affected) to match the implemented
    reality before marking the task complete. A stale plan misleads the next task
    and any plan-vs-code review.
  ```

- [ ] **Step 3: Edit `executing-plans`.** At the batch checkpoint guidance, add the
  same rule, scoped to checkpoints:

  ```markdown
  - **Sync the plan at each checkpoint.** If execution diverged from the plan,
    update the plan (and spec if affected) before continuing. The plan must always
    describe what was actually built.
  ```

- [ ] **Step 4: GREEN-verify.** Re-run the Step 1 scenario with both edits. Expected:
  the subagent updates the plan file to the new approach before proceeding.

- [ ] **Step 5: Commit.**

  ```bash
  git add skills/subagent-driven-development/SKILL.md skills/executing-plans/SKILL.md
  git commit -m "feat(skills): keep the plan in sync with code during execution"
  ```

---

## Phase 3 — `superdeveloper` orchestrator

### Task 4: `superdeveloper` skill

**Files:**
- Create: `skills/superdeveloper/SKILL.md`

- [ ] **Step 1: RED baseline.** Dispatch a pressure subagent told to "deliver this
  ticket end-to-end" with the individual skills available but no orchestrator.
  Record the failure modes to design against: it may skip the spec/plan gates, or
  (the key one) **stop after implementation without proceeding to verification**,
  needing a manual nudge.

- [ ] **Step 2: Write `skills/superdeveloper/SKILL.md`** (GREEN target draft):

  ```markdown
  ---
  name: superdeveloper
  description: Use to drive a feature end-to-end (intake → spec → plan → implementation → verification → branch finish) as one orchestrated workflow that chains the Superpowers skills with explicit human gates
  ---

  # superdeveloper

  ## Overview

  A thin orchestrator that takes a feature from intake to a finished branch by
  **delegating** to the existing Superpowers skills. It re-implements nothing; it
  sequences the skills and enforces the human gates. It runs in the main
  conversation (it needs interactive gates), and fans out per-task implementation to
  Task subagents via `subagent-driven-development`.

  **Announce at start:** "I'm using superdeveloper to drive this feature end-to-end."

  ## Workflow

  | Phase | Delegate to | Gate |
  |---|---|---|
  | Intake | read the ticket / design / component-library sources into context | — |
  | Design | `brainstorming` → spec | **Human: approve spec** |
  | Plan | `writing-plans` | **Human: "go"** |
  | Implement | `subagent-driven-development` | per-task pause (per that skill) |
  | Verify | `implementation-verifying` (runs `visual-verification`) | fix findings before proceeding |
  | Review | `receiving-code-review` on any feedback | — |
  | Finish | `finishing-a-development-branch` | **Human: PR/merge choice** |

  ## Reliability rules (non-negotiable)

  - **Never stall silently.** Auto-progress between phases. The ONLY stops are the
    named human gates above — emit them explicitly. Halting mid-phase without a
    named gate is a defect.
  - **After implementation, ALWAYS proceed to verification.** Never present a "done"
    summary that skips the Verify phase.
  - **Pin one model for the whole run.** Do not switch models mid-delivery.
  - **Name the stop.** When you pause, say whether it is an intentional gate or a
    blocker — never an ambiguous stop.
  - **Branch hygiene.** Work on a feature branch; never the default branch.

  ## What this is NOT

  - Not a re-implementation of TDD, review, or verification — it delegates.
  - Not a silent autonomous runner — it honors the human gates.
  - Not a replacement for the individual skills — each stays independently usable.
  ```

- [ ] **Step 3: GREEN-verify.** Dispatch a fresh pressure subagent: "deliver this
  with superdeveloper", then after the implement phase give it an *ambiguous* nudge
  ("ok"). Expected GREEN: it stops at the spec gate and the plan gate, and after
  implementation it **proceeds to verification on its own** rather than stopping —
  while still honoring the named gates.

- [ ] **Step 4: Refactor** to close loopholes (e.g. if it still stalled before
  verify, strengthen the "ALWAYS proceed to verification" rule); re-run until GREEN.

- [ ] **Step 5: Commit.**

  ```bash
  git add skills/superdeveloper/SKILL.md
  git commit -m "feat(skills): add superdeveloper end-to-end orchestrator"
  ```

---

## Self-review (done at authoring time)

- **Spec coverage:** W1 → Task 1; W2 → Task 2; W3 → Task 3 (**moved to the upstream-v6 sync plan**); W4 → Task 4. "Out of
  scope" (project specifics) → companion storefront plan, not here. ✅
- **Placeholders:** none — content blocks are real draft prose, refined via the
  writing-skills baseline loop (which is a method, not a placeholder). ✅
- **Consistency:** skill names (`visual-verification`, `superdeveloper`) and the
  cross-reference (`implementation-verifying` invokes `visual-verification`) match
  across tasks. ✅

## Notes for the executor

- This is meta-work (authoring skills). Follow `writing-skills` strictly — the RED
  baseline is mandatory; the draft content is the GREEN *target*, not the final
  word. Tighten wording against real baseline rationalizations.
- Validate against the fork's `tests/` suites (skill-triggering, explicit-skill-
  requests, subagent-driven-dev) after each task.
