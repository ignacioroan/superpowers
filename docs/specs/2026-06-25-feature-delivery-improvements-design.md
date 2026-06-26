# feature-delivery-improvements — Design Spec

**Date:** 2026-06-25
**Status:** Draft
**Scope:** Superpowers fork (this repo). Project-agnostic: no consuming project is
assumed. Project-specific knowledge (component catalogs, layout primitives, naming)
is explicitly out of scope and belongs in each consumer's own agent guide.

---

## Motivation

A controlled experiment delivered the same UI feature twice — once driven by the
Superpowers skills, once by a monolithic "developer agent" persona — and compared
the results. Five gaps generalized beyond the specific feature and motivate this
spec. None of the gaps are project-specific; they are workflow gaps.

1. **No rendered visual verification.** Both runs shipped a layout that was correct
   on paper but broken on screen, and only *rendering the implementation and looking
   at it* surfaced the defect. The existing `implementation-verifying` skill
   cross-checks design sources by **reading** them; it never **renders** the
   implementation. This is the biggest gap.

2. **No semantic-accessibility checklist.** Empty semantic wrappers and
   non-enforced decorative-image alt text shipped, because nothing forced an a11y
   pass on the rendered structure.

3. **No "reuse before rebuild" prompt.** Hand-rolled markup/styles were used where a
   component library already provided the primitive, because nothing prompted the
   agent to check for an existing component first.

4. **No layout-primitive prompt.** A container shipped as a bare flex element with
   no shared page layout (gutters/grid), because nothing prompted "does this use the
   project's layout primitive?".

5. **Plan ↔ code drift.** Fixes landed in code but the plan file was left stale,
   because nothing instructed the executor to keep the plan in sync.

A sixth observation concerns orchestration: a monolithic agent persona that chains
intake → plan → build → verify is convenient, but the one observed stalled between
phases and needed a manual nudge, switched models mid-run, and once skipped a gate.
Superpowers has every piece as a composable skill but no single "deliver this
feature end-to-end" entry point.

**Goal:** Encode these workflow learnings into the fork so future feature work
verifies the *rendered* output, doesn't skip semantic-a11y / component-reuse /
layout checks, keeps the plan in sync, and can optionally be driven end-to-end by
one thin orchestrator that guarantees progression while honoring Superpowers' human
gates — all without baking in any project's specifics.

---

## Design principles

- **Generic where it generalizes; frontend-domain where it doesn't.** Render-and-
  compare and plan-sync are universal. The screenshot matrix, a11y checklist, and
  reuse/layout checks are frontend-domain (they assume a UI, a design source, a
  component library) and live alongside the already frontend-domain
  `implementation-verifying`. None of them name a specific project's components.
- **No project specifics in skills.** Concrete component catalogs, layout
  primitives, and naming are per-project and belong in each consumer's own agent
  guide (e.g. its `AGENTS.md`) — `writing-skills`' rule that project-specific
  conventions do not go in skills. Skills carry the *principle and the habit of
  checking*; the consumer supplies the *concrete answers*.
- **Composable skills first, thin orchestration on top.** Nothing is re-implemented
  inside the orchestrator; it delegates.
- **One bounded responsibility per skill.**
- **Build/edit skills via `writing-skills` TDD** (RED baseline → write → GREEN
  verify with a pressure subagent → refactor). Every workstream that creates or
  edits a skill MUST follow that loop.

---

## Workstream 1 — New skill: `visual-verification` (frontend-domain)

**Responsibility (single):** Render a UI implementation and capture a *screenshot
matrix* (viewports × states), then compare it against the design reference and the
expected behavior, surfacing visual/layout defects with root-cause prompts. It does
NOT do the full design-source cross-check (that stays in `implementation-verifying`,
which calls this skill).

**Files:** `skills/visual-verification/SKILL.md` (+ optional `failure-modes.md`).

**`SKILL.md` frontmatter:**
- `name: visual-verification`
- `description: Use when UI work is implemented and you need to confirm the rendered output matches the design — captures a screenshot matrix across viewports and states from a running dev server / component explorer and compares against the reference, before claiming a UI change is done`

