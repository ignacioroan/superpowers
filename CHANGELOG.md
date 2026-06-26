# Changelog

Fork-specific changes tracked here to make future upstream syncs easier to review.

## 2026-06-26 — Feature-delivery skills

Executed `docs/plans/2026-06-25-feature-delivery-improvements.md` (on the post-v6
baseline). Skills authored/edited with the `writing-skills` method; UI behaviors
validated against a real throwaway fixture rendered via the preview MCP.

- **New skill `visual-verification`** — renders the implementation and captures a
  viewport × state matrix, compares to the design, and traces layout defects to
  computed styles (`failure-modes.md` reference). RED/GREEN validated: against a
  fixture whose source reads like a row but renders as a column (mobile-first base +
  missing desktop override), a no-skill baseline declared it shippable from source;
  with the skill the agent caught the column-not-row defect and root-caused it.
- **Enhanced `implementation-verifying`** — added a render step (invokes
  `visual-verification`), a Running-UI prerequisite, an accessibility checklist, and
  reuse-before-rebuild + layout-primitive gap-table rows. Adopted as *formalization*:
  the planned RED baseline did not hold (a thorough agent already caught the four
  planted defects by reading source, incl. tracing the cascade), so the value is
  making those checks explicit/reliable across agents and adding the render step for
  defects not deducible from source. Render behavior itself is validated at the
  `visual-verification` level.
- **New skill `superdeveloper`** — a thin main-thread orchestrator chaining
  brainstorming → writing-plans → subagent-driven-development → implementation-verifying
  → finishing-a-development-branch with explicit human gates. v6-reconciled (honors
  SDD pause-by-default; pins only the orchestrator's model; distinguished from the
  `using-superpowers` bootstrap). RED/GREEN validated: baseline skipped verification
  and jumped to PR after implementation; with the skill the agent proceeds to the
  Verify phase first.

## 2026-06-25 — Upstream v6.0.3 sync

Selective, manual port of the mother project's `v6.0.3` into the fork. Plan and
rationale: `docs/plans/2026-06-25-upstream-v6-sync.md` and
`docs/specs/2026-06-25-upstream-v6-sync-design.md`.

**Subagent-Driven Development (Tier 2):**

- Adopted the upstream v6 SDD rework: a single `task-reviewer-prompt.md` (spec + quality in one pass) replacing the separate `spec-reviewer-prompt.md` and `code-quality-reviewer-prompt.md`; file handoffs and a progress ledger in a self-ignoring working-tree `.superpowers/sdd/` workspace (the v6.0.3 fix that keeps artifacts out of the protected `.git/` path) via new `scripts/task-brief`, `scripts/review-package`, and `scripts/sdd-workspace`; pre-flight plan review; explicit per-dispatch model selection.
- **Re-applied the fork's pause-by-default rule on the new v6 execution loop** (RED/GREEN validated): mandatory pause after each task, with uninterrupted runs only when the human partner explicitly asks. This re-inverts upstream's "continuous execution".
- Added a **plan-sync** rule to `subagent-driven-development` and `executing-plans`: when execution diverges from the plan (a bug forces a redesign, a review changes a decision), the agent updates the plan file before marking the task complete or continuing.

**Skill-authoring improvements (Tier 1):**

- `writing-skills`: added the "Match the Form to the Failure" table and the "Micro-Test Wording" section, plus two new checklist items (the fork's CSO naming and `references/` set were intentionally kept — no wholesale vendor-neutral rewrite).
- `writing-plans`: added Task Right-Sizing, a Global Constraints template block, and a per-task Interfaces block.
- Correctness fixes: `systematic-debugging` "Ultrathink"→"Ultra-think" (stops forcing extended thinking); `test-driven-development` `@`-link → markdown link; `using-superpowers` bootstrap now names `systematic-debugging` instead of the nonexistent `debugging` skill.

**Security-sensitive merges (Tier 3) — fork hardening preserved:**

- `using-git-worktrees`: adopted v6 step renumbering and dropped the legacy `~/.config/superpowers/worktrees/` global path; **kept the installer security gate**.
- `finishing-a-development-branch`: made PR creation forge-neutral (removed the hardcoded `gh pr create` block) and dropped the legacy global worktree path; **kept the push/PR confirmation gate**.
- `requesting-code-review/code-reviewer.md`: adopted the v6 placeholder (`{}`→`[]`) and read-only/skeptical-reviewer wording (used by the v6 SDD final review); **kept the Untrusted Input Warning**.
- `hooks/session-start`: adopted the v6 Windows EPIPE robustness (`printf … | cat`); **kept the fork's `escape_for_json` and Copilot/OpenCode platform detection**.

**Deliberately NOT adopted** (contradict the fork's vision or out of scope): the brainstorming visual-companion server and its v6 auth hardening (the fork removed the server for security); all Codex artifacts; the evals submodule + "drill"; the Kimi/Pi/Antigravity harnesses; the README/PR/issue-template/governance changes; the shell-lint + pre-commit tooling.

## 2026-06-05

- Added this changelog to record dated changes made in the fork.
- Updated `skills/subagent-driven-development/SKILL.md` so the default behavior is to pause after each completed task and wait for explicit authorization before continuing. Added a narrow exception for uninterrupted runs only when the human partner clearly asks for that behavior.
- Updated `skills/executing-plans/SKILL.md` to match the same default pause behavior between tasks and before branch-finishing work unless the human partner explicitly requests uninterrupted execution.
- Clarified in both pause-by-default skills that when work is not structured as explicitly numbered tasks, the agent must report by descriptive name only and must not invent ordinal numbering.
- Refined both mandatory pause rules so the "next task" line preserves numbered-plan behavior while explicitly falling back to descriptive names only for non-numbered work, and normalized the authorization wording to "next task or wrap-up step".
