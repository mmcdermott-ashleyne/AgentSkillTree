# AgentSkillTree

Turn LLMs into reliable junior devs by providing a set of structured, repo-local skills that enforce a consistent, safe workflow.

This repo is feature-first. Most skills resolve a feature name to a stable mapping in the registry, then operate with safe git/no-git guardrails.

## Quick start
1) Run `$bootstrap-agents` to create `.dev-docs` scaffolding and templates.
2) Run `$feature-create` with `Feature: <name>` to create a feature entry and spec.
3) If in a git repo, run `$agent-init` to create or reuse a worktree.
4) Run `$plan`, `$build`, `$review`, `$pr-draft`, `$pr-open`.
5) Optionally run `$pr-automerge-cleanup` to merge and clean up.

## Core concepts
### Feature-first identity
Preferred identity is a feature name or ID:
- `Feature: Monday Tools`
- `FEATURE_ID=monday-tools`
- `FEATURE=Monday Tools`

Back-compat overrides:
- `AGENT_ID`
- `AGENT_SLUG`

If both are provided and conflict, skills warn and update the registry.

### Registry and pointers
- Registry: `.dev-docs/features/REGISTRY.json`
- Current pointer: `.dev-docs/features/CURRENT_FEATURE.txt`
- Feature spec: `.dev-docs/features/<feature_id>/SPEC.md`
- Feature status: `.dev-docs/features/<feature_id>/STATUS.json`

### Stable mapping
- Branch: `feat/<agent_id>/<slug>`
- Worktree: `.worktrees/agent-<agent_id>`
- Agent context: `.dev-docs/context/agents/<agent_id>/`

## Modes and git detection
Each skill starts with a preflight:
- `git --version`
- `git rev-parse --is-inside-work-tree`

Mode resolution:
- If both succeed: `EFFECTIVE_MODE=git`
- Else: `EFFECTIVE_MODE=local`
- If TASKS says `Mode: auto`, use `EFFECTIVE_MODE`
- If TASKS says `Mode: git` but git is unavailable, the skill errors

## Safety rules (non-negotiable)
- Never delete outside `.worktrees/`.
- Never run `git clean -fdx`.
- Never `git reset --hard` unless explicitly allowed in TASKS.
- Refuse to operate on base branch in build and PR steps.
- Build does not create worktrees or branches.

## How to use the skills
### Bootstrap
Use once per repo (safe to rerun):
- `$bootstrap-agents`

### Create or select a feature
- `$feature-create` with `Feature: <name>`
- `$feature-select` to set current feature pointer
- `$feature-status` to show resolved mapping and paths

### Initialize a worktree (git repos only)
- `$agent-init`

### Plan and build
- `$plan` writes `.dev-docs/context/agents/<agent_id>/TASKS.md`
- `$build` implements the plan; refuses base branch

### Review and PR
- `$review` runs checks and updates TASKS
- `$pr-draft` generates `.dev-docs/context/agents/<agent_id>/PR_DRAFT.md`
- `$pr-open` commits, pushes, and opens a PR (git only)

### Merge and cleanup
- `$pr-automerge-cleanup` merges and removes the worktree and branch
- `$ship` runs plan -> build -> review -> pr-draft -> pr-open -> pr-automerge-cleanup (if enabled) -> handoff

## Ship autopmerge
In TASKS Controls, set:
- `AutoMerge: true`
- `Merge Method: merge|squash|rebase` (optional, default squash)

When `AutoMerge: true` and Mode resolves to git, `$ship` will run `$pr-automerge-cleanup`.

## Directory layout
```
.dev-docs/
  commands.md
  review-checklist.md
  design/FEATURE_SYSTEM.md
  features/
    REGISTRY.json
    CURRENT_FEATURE.txt
    _template/
      SPEC.md
      STATUS.json
    <feature_id>/
      SPEC.md
      STATUS.json
  context/
    agents/
      default/
        TASKS.md
        WORKING.md
        PR_DRAFT.md
      <agent_id>/
        TASKS.md
        WORKING.md
        PR_DRAFT.md
        history/
.worktrees/
  agent-<agent_id>/
```

## Troubleshooting
- Worktree missing: run `$agent-init` to recreate it.
- Mode mismatch: check `## Controls` in TASKS.
- Git errors: ensure you are inside a repo and `git` is on PATH.
- PR errors: verify `gh` is installed and authenticated.
