---
name: pr-automerge-cleanup
description: "Auto-merge PR via gh and fully clean up feature worktree/branch/metadata. Feature-first, safe, idempotent."
metadata:
  short-description: "Auto-merge PR + full cleanup (feature-first)"
---

# pr-automerge-cleanup (feature-first)

This skill can:
1) Ensure a PR exists (optional),
2) Auto-merge via `gh`,
3) Clean up worktree, branch, and metadata safely.

## Inputs
- Feature identity (`FEATURE_ID` / `FEATURE` / prompt `Feature: <name>`)
- Optional PR number: `PR_NUMBER` or `pr=<number>`
- Optional merge method: `MERGE_METHOD` in {merge,squash,rebase} (default squash)
- Optional auto-open behavior: `AUTO_OPEN_PR` in {true,false} (default true)
- Optional base branch override: `BASE_BRANCH` or `base=<name>`
- Optional safety override: `ALLOW_MERGE_WITHOUT_CHECKS` in {true,false} (default false)

## Controls (from TASKS)
- `Mode: local|git` (auto resolves to git here)
- `Cleanup ResetHard: true|false` (default false)

## Guardrails (non-negotiable)
- Never deletes outside `.worktrees/`.
- Never runs `git clean -fdx`.
- Never runs `git reset --hard` unless explicitly allowed.
- Only deletes remote branches via `gh pr merge --delete-branch`.

## Workflow
1) Preflight:
   - `git --version` and `git rev-parse --is-inside-work-tree` must succeed.
   - `REPO_ROOT = git rev-parse --show-toplevel`.

2) Resolve feature and mapping.
   - Ensure TASKS exists; read `Mode`.
   - If `Mode: local`, stop: `ERROR: Mode=local; refusing to auto-merge/cleanup`.

3) Validate IDs are path-safe.

4) Determine worktree:
   - `AGENT_WT = <REPO_ROOT>/<worktree>` from registry.
   - If missing, stop with ERROR (cleanup requires a known worktree path).

5) Ensure `gh` is available and authenticated.

6) Determine branch + base + PR:
   - Use WORKDIR = `AGENT_WT` if valid; else ERROR.
   - `HEAD_BRANCH = git rev-parse --abbrev-ref HEAD`.
   - `BASE_BRANCH` from input or `gh repo view`, else `main`.
   - Refuse base branch.

7) PR resolution:
   - If `PR_NUMBER` provided, validate head/base.
   - Else if `AUTO_OPEN_PR=true`, locate or create PR (use PR_DRAFT if present).
   - Else error.

8) Ensure branch is pushed (commit if needed, then push).

9) Gate merge on checks unless overridden.

10) Merge via `gh pr merge --<method> --delete-branch`.

11) Sync base branch in repo root:
   - `git fetch origin <base> --prune`
   - `git checkout <base>`
   - `git pull --ff-only origin <base>` or `git reset --hard origin/<base>` if allowed.

12) Remove local worktree safely:
   - `git worktree remove --force "<AGENT_WT>"`
   - If locked, delete only `<AGENT_WT>/.dev-docs/.tmp` and retry.

13) Prune worktree metadata:
   - `git worktree prune`

14) Delete local branch:
   - `git branch -d "<HEAD_BRANCH>"` (or `-D` if needed after merge).

## Output
If successful, print EXACTLY these 6 lines and nothing else:
?? PR merged: <PR_URL>
? Base synced: <BASE_BRANCH>
?? Worktree removed: <AGENT_WT or "none">
?? Local branch deleted: <HEAD_BRANCH or "none">
??? Remote branch deleted: yes
?? Worktree metadata pruned
