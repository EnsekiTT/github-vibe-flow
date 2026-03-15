---
name: vf-merge
description: "Use after PRs are merged to clean up worktrees, branches, design documents, and sync local environment to the latest default branch."
---

# Vibe Flow Merge (Cleanup)

## Overview

Clean up all resources created during a vibe-flow cycle and sync the local environment to the latest default branch state.

**Announce at start:** "I'm using the vf-merge skill to clean up after the completed flow."

## The Process

### 0. Merge Request

Before cleanup, check for open vibe-flow PRs that need merging.

```bash
# List open PRs with vibe-flow label
gh pr list --label vibe-flow --state open --json number,title,headRefName
```

**If no open PRs:** Skip to Step 1.

**If open PRs exist:**

For each open PR, ask the human for approval individually:

```
Open vibe-flow PRs found:

PR #15: Add auth endpoint (branch: vf/12-add-auth)
  → Merge this PR? [Yes / No / Skip]
```

For each approved PR, merge via gh CLI:

```bash
gh pr merge <number> --merge --delete-branch
```

- `--merge`: merge commit strategy
- `--delete-branch`: automatically delete the remote branch after merge

After all approved PRs are merged (or skipped), proceed to Step 1.

**Error handling:**
- If merge fails due to conflicts, report the conflict and ask the human to resolve manually
- If merge fails due to required checks, report the status and suggest waiting

### 1. Identify Cleanup Targets

Find all merged vibe-flow PRs and their associated resources:

```bash
# List merged PRs with vibe-flow label
gh pr list --label vibe-flow --state merged --json number,title,headRefName
```

For each merged PR, identify:
- Worktree path: `.worktrees/vf/<issue>-<slug>`
- Local branch: `vf/<issue>-<slug>`
- Remote branch: `origin/vf/<issue>-<slug>`

Also find:
- Design documents in `docs/plans/` referenced by the PRs
- Any remaining tmux panes named `vf-*`

### 2. Present Cleanup Plan

Display the full list of resources to be cleaned up:

```
Cleanup targets:

Worktrees:
  - .worktrees/vf/12-add-auth (merged PR #15)
  - .worktrees/vf/13-add-api (merged PR #16)

Branches (local + remote):
  - vf/12-add-auth
  - vf/13-add-api

Design documents:
  - docs/plans/2026-03-15-auth-design.md

tmux panes:
  - vf-12, vf-13, vf-monitor

Options for design documents:
  1. Delete
  2. Move to docs/plans/archive/

Proceed with cleanup?
```

Wait for human confirmation.

### 3. Execute Cleanup

After confirmation:

```bash
REPO_ROOT=$(git rev-parse --show-toplevel)

# Remove worktrees
git worktree remove "${REPO_ROOT}/.worktrees/vf/12-add-auth" --force
git worktree remove "${REPO_ROOT}/.worktrees/vf/13-add-api" --force

# Delete local branches
git branch -d vf/12-add-auth
git branch -d vf/13-add-api

# Delete remote branches (skip if already deleted by --delete-branch in Step 0)
git push origin --delete vf/12-add-auth 2>/dev/null || true
git push origin --delete vf/13-add-api 2>/dev/null || true

# Handle design documents (based on user choice)
# Option 1: Delete
rm docs/plans/2026-03-15-auth-design.md

# Option 2: Archive
mkdir -p docs/plans/archive
mv docs/plans/2026-03-15-auth-design.md docs/plans/archive/

# Close tmux panes
tmux kill-pane -t vf-12 2>/dev/null
tmux kill-pane -t vf-13 2>/dev/null
tmux kill-pane -t vf-monitor 2>/dev/null
```

### 4. Sync Local Environment

```bash
# Detect default branch
DEFAULT_BRANCH=$(git remote show origin | grep 'HEAD branch' | awk '{print $NF}')

# Switch to default branch and pull latest
git checkout "${DEFAULT_BRANCH}"
git pull origin "${DEFAULT_BRANCH}"
```

### 5. Report Results

```
Cleanup complete:
  - Removed 2 worktrees
  - Deleted 2 local branches
  - Deleted 2 remote branches
  - Archived 1 design document
  - Closed 3 tmux panes
  - Synced to main (latest: abc1234)
```

## Integration

**Called by:**
- vf-flow — as the final step, after all PRs are merged

**Pairs with:**
- vf-execute — cleans up resources created by vf-execute
- vf-monitor — closes monitor pane
