---
name: vf-execute
description: "Use to execute GitHub Issues with tmux + git worktree. Parses execution notation [#1, (#2, #3), #4] and manages parallel/serial agent dispatch."
---

# Vibe Flow Execute

## Overview

Execute GitHub Issues using tmux panes and git worktrees. Parse execution notation to determine serial/parallel execution order.

**Announce at start:** "I'm using the vf-execute skill to execute issues."

## The Process

### 1. Parse Input

Accept execution notation as argument or read from design document:
- Argument: `[#12, (#13, #14), #15]`
- Design doc path: extract notation from the document

**Parsing rules:**
- `[...]` = serial group (execute items left to right, wait for each)
- `(...)` = parallel group (execute all items concurrently, wait for all)
- `#N` = GitHub Issue number
- Nesting is supported: `[#1, (#2, [#3, #4]), #5]`

### 2. Check tmux Environment

```bash
# Check if inside tmux
if [ -n "$TMUX" ]; then
    echo "Inside tmux session"
else
    echo "Not in tmux - creating new session"
    tmux new-session -d -s vibe-flow
    # NOTE: Inform user they need to attach: tmux attach -t vibe-flow
fi
```

### 3. Fetch Issue Details

For each issue number in the notation, fetch details via GitHub MCP:
- Title
- Body (description + acceptance criteria)

### 4. Execute According to DAG

Process the execution notation as a DAG:

**For each serial step:**
1. If single issue: execute it (see "Launch Agent" below)
2. If parallel group: launch all issues concurrently

**For each parallel group:**
1. Launch all issues simultaneously
2. Wait for all to complete before proceeding

### 5. Launch Agent (per issue)

For each issue, build the agent prompt from `./agent-prompt.md` template with issue details substituted:

```bash
# Get repo root and issue slug
REPO_ROOT=$(git rev-parse --show-toplevel)
ISSUE_NUM=12
ISSUE_TITLE_SLUG="add-auth-endpoint"  # slugified issue title
BRANCH_NAME="vf/${ISSUE_NUM}-${ISSUE_TITLE_SLUG}"

# Create worktree
git worktree add "${REPO_ROOT}/.worktrees/${BRANCH_NAME}" -b "${BRANCH_NAME}"

# Launch Claude Code in a new tmux pane
tmux split-window -t vibe-flow -h
tmux send-keys -t vibe-flow "cd ${REPO_ROOT}/.worktrees/${BRANCH_NAME} && claude --print --dangerously-skip-permissions -p 'AGENT_PROMPT_HERE'" Enter

# Rename pane for identification
tmux select-pane -T "vf-${ISSUE_NUM}"
```

### 6. Monitor Progress

Track each agent's status by checking tmux pane activity:

```bash
# Check if a pane's process is still running
tmux list-panes -t vibe-flow -F '#{pane_title} #{pane_pid} #{pane_current_command}'
```

When a serial step completes (or all items in a parallel group complete), proceed to the next step.

### 7. Completion

When all issues are processed:
- Report summary of created PRs
- Suggest starting `/vf-monitor` to watch for human review comments

## Error Handling

- If worktree creation fails (branch exists): warn and skip or ask
- If tmux pane creation fails: report error
- If an agent exits with errors: report and ask whether to retry or skip

## Integration

**Called by:**
- vf-flow — as the third step

**Calls:**
- GitHub MCP Server — fetch issue details
- tmux — pane management
- git worktree — workspace isolation
