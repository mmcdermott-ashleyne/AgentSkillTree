---
name: build
description: "Implement the current plan from TASKS (feature-first). Git-aware; no git ops in local mode."
metadata:
  short-description: "Build feature (feature-first, git/local safe)"
---
# build

Implement the plan for the current feature. Safe to run in git or non-git folders.

## Inputs
- Feature identity (`FEATURE_ID` / `FEATURE` / prompt `Feature: <name>`)
- `.dev-docs/context/agents/<agent_id>/TASKS.md`
- `.dev-docs/commands.md`
- `AGENTS.md`

## Workflow
1) Universal preflight:
   - `git --version` (best-effort)
   - `git rev-parse --is-inside-work-tree` (best-effort)
   - Determine `EFFECTIVE_MODE` = `git` if both succeed, else `local`.

2) Resolve feature and mapping (see feature resolution rules).
   - Ensure TASKS exists (create from template if missing).
   - Read `Mode` from TASKS; if `Mode: auto`, use `EFFECTIVE_MODE`.
   - If `Mode: git` but git is not available, stop with ERROR.

3) In `Mode: git`:
   - Ensure inside a git worktree.
   - Ensure the resolved worktree exists.
     - If missing, stop with ERROR and instruct to run `$agent-init`.
   - Determine current branch: `git rev-parse --abbrev-ref HEAD`.
   - Determine base branch: prefer `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`, else `main`.
   - Refuse base branch: if on base, stop with ERROR.
   - Ensure branch matches registry branch; if not, stop with ERROR unless explicitly overridden in TASKS.

4) In `Mode: local`:
   - Do not create/switch branches.
   - Work only in current directory.

5) Execute tasks from TASKS:
   - Implement code changes per plan.
   - Add or update tests.
   - Update docs if needed.
   - Update TASKS checklist and add new tasks with reasons.

6) Optional quick sanity checks using `.dev-docs/commands.md`.

## Output
When implementation is complete, print EXACTLY these 3 lines and nothing else:
??? Build complete (code + tests updated)
?? TASKS updated at `.dev-docs/context/agents/<agent_id>/TASKS.md`
?? Next: run `$review`
