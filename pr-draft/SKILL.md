---
name: pr-draft
description: "Generate a PR-ready draft from TASKS, diff, and review status. Feature-first and git-aware."
metadata:
  short-description: "Draft PR from TASKS + diff (feature-first)"
---
# pr-draft

Prepare a PR draft for the current feature.

## Inputs
- Feature identity (`FEATURE_ID` / `FEATURE` / prompt `Feature: <name>`)
- `.dev-docs/context/agents/<agent_id>/TASKS.md`
- `.dev-docs/context/agents/<agent_id>/WORKING.md` (optional)
- `.github/pull_request_template.md` (optional)

## Workflow
1) Universal preflight:
   - `git --version` (best-effort)
   - `git rev-parse --is-inside-work-tree` (best-effort)
   - Determine `EFFECTIVE_MODE` = `git` if both succeed, else `local`.

2) Resolve feature and mapping.
   - Ensure TASKS exists (create from template if missing).
   - Read `Mode` from TASKS; if `Mode: auto`, use `EFFECTIVE_MODE`.

3) Capture repo state (git mode only):
   - Current branch: `git rev-parse --abbrev-ref HEAD`
   - Base branch: default via `gh repo view`, else `main`
   - Diff stat: `git diff --stat`
   - Cleanliness: `git status --porcelain`

4) From TASKS, extract:
   - Current Goal
   - Acceptance Criteria
   - Plan status
   - Review Findings (if present)

5) Determine PR title:
   - Prefer `feat|fix|chore(<agent_id>): <goal summary>`.

6) Compose PR body:
   - Use `.github/pull_request_template.md` if present.
   - Otherwise fill fallback sections: Summary, Why, What Changed, How Tested, Risks, Rollout/Rollback, Checklist.

7) Write `.dev-docs/context/agents/<agent_id>/PR_DRAFT.md` with:
   - `# PR Draft`
   - `## Title`
   - `## Body`
   - `## Metadata` (Branch, Base, Diff Stat, Working Tree Clean)

## Output
After writing the draft file, print EXACTLY these 2 lines and nothing else:
?? PR draft saved to `.dev-docs/context/agents/<agent_id>/PR_DRAFT.md`
?? Next: run `$pr-open` (if Mode=git)
