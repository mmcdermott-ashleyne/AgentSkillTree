---
name: plan
description: "Plan the current feature (feature-first). Resolve registry/spec, detect Mode, and write tasks." 
metadata:
  short-description: "Plan tasks + criteria (feature-first, git/local safe)"
---

# plan (feature-first)

You are planning the current feature. This skill must work independently and be safe in git and non-git folders.

## Inputs
- Feature identity (preferred): `FEATURE_ID` or `FEATURE` or prompt `Feature: <name>`
- Back-compat: `AGENT_ID` / `AGENT_SLUG` (warn if used)
- `.dev-docs/features/REGISTRY.json`
- `.dev-docs/features/CURRENT_FEATURE.txt`
- `.dev-docs/commands.md`
- `AGENTS.md`

## Workflow
1) Universal preflight:
   - `git --version` (best-effort)
   - `git rev-parse --is-inside-work-tree` (best-effort)
   - Determine `EFFECTIVE_MODE` = `git` if both succeed, else `local`.

2) Resolve feature (feature-first):
   - Use resolution order from `.dev-docs/design/FEATURE_SYSTEM.md`.
   - If feature missing and a feature name was provided, auto-create registry entry using `$feature-create` rules.
   - Update `.dev-docs/features/CURRENT_FEATURE.txt` to the resolved `feature_id`.

3) Resolve mapping:
   - From registry: `feature_id`, `name`, `agent_id`, `slug`, `branch`, `worktree`.
   - Validate path-safe values.

4) Determine spec source (in this precedence order):
   A) `.dev-docs/features/<feature_id>/SPEC.md` if present and non-empty.
   B) If missing, create from template and use it.
   C) If still missing, use the user prompt text.

5) Write/refresh TASKS at `.dev-docs/context/agents/<agent_id>/TASKS.md`:
   Must include:
   - `## Controls`:
     - Mode: `auto` (default) or explicit
     - Feature ID / Name
     - Agent ID / Slug
     - Branch (or None)
     - Worktree (or None)
     - Spec path
   - `## Feature Source` with Spec path used and prompt note
   - `## Current Goal` (1-2 sentences)
   - `## Acceptance Criteria` (testable bullets)
   - `## Plan` (ordered checklist)
   - `## Test Plan`
   - `## Risks / Open Questions` (optional)

6) If `EFFECTIVE_MODE=local`, set Branch/Worktree to `None` in TASKS.

## Output
After writing TASKS, print EXACTLY these 2 lines and nothing else:
?? Plan saved to `.dev-docs/context/agents/<agent_id>/TASKS.md`
?? Next: run `$build`
