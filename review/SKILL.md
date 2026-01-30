---
name: review
description: "Perform CI-parity self-review for the current feature. Feature-first, git/local safe."
metadata:
  short-description: "Self-review + fix issues (feature-first)"
---
# review

Review the current feature for PR readiness. Must work independently.

## Inputs
- Feature identity (`FEATURE_ID` / `FEATURE` / prompt `Feature: <name>`)
- `.dev-docs/context/agents/<agent_id>/TASKS.md`
- `.dev-docs/review-checklist.md`
- `.dev-docs/commands.md`

## Workflow
1) Universal preflight:
   - `git --version` (best-effort)
   - `git rev-parse --is-inside-work-tree` (best-effort)
   - Determine `EFFECTIVE_MODE` = `git` if both succeed, else `local`.

2) Resolve feature and mapping.
   - Ensure TASKS exists (create from template if missing).
   - Read `Mode` from TASKS; if `Mode: auto`, use `EFFECTIVE_MODE`.
   - If `Mode: git` but git is not available, stop with ERROR.

3) Capture repo state (git mode only):
   - `git status --porcelain`
   - `git diff --stat`
   - `git diff` (focus on high-risk areas)

4) Run CI-parity checks using `.dev-docs/commands.md`:
   - format, lint, typecheck, tests, build (as applicable)
   - If `Unknown`, pick best-effort defaults and update `.dev-docs/commands.md`.

5) Apply `.dev-docs/review-checklist.md`:
   - Identify and fix issues.

6) Update TASKS:
   - Mark completed tasks.
   - Add `## Review Findings` with issues found and resolved.

7) Ensure working tree is clean or document why remaining changes are intentional.

## Output
When review is complete, print EXACTLY these 4 lines and nothing else:
? Review complete (checks run)
?? Tests/lint/typecheck status recorded
?? TASKS updated at `.dev-docs/context/agents/<agent_id>/TASKS.md`
?? Next: prepare PR (use `$pr-draft` then `$pr-open` if Mode=git)
