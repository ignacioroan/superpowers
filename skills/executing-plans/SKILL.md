---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute one task at a time, and pause for explicit human authorization before continuing unless your human partner clearly asked for an uninterrupted run.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Tell your human partner that Superpowers works much better with access to subagents. The quality of its work will be significantly higher if run on a platform with subagent support (such as Claude Code or Codex). If subagents are available, use superpowers:subagent-driven-development instead of this skill.

## Mandatory Execution Rule

Every task ends with a mandatory pause unless your human partner explicitly asked you to do the full run without stopping.

- After completing a task, stop.
- Report which task was completed using its ordinal position within the current plan when that structure exists, for example `Completed Task 2 of 4: Rename CLI command`.
- If the current work does not have explicitly numbered tasks, report by descriptive name only with no invented numbers, for example `Completed: Update docs wording` and `Next: Run verification`.
- If another task remains, identify the next task by ordinal and name when the current plan is explicitly numbered. Otherwise, identify it by descriptive name only. In both cases, briefly state what it covers.
- If the completed task is the last task, provide a concise summary of the work completed across the full task list.
- Before considering a task complete, review any worktrees you used and leave them in an intentional state. Remove only worktrees you created and own. If a worktree is host-managed or should be preserved, still ensure you are not unintentionally keeping a branch alive.
- **Sync the plan at each checkpoint.** If execution diverged from the plan,
  update the plan (and spec if affected) before continuing. The plan must always
  describe what was actually built.
- Wait for explicit user authorization before starting the next task or wrap-up step.

This pause is mandatory even if the next step seems obvious.

**Exception:** Skip the pause only when your human partner explicitly asks for uninterrupted execution, for example "do the whole plan in one pass" or "finish all remaining tasks without stopping." Vague encouragement is not enough.

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Create TodoWrite and proceed

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed
5. If more tasks remain, report completion plus the next task and wait for explicit authorization before proceeding
6. If this was the last task, report a concise full-plan summary and wait for explicit authorization before Step 3 unless the user explicitly requested uninterrupted execution

### Step 3: Complete Development

After all tasks complete and verified, and after any required authorization pause:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Stop when blocked, don't guess
- Pause after each completed task unless your human partner explicitly requested uninterrupted execution
- Treat vague encouragement as insufficient to skip the pause
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **superpowers:using-git-worktrees** - Ensures isolated workspace (creates one or verifies existing)
- **superpowers:writing-plans** - Creates the plan this skill executes
- **superpowers:finishing-a-development-branch** - Complete development after all tasks
