# Changelog

Fork-specific changes tracked here to make future upstream syncs easier to review.

## 2026-06-05

- Added this changelog to record dated changes made in the fork.
- Updated `skills/subagent-driven-development/SKILL.md` so the default behavior is to pause after each completed task and wait for explicit authorization before continuing. Added a narrow exception for uninterrupted runs only when the human partner clearly asks for that behavior.
- Updated `skills/executing-plans/SKILL.md` to match the same default pause behavior between tasks and before branch-finishing work unless the human partner explicitly requests uninterrupted execution.
- Clarified in both pause-by-default skills that when work is not structured as explicitly numbered tasks, the agent must report by descriptive name only and must not invent ordinal numbering.
- Refined both mandatory pause rules so the "next task" line preserves numbered-plan behavior while explicitly falling back to descriptive names only for non-numbered work, and normalized the authorization wording to "next task or wrap-up step".
