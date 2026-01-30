---
name: bootstrap-agents
description: "Create or refresh AGENTS.md + .dev-docs baseline (feature registry, templates, commands, checklists) so Codex can run end-to-end."
metadata:
  short-description: "Bootstrap feature-first Codex OS (git/local safe)"
---
You are bootstrapping this repository so Codex can operate end-to-end with a feature-first workflow. This must be safe and idempotent.

## Goals
- Create/update `AGENTS.md` and `.dev-docs` scaffolding.
- Establish the feature registry and current feature pointer.
- Seed templates for feature specs, tasks, and PRs.
- Keep content ASCII and safe for OneDrive.

## Non-negotiables
- Do not modify application code in this skill.
- Do not run git operations beyond preflight.
- Re-running must not duplicate content.

## Workflow
1) Universal preflight (record only):
   - `git --version` (best-effort)
   - `git rev-parse --is-inside-work-tree` (best-effort)
   - Determine `EFFECTIVE_MODE` = git if both succeed, else local.

2) Ensure directories exist:
   - `.dev-docs/`, `.dev-docs/design/`, `.dev-docs/features/_template/`, `.dev-docs/context/agents/default/`, `.github/`

3) Write or update the docs listed below using the templates in this repo.
   - Keep content deterministic and idempotent.

## Files to create/update (repo-relative)
- `AGENTS.md`
- `.dev-docs/design/FEATURE_SYSTEM.md`
- `.dev-docs/features/REGISTRY.json`
- `.dev-docs/features/CURRENT_FEATURE.txt`
- `.dev-docs/features/_template/SPEC.md`
- `.dev-docs/features/_template/STATUS.json`
- `.dev-docs/commands.md`
- `.dev-docs/review-checklist.md`
- `.dev-docs/context/agents/default/TASKS.md`
- `.dev-docs/context/agents/default/WORKING.md`
- `.dev-docs/context/agents/default/PR_DRAFT.md`
- `.github/pull_request_template.md`

## Output
After writing files, print EXACTLY these 4 lines and nothing else:
? AGENTS.md created/updated
?? .dev-docs baseline created/updated
?? PR template created/updated
?? Next: run `$feature-create` on your target feature
