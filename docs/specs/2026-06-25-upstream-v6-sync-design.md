# upstream-v6-sync — Design Spec

**Date:** 2026-06-25
**Status:** Draft
**Scope:** Superpowers fork (this repo). Bring selected changes from the mother
project's `v6.0.3` into the fork, *without betraying the fork's custom vision*.
**Source of truth:** the sibling checkout `/Users/NNRodrigIg/dev-box/superpowers`
(its working tree is at tag `v6.0.3`, HEAD `896224c`). The fork's baseline is
`v5.1.0`.

---

## Motivation

The fork last synced manually from upstream at `v5.1.0`. Upstream has since shipped
`v6.0.0 → v6.0.3`. A full review of `v5.1.0..v6.0.3` (commits, release notes, and
file-by-file diffs) classified every change against the fork's identity. This spec
records what we adopt, what we re-apply on top, and what we deliberately skip.

A clean `git merge`/`cherry-pick` is not possible: the fork has diverged
deliberately (Codex plugin dropped, brainstorming visual-companion server removed
for security, curated README/tests, pause-by-default execution, `implementation-*`
skills). Like the `v5.1.0` sync, this is a **manual, file-by-file** port from the
sibling checkout, re-applying the fork's customizations where they overlap.

## The fork's vision (the filter)

1. **Pause-by-default execution** — the fork inverts upstream's "continuous
   execution" in `subagent-driven-development` and `executing-plans`. Non-negotiable.
2. **`implementation-*` skills** (fork-original) + the feature-delivery roadmap
   (`visual-verification`, `superdeveloper`, plan-sync) in
   `docs/plans/2026-06-25-feature-delivery-improvements.md`.
3. **Security hardening** the fork added and must preserve:
   - `using-git-worktrees/SKILL.md` — "Security gate — BEFORE running any installer".
   - `finishing-a-development-branch/SKILL.md` — "Confirmation gate — REQUIRED before
     running any of the commands below" (push/PR).
   - `requesting-code-review/code-reviewer.md` — "Untrusted Input Warning"
     (prompt-injection).
   - `hooks/session-start` — O(n²)→parameter-substitution `escape_for_json`, plus
     Copilot/OpenCode platform-detection injection.
4. **Visual companion server removed** for security (`ea18da3` deleted `server.cjs`
   et al.). We do **not** re-introduce it.
5. **Codex plugin dropped** entirely.
6. **Curated README and curated `tests/`** — not imported wholesale from upstream.

## Adoption decisions (by tier)

### Tier 1 — clean content adopts (low risk)

