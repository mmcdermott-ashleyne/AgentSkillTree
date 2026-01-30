---
name: agent-init
description: "Feature-first init: create or reuse worktree/branch and seed per-feature context. Git-aware and idempotent."
metadata:
  short-description: "Init feature worktree + context (feature-first, safe)"
---
# agent-init (feature-first, idempotent)

Create or reuse a feature worktree/branch (git mode) and seed per-feature context. This skill is safe to rerun.

## Inputs
- Feature identity (preferred):
  - `FEATURE_ID` or `FEATURE` env
  - Prompt line: `Feature: <name>` or `feature=<name>`
  - Fallback: `.dev-docs/features/CURRENT_FEATURE.txt`
- Back-compat overrides (warn when used):
  - `AGENT_ID`
  - `AGENT_SLUG`
- Optional `MODE` (auto|local|git) via TASKS or prompt

## Conventions
- Registry: `.dev-docs/features/REGISTRY.json`
- Current pointer: `.dev-docs/features/CURRENT_FEATURE.txt`
- Branch: `feat/<agent_id>/<slug>`
- Worktree path (repo-relative): `.worktrees/agent-<agent_id>`
- Agent context root (inside worktree or local cwd): `.dev-docs/context/agents/<agent_id>/`
- Feature spec: `.dev-docs/features/<feature_id>/SPEC.md`

## Workflow
1) Universal preflight (always first):
   - Check `git --version` (best-effort).
   - Check `git rev-parse --is-inside-work-tree` (best-effort).
   - Determine `EFFECTIVE_MODE`:
     - If both succeed: `git`.
     - Else: `local`.
   - If requested Mode is `git` but git is not available: `ERROR`.

2) Resolve feature (feature-first):
   - Use resolution order from `.dev-docs/design/FEATURE_SYSTEM.md`.
   - If feature not found and a feature name was provided, auto-create a registry entry using the `$feature-create` rules (idempotent).
   - Update `.dev-docs/features/CURRENT_FEATURE.txt` to the resolved `feature_id`.

3) Resolve mapping:
   - From registry (or derived): `feature_id`, `agent_id`, `slug`, `branch`, `worktree`.
   - Validate `feature_id`, `agent_id`, `slug` are path-safe (no `/`, `\`, `..`, `:`, or whitespace).
   - If `AGENT_ID`/`AGENT_SLUG` provided and differ from registry, warn and update registry with final resolved values.

4) Git path setup (only if `EFFECTIVE_MODE=git`):
   - `REPO_ROOT = git rev-parse --show-toplevel`.
   - `WORKTREES_DIR = <REPO_ROOT>/.worktrees`.
   - `WORKTREE_PATH = <REPO_ROOT>/<worktree>` (from registry).

5) Idempotency checks:
   - If `WORKTREE_PATH` exists:
     - Verify it is a valid worktree for this repo.
     - If valid, do NOT recreate.
     - If invalid, stop: `ERROR: worktree path exists but is not a valid git worktree; refusing to overwrite`.
   - If `WORKTREE_PATH` does not exist:
     - If the feature branch exists, add worktree on that branch.
     - Else create branch from base and add worktree.
     - Base branch detection: prefer `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`, else `main`.

6) Local mode (if `EFFECTIVE_MODE=local`):
   - Do not run git commands.
   - Use the current working directory as `WORKDIR`.

7) Seed required directories/files (in WORKDIR):
   - Ensure directories:
     - `.dev-docs/context/agents/<agent_id>/history/`
     - `.dev-docs/features/<feature_id>/`
   - Ensure feature spec:
     - `.dev-docs/features/<feature_id>/SPEC.md` (create from template if missing)
     - `.dev-docs/features/<feature_id>/STATUS.json` (create if missing)
   - Ensure per-agent TASKS exists:
     - `.dev-docs/context/agents/<agent_id>/TASKS.md` (copy default template if available)
   - Ensure TASKS contains Controls with:
     - Mode
     - Feature ID/Name
     - Agent ID
     - Slug
     - Branch (or None)
     - Worktree (or None)
     - Spec path
   - Ensure WORKING.md and PR_DRAFT.md exist (copy defaults if available).

8) Output:
Print EXACTLY these two lines and nothing else:
AGENT_WORKTREE=<WORKTREE_PATH or current directory>
AGENT_BRANCH=<BRANCH or "none">
