---
name: feature-select
description: "Select an existing feature as current without modifying registry data."
metadata:
  short-description: "Set CURRENT_FEATURE"
---
# feature-select

Set the current feature pointer to an existing feature.

## Inputs
- Feature identity (required):
  - `FEATURE_ID` or `FEATURE` env
  - Prompt line `Feature: <name>` or `feature=<name>`

## Workflow
1) Universal preflight (record only):
   - `git --version` (best-effort)
   - `git rev-parse --is-inside-work-tree` (best-effort)
   - Determine `EFFECTIVE_MODE` = git if both succeed, else local.

2) Resolve feature:
   - Use explicit feature inputs.
   - Validate the feature exists in `.dev-docs/features/REGISTRY.json`.
   - If missing, stop with `ERROR: feature not found; run $feature-create`.

3) Update pointer:
   - Write `feature_id` to `.dev-docs/features/CURRENT_FEATURE.txt`.

## Output
After completion, print EXACTLY these 2 lines and nothing else:
FEATURE_CURRENT=<feature_id>
FEATURE_CURRENT_SET=true
