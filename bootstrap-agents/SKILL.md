---
name: bootstrap-agents
description: "Create or refresh AGENTS.md + .dev-docs baseline (feature registry, templates, commands, checklists, retention/ledger) so Codex can run end-to-end."
metadata:
  short-description: "Bootstrap feature-first Codex OS (git/local safe, robust)"
---

# bootstrap-agents (robust, feature-first)

You are bootstrapping this repository so Codex can operate end-to-end with a feature-first workflow. This must be safe and idempotent.

## Goals
- Create/update `AGENTS.md` and `.dev-docs` scaffolding.
- Establish the feature registry and current feature pointer.
- Seed templates for feature specs, status, tasks, PRs, and durable history/ledger.
- Keep content ASCII and safe for OneDrive.

## Non-negotiables
- Do not modify application code in this skill.
- Do not run git operations beyond preflight.
- Re-running must not duplicate content.
- Never delete files.

## Workflow
1) Universal preflight (record only):
   - `git --version` (best-effort)
   - `git rev-parse --is-inside-work-tree` (best-effort)
   - Determine `EFFECTIVE_MODE` = git if both succeed, else local.

2) Ensure directories exist:
   - `.dev-docs/`
   - `.dev-docs/design/`
   - `.dev-docs/features/_template/`
   - `.dev-docs/context/agents/default/`
   - `.github/`

3) Write or update the docs listed below using deterministic templates.
   - If a file exists, update only if content differs.
   - Keep templates stable.

4) Robust bootstrap additions:
   - Ensure `.gitignore` includes (append if missing; do not remove existing lines):
     - `.worktrees/`
     - `.dev-docs/.tmp/`
   - Ensure durable feature ledger index exists:
     - `.dev-docs/features/FEATURE_LOG.md`
       - If missing, create with a short header and a table placeholder.
   - Ensure retention / durability guidance exists:
     - `.dev-docs/design/HISTORY_AND_RETENTION.md`
       - Must explain:
         - Agent context under `.dev-docs/context/agents/<agent_id>/` is ephemeral in git worktrees.
         - Durable completion records belong under `.dev-docs/features/<feature_id>/`.
         - Completion record convention: update `STATUS.json`, and optionally copy final handoff snapshot to `features/<id>/history/`.

## Files to create/update (repo-relative)
- `AGENTS.md`
- `.dev-docs/design/FEATURE_SYSTEM.md`
- `.dev-docs/design/HISTORY_AND_RETENTION.md`
- `.dev-docs/features/REGISTRY.json`
- `.dev-docs/features/CURRENT_FEATURE.txt`
- `.dev-docs/features/FEATURE_LOG.md`
- `.dev-docs/features/_template/SPEC.md`
- `.dev-docs/features/_template/STATUS.json`
- `.dev-docs/commands.md`
- `.dev-docs/review-checklist.md`
- `.dev-docs/context/agents/default/TASKS.md`
- `.dev-docs/context/agents/default/WORKING.md`
- `.dev-docs/context/agents/default/PR_DRAFT.md`
- `.github/pull_request_template.md`
- `.gitignore` (idempotent append-only)

## Template requirements

### STATUS.json template (must include these optional completion fields)
Include (as keys, may be null/empty in template):
- `state`: one of `planned|in_progress|completed|abandoned`
- `created_at`, `updated_at`
- `completed_at`
- `pr_url`
- `merge_commit`
- `merge_method`
- `agent_id`

### FEATURE_LOG.md template
- Header: `# Feature Log`
- Notes: One line explaining it’s a durable ledger on main.
- A markdown table skeleton with columns: Date, Feature, PR, Commit, Notes.

### HISTORY_AND_RETENTION.md template
Must contain:
- “Ephemeral vs durable” explanation
- Recommended convention for recording completion
- Recommended location for archived handoff snapshots:
  `.dev-docs/features/<feature_id>/history/`

## Output
After writing files, print EXACTLY these 5 lines and nothing else:
✅ AGENTS.md created/updated
📁 .dev-docs baseline created/updated
🧾 Feature ledger created/updated
🧷 Retention guidance created/updated
🔜 Next: run `$feature-create` then `$agent-init` on your target feature
