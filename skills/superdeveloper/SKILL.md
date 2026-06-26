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
| Implement | `subagent-driven-development` (v6: single task-reviewer + broad final review) | per-task pause-by-default (per that skill) |
| Verify | `implementation-verifying` (runs `visual-verification`) | fix findings before proceeding |
| Review | `receiving-code-review` on any feedback | — |
| Finish | `finishing-a-development-branch` | **Human: PR/merge choice** |

## Reliability rules (non-negotiable)

- **Never stall silently.** A legitimate stop is one of: a named human gate above,
  or a checkpoint a delegated skill defines itself — above all
  `subagent-driven-development`'s pause-by-default between tasks. Emit every stop
  explicitly and say which kind it is. Inventing a stop that is neither is a
  defect; so is suppressing a sub-skill's checkpoint (e.g. forcing an uninterrupted
  SDD run the human did not ask for). "Auto-progress" means advancing between
  phases without inventing extra stops — not overriding a sub-skill's gates.
- **After implementation, ALWAYS proceed to verification.** Never present a "done"
  summary that skips the Verify phase.
- **Pin the orchestrator's model; let delegates tier their own.** Don't flip the
  model you run the orchestration on mid-delivery. Do NOT override the per-task
  model choice each delegated skill makes — `subagent-driven-development`'s Model
  Selection deliberately picks cheap/standard/capable per task; that tiering is a
  cost feature, not the mid-run model-switching this rule forbids.
- **Name the stop.** When you pause, say whether it is an intentional human gate, a
  sub-skill checkpoint, or a blocker — never an ambiguous stop.
- **Branch hygiene.** Work on a feature branch; never the default branch.

## What this is NOT

- Not a re-implementation of TDD, review, or verification — it delegates.
- Not a silent autonomous runner — it honors the human gates.
- Not a replacement for the individual skills — each stays independently usable.
- **Not the `using-superpowers` bootstrap.** `using-superpowers` is the always-on
  router: for *any* message it makes you check for and invoke the right skill,
  one at a time, reactively. `superdeveloper` is the opposite shape — an opt-in,
  fixed pipeline for one task (end-to-end feature delivery: intake → spec → plan →
  implement → verify → finish) with explicit human gates. The bootstrap still
  applies *inside* a superdeveloper run (each phase invokes its skill through it);
  superdeveloper sits on top deciding the order and the gates.
