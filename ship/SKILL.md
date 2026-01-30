---
name: ship
description: "Run the full dev loop for the current feature with guardrails. Feature-first and git/local safe."
metadata:
  short-description: "Full dev loop orchestrator (feature-first)"
---

# ship (feature-first orchestrator)

Run the full developer loop for the current feature. Each step must also be runnable independently.

## Inputs
- Feature identity (`FEATURE_ID` / `FEATURE` / prompt `Feature: <name>`)
- `.dev-docs/context/agents/<agent_id>/TASKS.md`
- `.dev-docs/context/agents/<agent_id>/PR_DRAFT.md`
- `.dev-docs/context/agents/<agent_id>/WORKING.md`
- `.dev-docs/features/REGISTRY.json`
- `AGENTS.md`

## Controls (from TASKS)
- `Mode: auto|local|git`
- `AutoMerge: true|false` (default false)
- `Merge Method: merge|squash|rebase` (default squash)

## Guardrails
- Resolve feature and mapping before doing anything.
- Validate agent/worktree consistency; stop on mismatch.
- Respect Mode (auto/local/git) and do not run git steps in local mode.

## Workflow
1) Universal preflight:
   - `git --version` (best-effort)
   - `git rev-parse --is-inside-work-tree` (best-effort)
   - Determine `EFFECTIVE_MODE` = git if both succeed, else local.

2) Resolve feature and mapping.

3) Run `$plan` for this feature.

4) Run `$build` for this feature.

5) Run `$review` for this feature.

6) Run `$pr-draft` for this feature.

7) If Mode resolves to `git`, run `$pr-open`.

8) If Mode resolves to `git` AND `AutoMerge: true`:
   - Run `$pr-automerge-cleanup` with `MERGE_METHOD` if provided.

9) Always run `$handoff`.

## Output
When the loop is finished, print EXACTLY this line and nothing else:
? Ship loop complete
