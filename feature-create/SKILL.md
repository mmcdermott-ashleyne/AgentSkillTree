---
name: feature-create
description: "Create or update a feature registry entry and seed feature files. Feature-first, safe, idempotent."
metadata:
  short-description: "Create feature + spec + registry"
---
# feature-create

Create or update a feature in the registry, seed its spec/status files, and set it as current.

## Inputs
- Feature identity (required):
  - `FEATURE_ID` or `FEATURE` env
  - Prompt line `Feature: <name>` or `feature=<name>`
- Optional overrides:
  - `AGENT_ID`, `AGENT_SLUG`

## Conventions
- Registry: `.dev-docs/features/REGISTRY.json`
- Current pointer: `.dev-docs/features/CURRENT_FEATURE.txt`
- Feature spec: `.dev-docs/features/<feature_id>/SPEC.md`
- Feature status: `.dev-docs/features/<feature_id>/STATUS.json`
- Branch: `feat/<agent_id>/<slug>`
- Worktree: `.worktrees/agent-<agent_id>`

## Workflow
1) Universal preflight (record only):
   - `git --version` (best-effort)
   - `git rev-parse --is-inside-work-tree` (best-effort)
   - Determine `EFFECTIVE_MODE` = git if both succeed, else local.

2) Resolve feature identity:
   - Use explicit feature inputs.
   - If missing, stop with `ERROR: feature identity required`.

3) Normalize and validate:
   - Derive `feature_id` (lower-kebab, ASCII, no spaces).
   - Derive `agent_id` from override or `feature_id`.
   - Derive `slug` from override or `feature_id`.
   - Validate `feature_id`, `agent_id`, `slug` are path-safe (no `/`, `\`, `..`, `:`, whitespace).

4) Load or initialize registry:
   - If registry missing, create with defaults.

5) Upsert registry entry:
   - Preserve existing fields if present.
   - Update `name`, `agent_id`, `slug`, `branch`, `worktree`, `updated_at`.
   - If missing, set `created_at`.

6) Ensure feature files:
   - Create `.dev-docs/features/<feature_id>/` if missing.
   - Create `SPEC.md` from template if missing.
   - Create `STATUS.json` from template if missing.

7) Update current feature pointer:
   - Write `feature_id` to `.dev-docs/features/CURRENT_FEATURE.txt`.

## Output
After completion, print EXACTLY these 3 lines and nothing else:
FEATURE_ID=<feature_id>
FEATURE_SPEC=.dev-docs/features/<feature_id>/SPEC.md
FEATURE_CURRENT_SET=true
