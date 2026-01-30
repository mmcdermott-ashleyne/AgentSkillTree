---
name: pr-open
description: "Mode-aware PR open for current feature: commit (if needed) + push + open PR via gh."
metadata:
  short-description: "Worktree-aware commit + push + open PR (feature-first)"
---

# pr-open

Open a PR for the current feature branch. This skill is end-to-end: commit + push + open PR.

## Inputs
- Feature identity (`FEATURE_ID` / `FEATURE` / prompt `Feature: <name>`)
- `.dev-docs/context/agents/<agent_id>/TASKS.md` (for Mode)
- `.dev-docs/context/agents/<agent_id>/PR_DRAFT.md` (for PR title/body)

## Requirements
- Git installed and available on PATH.
- Remote `origin` configured.
- GitHub CLI `gh` installed and authenticated.

## Workflow
1) Universal preflight:
   - `git --version` and `git rev-parse --is-inside-work-tree` must succeed.
   - If not, `ERROR: not in a git repo; Mode=local`.

2) Resolve feature and mapping.
   - Ensure TASKS exists.
   - Read `Mode` from TASKS; if `Mode: auto`, treat as `git` here.
   - If `Mode: local`, stop: `ERROR: Mode=local; refusing to commit/push/open PR`.

3) Determine repo root and agent worktree:
   - `REPO_ROOT = git rev-parse --show-toplevel`.
   - `AGENT_WT = <REPO_ROOT>/<worktree>` from registry.

4) Choose working directory:
   - If `AGENT_WT` exists, verify it is a valid worktree and use it.
   - If missing, stop with ERROR and instruct to run `$agent-init`.

5) Preflight checks (from WORKDIR):
   - `git remote get-url origin` must succeed.
   - `gh --version` and `gh auth status` must succeed.

6) Detect current branch and base:
   - `HEAD_BRANCH = git rev-parse --abbrev-ref HEAD`
   - `BASE_BRANCH = gh repo view --json defaultBranchRef -q .defaultBranchRef.name` else `main`
   - If `HEAD_BRANCH == BASE_BRANCH`, stop with ERROR.

7) Prepare PR title/body:
   - From `PR_DRAFT.md` if present.
   - Fallback title: `chore(<agent_id>): <branch>`.
   - Body fallback includes Summary/How Tested/Risks.
   - Write body to `.dev-docs/.tmp/PR_BODY_<agent_id>.md` inside WORKDIR.

8) Commit if needed:
   - If `git status --porcelain` not empty:
     - `git add -A`
     - `git commit -m "<title>"`

9) Push:
   - If upstream not set: `git push -u origin <branch>`
   - Else: `git push`

10) Open PR:
   - Try `gh pr create --base "<base>" --head "<branch>" --title "<title>" --body-file "<bodyfile>"`.
   - If already exists, locate with `gh pr view --head "<branch>" --json url -q .url`.

## Output
If successful, print EXACTLY these 2 lines and nothing else:
?? PR opened: <PR_URL>
?? Branch pushed: <BRANCH>