**Content outline:**
1. **Announce** + core principle: *"A spec you read is not the output you ship.
   Render it and look."*
2. **Pick a render surface** (tool-agnostic): a component explorer story, an app
   route, or a harness, served by a running dev server.
3. **Capture tool** (tool-agnostic — state the *contract*, not a tool): capture
   N viewports × M states to a known directory. List the kinds of tools that can
   satisfy it (a preview/automation MCP, a headless-browser skill, a connected
   browser) without hard-coding one.
4. **Viewport matrix:** at minimum mobile / tablet / desktop, plus any breakpoints
   the design defines. **State matrix:** the meaningful variants (empty, min, max,
   with/without each optional element, long-content, error/edge states).
5. **Compare** against the design reference frame and expected behavior. When layout
   looks wrong, **inspect computed styles** (e.g. `getComputedStyle`) to find the
   root cause — do not guess.
6. **Output:** a findings list (defect, viewport/state, suspected root cause, fix
   direction); feed fixes back, re-capture, confirm.
7. **`failure-modes.md`** — a short reference of *universal* rendered-layout
   failure modes (true in any CSS project, not project-specific):
   - A flex item with `flex: 0 0 auto` and a percentage-width child has no definite
     size — the item fills its container (a "list" then shows only one item).
   - `align-items: stretch` (the flex default) stretches media/children along the
     cross axis to full width.
   - An absolutely-positioned only-child collapses a parent's shrink-to-fit width.
   - A container with no shared page layout has no gutters/grid and sits flush to the
     viewport edges.

**Acceptance:** a pressure subagent given an implemented UI + a running explorer
produces a viewport×state screenshot matrix and identifies a *planted* layout defect
with its root cause — and does NOT declare the UI done from reading source alone.

---

## Workstream 2 — Enhance `implementation-verifying` (frontend-domain)

**File:** `skills/implementation-verifying/SKILL.md` (edit).

**Changes (all phrased as generic principles — no project component is named):**
1. **Invoke `visual-verification`.** Add a required step between "read the
   implementation" and "compare": render it and run `visual-verification` across
   viewports and states; the gap table must include rows sourced from the rendered
   comparison, not only from reading design nodes. Add "running dev server /
   component explorer" to the Prerequisites.
2. **a11y semantic checklist** (new section): every semantic wrapper has content or
   is conditionally rendered (no empty semantic elements); decorative images are
   forced to empty alt **at the component level** (driven by a prop), not only in
   fixtures; controls have accessible names; heading hierarchy is correct and
   configurable; regions are labelled where the pattern needs it.
3. **"Reuse before rebuild" check** (new gap-table row-class): before accepting any
   hand-rolled markup/style for a common UI primitive (surface/card, button, link,
   list/carousel, image, input), confirm no component-library component covers it.
   The skill carries the *principle*; the consumer's agent guide carries the
   *concrete catalog*.
4. **Layout-primitive check** (new row-class): confirm the block/page container uses
   the project's shared layout primitive (grid/gutters/rhythm) rather than a bare
   flex element. Principle here; specifics in the consumer's agent guide.

**Acceptance:** a pressure subagent verifying a feature that has (a) a layout defect
visible only when rendered, (b) an empty semantic wrapper, (c) a hand-rolled
primitive a component library covers, and (d) a bare container with no shared
layout — flags all four. Without the edits it flags none.

---

## Workstream 3 — Plan ↔ code sync (generic)

**Files:** `skills/subagent-driven-development/SKILL.md` and
`skills/executing-plans/SKILL.md` (edit).

**Change:** add to both — *the plan is a living artifact. When execution diverges
from the plan (a task's approach changes, a bug forces a redesign, a review changes
a decision), update the plan (and spec, if affected) to match before moving on.*
Add it to the per-task loop (subagent-driven) and to the batch checkpoints
(executing-plans).

**Acceptance:** a pressure subagent that changes a task's approach mid-execution
updates the plan file rather than leaving it stale.

