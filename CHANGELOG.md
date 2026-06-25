# Changelog

Fork-specific changes tracked here to make future upstream syncs easier to review.

## 2026-06-25

- Adopted the upstream v6 subagent-driven-development rework: single task reviewer, file handoffs via `.superpowers/sdd/` workspace, progress ledger, new `task-reviewer-prompt.md` replacing the two separate spec/quality reviewers.
- Re-applied the fork's pause-by-default rule on the new v6 execution loop: mandatory pause after each task, uninterrupted runs only when human partner explicitly requests them.
- Added a plan-sync rule to `subagent-driven-development/SKILL.md` and `executing-plans/SKILL.md`: when execution diverges from the plan (bug forces a redesign, review changes a decision), the agent updates the plan file before marking the task complete or continuing.
- Adopted upstream v6 `writing-skills` additions (form-to-failure table, micro-test wording section, two new checklist items) and `writing-plans` additions (task right-sizing, global constraints section, interfaces block in task template).
- Applied small correctness fixes from upstream v6: `systematic-debugging` keyword in `using-superpowers`, markdown link fix in `test-driven-development`, hyphen fix in `systematic-debugging`.

## 2026-06-05

- Added this changelog to record dated changes made in the fork.
- Updated `skills/subagent-driven-development/SKILL.md` so the default behavior is to pause after each completed task and wait for explicit authorization before continuing. Added a narrow exception for uninterrupted runs only when the human partner clearly asks for that behavior.
- Updated `skills/executing-plans/SKILL.md` to match the same default pause behavior between tasks and before branch-finishing work unless the human partner explicitly requests uninterrupted execution.
- Clarified in both pause-by-default skills that when work is not structured as explicitly numbered tasks, the agent must report by descriptive name only and must not invent ordinal numbering.
- Refined both mandatory pause rules so the "next task" line preserves numbered-plan behavior while explicitly falling back to descriptive names only for non-numbered work, and normalized the authorization wording to "next task or wrap-up step".
