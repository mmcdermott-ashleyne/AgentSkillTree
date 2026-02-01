---
name: pr-automerge-cleanup
description: "Auto-merge PR via gh and fully clean up feature worktree/branch/metadata. Feature-first, safe, idempotent."
metadata:
  short-description: "Auto-merge PR + full cleanup (feature-first, durable record)"
---

# pr-automerge-cleanup (feature-first)

This skill can:
1) Ensure a PR exists (optional),
2) Auto-merge via `gh`,
3) Record durable completion,
4) Clean up worktree, branch, and metadata safely.

## Inputs
- Feature identity (`FEATURE_ID` / `FEATURE` / prompt `Feature: <name>`)
- Optional PR number: `PR_NUMBER` or `pr=<number>`
- Optional merge method: `MERGE_METHOD` in {merge,squash,rebase} (default squash)
- Optional auto-open behavior: `AUTO_OPEN_PR` in {true,false} (default true)
- Optional base branch override: `BASE_BRANCH` or `base=<name>`
- Optional safety override: `ALLOW_MERGE_WITHOUT_CHECKS` in {true,false} (default false)
- Optional archive control: `ARCHIVE_MODE` in {commit,pr-only,none} (default commit)
- Optional archive snapshot: `ARCHIVE_SNAPSHOT` in {true,false} (default true)

## Controls (from TASKS)
- `Mode: local|git` (auto resolves to git here)
- `Cleanup ResetHard: true|false` (default false)

## Guardrails (non-negotiable)
- Never deletes outside `.worktrees/`.
- Never runs `git clean -fdx`.
- Never runs `git reset --hard` unless explicitly allowed.
- Only deletes remote branches via `gh pr merge --delete-branch`.

## Workflow
1) Preflight:
   - `git --version` and `git rev-parse --is-inside-work-tree` must succeed.
   - `REPO_ROOT = git rev-parse --show-toplevel`.

2) Resolve feature and mapping.
   - Ensure TASKS exists; read `Mode`.
   - If `Mode: local`, stop: `ERROR: Mode=local; refusing to auto-merge/cleanup`.

3) Validate IDs are path-safe.

4) Determine worktree:
   - `AGENT_WT = <REPO_ROOT>/<worktree>` from registry.
   - If missing, stop with ERROR (cleanup requires a known worktree path).

5) Ensure `gh` is available and authenticated.

6) Determine branch + base + PR:
   - Use WORKDIR = `AGENT_WT` if valid; else ERROR.
   - `HEAD_BRANCH = git rev-parse --abbrev-ref HEAD`.
   - `BASE_BRANCH` from input or `gh repo view`, else `main`.
   - Refuse base branch.

7) PR resolution:
   - If `PR_NUMBER` provided, validate head/base.
   - Else if `AUTO_OPEN_PR=true`, locate or create PR (use PR_DRAFT if present).
   - Else error.

8) Ensure branch is pushed (commit if needed, then push).

9) Gate merge on checks unless overridden.

9.5) Archive & record feature completion (durable, BEFORE merge):
   - This phase MUST run before step 10 and before worktree removal.
   - Default `ARCHIVE_MODE=commit`.

   a) Ensure a final handoff exists (best-effort):
      - If `.dev-docs/context/agents/<agent_id>/history/` has no snapshot for this feature,
        run `$handoff` (tag: "final").

   b) Build completion metadata (best-effort):
      - `COMPLETED_AT` = current UTC ISO timestamp
      - `PR_URL` from gh
      - `MERGE_METHOD` resolved
      - `HEAD_SHA` = `git rev-parse HEAD` (worktree)
      - `AGENT_ID` from mapping

   c) Update `.dev-docs/features/<feature_id>/STATUS.json` (repo-owned, durable):
      - Set/overwrite these keys:
        - `state` = `completed`
        - `completed_at` = `COMPLETED_AT`
        - `updated_at` = `COMPLETED_AT`
        - `pr_url` = `PR_URL`
        - `merge_method` = `MERGE_METHOD`
        - `merge_commit` = `HEAD_SHA` (or best-effort; may be updated post-merge if available)
        - `agent_id` = `AGENT_ID`
      - Do not delete any other fields.

   d) Optionally copy latest snapshot into feature history (if `ARCHIVE_SNAPSHOT=true`):
      - Source: newest file in `.dev-docs/context/agents/<agent_id>/history/`
      - Dest: `.dev-docs/features/<feature_id>/history/<TS>--final.md`
      - If dest exists, do nothing.

   e) Optional ledger append (best-effort, idempotent-by-line):
      - If `.dev-docs/features/FEATURE_LOG.md` exists:
        - Append a single new row if an identical PR_URL is not already present:
          `| <YYYY-MM-DD> | <feature_id> | <PR_URL> | <HEAD_SHA> | completed |`

   f) If `ARCHIVE_MODE=commit`:
      - Stage only:
        - `.dev-docs/features/<feature_id>/STATUS.json`
        - `.dev-docs/features/<feature_id>/history/*` (if created)
        - `.dev-docs/features/FEATURE_LOG.md` (if modified)
      - Commit message:
        `chore(feature): record completion for <feature_id>`
      - If nothing changed, do not commit.

   g) If `ARCHIVE_MODE=pr-only`:
      - Do not commit.
      - Ensure PR body contains a brief “Completion Record” section (best-effort) noting:
        - completed_at, feature_id, and a pointer to STATUS.json path.

   h) If `ARCHIVE_MODE=none`:
      - Skip c–g (but still allow merge/cleanup).

10) Merge via `gh pr merge --<method> --delete-branch`.

11) Sync base branch in repo root:
   - `git fetch origin <base> --prune`
   - `git checkout <base>`
   - `git pull --ff-only origin <base>` or `git reset --hard origin/<base>` if allowed.

12) Remove local worktree safely:
   - `git worktree remove --force "<AGENT_WT>"`
   - If locked, delete only `<AGENT_WT>/.dev-docs/.tmp` and retry.

13) Prune worktree metadata:
   - `git worktree prune`

14) Delete local branch:
   - `git branch -d "<HEAD_BRANCH>"` (or `-D` if needed after merge).

## Output
If successful, print EXACTLY these 6 lines and nothing else:
✅ PR merged: <PR_URL>
✅ Base synced: <BASE_BRANCH>
🧹 Worktree removed: <AGENT_WT or "none">
🪓 Local branch deleted: <HEAD_BRANCH or "none">
🌐 Remote branch deleted: yes
🧽 Worktree metadata pruned