---

## Workstream 4 — `superdeveloper` orchestrator (the "agent")

**Realization decision (important):** In Claude Code, a Task-tool *subagent* runs in
isolation and cannot pause for interactive human gates. An NNAI-style "developer
agent" is a *main-thread persona*, not an isolated subagent. The faithful Claude
Code equivalent of a gated end-to-end workflow is therefore an **orchestrator skill
that runs in the main conversation**, not a Task subagent. So `superdeveloper` is
`skills/superdeveloper/SKILL.md`. (Per-task implementation still fans out to Task
subagents via `subagent-driven-development`; only the orchestration and gates stay
in the main thread.)

**File:** `skills/superdeveloper/SKILL.md`.

**`SKILL.md` frontmatter:**
- `name: superdeveloper`
- `description: Use to drive a feature end-to-end (intake → spec → plan → implementation → verification → branch finish) as one orchestrated workflow that chains the Superpowers skills with explicit human gates`

**Workflow (delegates; never re-implements):**

| Phase | Delegates to | Gate |
|---|---|---|
| Intake | read the ticket / design / component-library sources into context | — |
| Design | `brainstorming` → spec | **Human: approve spec** |
| Plan | `writing-plans` | **Human: "go"** |
| Implement | `subagent-driven-development` (v6: TDD + single task-reviewer review + broad final review + plan-sync) | per-task pause-by-default (per that skill) |
| Verify | `implementation-verifying` (which runs `visual-verification`) | findings fixed before proceeding |
| Review | `receiving-code-review` on any feedback | — |
| Finish | `finishing-a-development-branch` | **Human: PR/merge choice** |

**Reliability rules (baked in):**
- **Never stall silently.** Auto-progress between phases. The ONLY stops are the
  named human gates above; emit them explicitly. Halting mid-phase without a named
  gate is a defect.
- **Pin the orchestrator's model** (no mid-run switch of the orchestration model);
  let each delegated skill tier its own per-task models — SDD's Model Selection picks
  cheap/standard/capable per task, which is a cost feature, not mid-run switching.
- **After implementation, always proceed to verification** — never present a "done"
  summary that skips verification.
- **Distinguish intentional gates from accidental stalls** and say which is which.
- **Branch hygiene:** work on a feature branch; never the default branch.

**What it is NOT:** a re-implementation of TDD, review, or verification logic; a
silent autonomous runner; a replacement for the individual skills (each stays
independently invocable).

**Acceptance:** a pressure subagent told to "deliver this with superdeveloper" runs
intake→spec, stops at the spec gate, and — when nudged with ambiguity after
implementation — proceeds to verification on its own rather than stopping, while
still stopping at the named human gates.

---

## Out of scope — project-specific knowledge

Per `writing-skills`, project specifics never go in skills. Each consuming project
documents, in its own agent guide (e.g. `AGENTS.md`), the concrete answers the
generic checks resolve to: its component-library catalog (so "reuse before rebuild"
resolves to real components), its layout primitive (so the layout check resolves),
its a11y conventions, and any framework/component gotchas. A consumer may keep a
companion spec for these; this spec does not assume or name any particular consumer.

---

## Phasing (for a fresh execution session)

1. **Phase 1 (highest value):** W1 `visual-verification` + W2 `implementation-
   verifying` edits — fixes the decisive gap and the a11y/reuse/layout misses.
2. **Phase 2 (small):** W3 plan-sync edits.
3. **Phase 3:** W4 `superdeveloper` orchestrator (depends on W1/W2 existing).

Each phase is independently shippable. All skill creation/edits follow
`writing-skills` (RED baseline → write → GREEN verify → refactor) and should be
validated against the fork's existing skill tests under `tests/`.

---

## References

- Authoring method: `writing-skills` (skill TDD) + `anthropic-best-practices.md`.
- Skills edited: `implementation-verifying`, `subagent-driven-development`,
  `executing-plans`. New: `visual-verification`, `superdeveloper`.