- **`writing-skills/SKILL.md`** — insert two new sections only ("Match the Form to
  the Failure", "Micro-Test Wording Before Full Scenarios"), the bulletproofing
  **Scope** note, and the two new Skill-Creation-Checklist items. *Targeted
  insertion, not wholesale copy* — the fork keeps its own voice (no CSO→SDO rename)
  and its existing `references/` set (the wholesale v6 file links to
  `claude-code-tools.md`/`pi-tools.md`/`antigravity-tools.md` the fork does not ship).
  High value: this is the method that authors the feature-delivery skills.
- **`writing-plans/SKILL.md`** — insert three blocks: **Task Right-Sizing**, **Global
  Constraints** (template), **Interfaces** (per-task template). Clean inserts.
- **`systematic-debugging/SKILL.md`** — one line: `Ultrathink` → `Ultra-think`
  (stops the skill from forcing extended thinking every session).
- **`test-driven-development/SKILL.md`** — one line: `@testing-anti-patterns.md` →
  markdown link.

### Tier 2 — SDD v6 rework (high value, high effort; re-apply our customization)

Adopt the upstream rewrite of `subagent-driven-development` (single task-reviewer,
broad final review, pre-flight plan review, model-selection guidance, file handoffs,
progress ledger, read-only/skeptical reviewers, no pre-judging) **and its v6.0.3
fix** (artifacts in working-tree `.superpowers/sdd/`, not `.git/`).

- **Bring** (copy from upstream, verbatim): `task-reviewer-prompt.md`,
  `implementer-prompt.md` (v6), `scripts/task-brief`, `scripts/review-package`,
  `scripts/sdd-workspace`.
- **Delete:** `spec-reviewer-prompt.md`, `code-quality-reviewer-prompt.md` (merged
  into the single task reviewer).
- **Re-apply our pause-by-default** onto the new `SKILL.md`: replace the v6
  "Continuous execution" paragraph with the fork's "Mandatory Execution Rule",
  re-insert the flowchart pause/authorization nodes, and restore the pause-aware
  wording in "vs. Executing Plans" / "Advantages" / "Red Flags".
- **Absorb plan-sync** (feature-delivery Workstream 3, moved here): add the plan-sync
  rule to the new SDD per-task loop and to `executing-plans` checkpoints.
- Validate every behavior-shaping change with `writing-skills` (RED baseline → GREEN
  pressure-subagent → refactor).

### Tier 3 — security-sensitive merges + small fixes (careful)

- **`using-git-worktrees/SKILL.md`** — apply the v6 changes (step renumbering;
  remove the legacy `~/.config/superpowers/worktrees/` global path). **Preserve** the
  installer security gate.
- **`finishing-a-development-branch/SKILL.md`** — adopt forge-neutrality (remove the
  hardcoded `gh pr create` block; also drop the legacy global worktree path).
  **Preserve** the push/PR confirmation gate.
- **`requesting-code-review/code-reviewer.md`** — adopt the placeholder `{}`→`[]` fix
  and read-only/skeptical reviewer wording (the v6 SDD final review points here).
  **Preserve** the Untrusted Input Warning.
- **`hooks/session-start`** — optionally adopt the Windows EPIPE robustness
  (`printf … | cat`). **Preserve** the fork's `escape_for_json` and platform
  detection. Skip all Codex (`session-start-codex`, `hooks-codex.json`) parts.
- **`using-superpowers/SKILL.md`** — adopt only the bootstrap correctness fix
  (`debugging` → `systematic-debugging` in the skill-priority text). Skip the
  wholesale vendor-neutral rewrite.

## Explicitly NOT adopted (contradicts the vision or irrelevant)

- **All Brainstorming Visual Companion work** (per-session key auth, file-server
  sandbox, reconnect, idle timeout, Windows launcher, `server.cjs`/`helper.js`,
  browser auto-open, "offer companion just-in-time"). The fork deleted the companion
  server for security; re-introducing it reverses that decision. *(The fork's
  `tests/brainstorm-server/` is now orphaned — separate cleanup, out of scope here.)*
- **Codex** — native hooks, `session-start-codex`, `hooks-codex.json`, sync-to-codex,
  `.codex-plugin/`.
- **Evals submodule + "drill"** — would delete the very `tests/` suites the
  feature-delivery plan depends on and adds a submodule.
- **New harnesses** Kimi / Pi / Antigravity — outside the fork's Claude Code +
  OpenCode focus.
- **README rewrite, PR/issue templates, contributor disclosure, `docs/
  porting-to-a-new-harness.md`** — upstream-project governance; the fork curates its
  own README and CLAUDE.md.
- **Shell-lint + pre-commit + `package.json`/test reorg** — optional dev tooling;
  conflicts with the curated-tests decision. (`scripts/lint-shell.sh` could be
  rescued standalone later if wanted.)

## Phasing

1. **Phase 1 — Tier 1.** Clean content adopts. Independently shippable.
2. **Phase 2 — Tier 2.** SDD v6 + re-applied pause-by-default + absorbed plan-sync.
   The largest, behavior-shaping phase; gated by `writing-skills` validation.
3. **Phase 3 — Tier 3.** Security-sensitive merges and small fixes.

Each phase ends green and committed. After Phase 2 the fork's SDD is on the v6
baseline, so the feature-delivery plan can be executed on top.

## Acceptance

- Phase 1: the new `writing-skills`/`writing-plans` sections are present; the two
  one-line fixes are applied; no fork voice or references regressed.
- Phase 2: a pressure subagent running a multi-task plan with the new SDD **pauses
  after each task** (and runs uninterrupted only when explicitly told); the new
  scripts produce briefs/packages under `.superpowers/sdd/`; the old two reviewer
  prompts are gone; plan-sync fires when execution diverges from the plan.
- Phase 3: all four security customizations remain intact (grep anchors present);
  PR creation is forge-neutral; the bootstrap no longer names a nonexistent
  `debugging` skill.

## References

- Review source: `git -C /Users/NNRodrigIg/dev-box/superpowers diff v5.1.0..v6.0.3`
  and `RELEASE-NOTES.md`.
- Companion plan: `docs/plans/2026-06-25-upstream-v6-sync.md`.
- Downstream dependent: `docs/plans/2026-06-25-feature-delivery-improvements.md`.
