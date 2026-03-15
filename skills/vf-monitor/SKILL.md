---
name: vf-monitor
description: "Use to monitor PRs for human review comments and automatically dispatch agents to respond. Runs as a persistent polling loop in a tmux pane."
---

# Vibe Flow Monitor

## Overview

Run a persistent polling loop that watches for human review comments on vibe-flow PRs and automatically dispatches agents to respond.

**Announce at start:** "I'm using the vf-monitor skill to start monitoring PRs for review comments."

## The Process

### 1. Setup

```bash
# Ensure we're in tmux
if [ -z "$TMUX" ]; then
    echo "ERROR: vf-monitor must run inside tmux"
    exit 1
fi
```

Identify monitoring targets:
- Find all open PRs with the `vibe-flow` label via GitHub MCP
- Record the latest comment/review timestamp for each PR (to detect new ones)

### 2. Polling Loop

Run a loop with configurable interval (default: 30 seconds, override via argument):

```bash
INTERVAL=${1:-30}  # seconds
```

Each iteration:

1. **Fetch PR updates** via GitHub MCP for each monitored PR:
   - New reviews (especially `changes_requested`)
   - New review comments
   - New issue comments

2. **Filter out agent comments:**
   - Ignore comments from the bot/agent user
   - Only process comments from human users

3. **On new human comment detected:**
   - Identify the PR and branch
   - Check if worktree exists for the branch, create if needed:
     ```bash
     REPO_ROOT=$(git rev-parse --show-toplevel)
     WORKTREE_PATH="${REPO_ROOT}/.worktrees/${BRANCH_NAME}"
     if [ ! -d "$WORKTREE_PATH" ]; then
         git worktree add "$WORKTREE_PATH" "${BRANCH_NAME}"
     fi
     ```
   - Build prompt from `./review-responder-prompt.md` template
   - Launch Claude Code in a new tmux pane:
     ```bash
     tmux split-window -t vibe-flow -h
     tmux send-keys -t vibe-flow "cd ${WORKTREE_PATH} && claude --print --dangerously-skip-permissions -p 'RESPONDER_PROMPT'" Enter
     tmux select-pane -T "vf-review-${PR_NUMBER}"
     ```
   - Update the last-seen timestamp for this PR

4. **Check termination conditions:**
   - If all monitored PRs are merged or closed: exit the loop
   - Log: "All monitored PRs resolved. Stopping monitor."

### 3. Manual Control

The monitor can be stopped by:
- Killing the tmux pane
- All PRs being resolved (auto-stop)

## Configuration

| Parameter | Default | Description |
|---|---|---|
| interval | 30 | Polling interval in seconds |

## Integration

**Called by:**
- vf-execute — suggested after all issues complete
- vf-flow — as the fifth step

**Calls:**
- GitHub MCP Server — PR comment polling
- tmux — pane management for responder agents
- git worktree — workspace for responder agents
