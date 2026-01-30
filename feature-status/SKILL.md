---
name: feature-status
description: "Show resolved feature mapping and paths. No changes."
metadata:
  short-description: "Show feature mapping"
---
# feature-status

Display the resolved feature mapping and paths without modifying files.

## Inputs
- Feature identity (optional):
  - `FEATURE_ID` or `FEATURE` env
  - Prompt line `Feature: <name>` or `feature=<name>`
  - Fallback: `.dev-docs/features/CURRENT_FEATURE.txt`

## Workflow
1) Universal preflight (record only):
   - `git --version` (best-effort)
   - `git rev-parse --is-inside-work-tree` (best-effort)
   - Determine `EFFECTIVE_MODE` = git if both succeed, else local.

2) Resolve feature:
   - Use explicit inputs or current pointer.
   - If missing, stop with `ERROR: no feature selected`.

3) Load registry entry and print mapping:
   - feature_id, name, agent_id, slug, branch, worktree
   - spec path, status path, tasks path
   - mode = EFFECTIVE_MODE

## Output
After completion, print EXACTLY these 7 lines and nothing else:
FEATURE_ID=<feature_id>
FEATURE_NAME=<name>
AGENT_ID=<agent_id>
BRANCH=<branch or none>
WORKTREE=<worktree or none>
SPEC=.dev-docs/features/<feature_id>/SPEC.md
MODE=<git|local>
