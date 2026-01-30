---
name: handoff
description: "Save durable session handoff for a feature to WORKING.md plus history snapshot. Feature-first, safe."
metadata:
  short-description: "Save session context (feature-first)"
---
# handoff

Prepare a context handoff for the current feature.

## Goals
- Produce an accurate, high-signal handoff.
- Keep a stable per-agent latest pointer and immutable history.

## Paths
- `.dev-docs/context/agents/<agent_id>/WORKING.md`
- `.dev-docs/context/agents/<agent_id>/history/<TS_FILE>--<TAG_SLUG>.md`

## Workflow
1) Universal preflight:
   - `git --version` and `git rev-parse --is-inside-work-tree` (best-effort).
   - Determine `EFFECTIVE_MODE` = git if both succeed, else local.

2) Resolve feature and mapping.
   - Ensure TASKS exists.

3) Compute timestamps (UTC):
   - `TS_ISO`: `YYYY-MM-DDTHH:MM:SSZ`
   - `TS_FILE`: `YYYY-MM-DDTHH-MM-SSZ`

4) Choose TAG:
   - Use user-provided tag if present.
   - Else derive from TASKS Current Goal.

5) Ensure directories exist:
   - `.dev-docs/context/agents/<agent_id>/history/`

6) Capture repo state (best-effort; use `Unknown` if not git):
   - Branch, commit, status, diff stat.

7) Write snapshot and latest pointer using the template.

8) Optionally update history index.

## Output
After writing the files, print EXACTLY these 3 lines and NOTHING ELSE:
? Context saved to `.dev-docs/context/agents/<agent_id>/WORKING.md`
?? Snapshot saved to `.dev-docs/context/agents/<agent_id>/history/<TS_FILE>--<TAG_SLUG>.md`
?? To resume: `/mention .dev-docs/context/agents/<agent_id>/WORKING.md`

## Handoff file template
# Session Context - <TS_ISO>

## Current Session Overview
- **Feature ID**: <feature_id>
- **Feature Name**: <feature name>
- **Agent ID**: <agent_id>
- **Main Task/Feature**: <from TASKS>
- **Current Status**: <where we are>
- **Last Known Good State**: <what was working>

## Repo Snapshot
- **Branch**: <branch or Unknown>
- **Commit**: <sha or Unknown>
- **Dirty Working Tree**: <Yes/No/Unknown>
- **git status --porcelain**:
  - <paste or summarize>
- **git diff --stat**:
  - <paste or summarize>

## Progress vs Tasks
- **Completed**:
  - <bullets>
- **In Progress**:
  - <bullets>
- **Next Steps (priority order)**:
  1. <next action>
  2. <next action>
  3. <next action>

## Running / Testing
- **Commands run**:
  - <commands + results>
- **Known Issues / Gotchas**:
  - <list>

## Resume Checklist
- [ ] `/mention .dev-docs/context/agents/<agent_id>/WORKING.md`
- [ ] Review `git status` and `git diff`
- [ ] Run checks from `.dev-docs/commands.md`
- [ ] Start with "Next Steps #1"
