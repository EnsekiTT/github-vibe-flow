# github-vibe-flow Plugin Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Implement a Claude Code Plugin that enables GitHub Issue-driven collaboration between Coding Agents and humans using tmux + git worktree.

**Architecture:** Claude Code Plugin with 6 skills (`/vf-*`), 6 commands (thin wrappers), prompt templates for agent dispatch, and a session-start hook. Skills are markdown files with YAML frontmatter. GitHub operations use MCP. Multi-agent execution uses tmux panes with separate Claude Code instances in git worktrees.

**Tech Stack:** Claude Code Plugin (SKILL.md + commands), Bash (tmux/git/gh CLI), GitHub MCP Server

---

### Task 1: Plugin Scaffold

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `README.md` (update existing)

**Step 1: Create plugin.json**

```json
{
  "name": "github-vibe-flow",
  "description": "GitHub Issue-driven collaboration between Coding Agents and humans: design, issue creation, parallel execution, self-review, monitoring, and cleanup",
  "version": "0.1.0",
  "author": {
    "name": "ensekitt"
  },
  "repository": "https://github.com/ensekitt/github-vibe-flow",
  "license": "MIT",
  "keywords": ["github", "collaboration", "agent", "tmux", "worktree", "issue", "pr"]
}
```

**Step 2: Update README.md**

```markdown
# github-vibe-flow

GitHub Issue-driven collaboration between Coding Agents and humans.

## Skills

| Skill | Command | Description |
|---|---|---|
| `vf-design` | `/vf-design` | Design documents via brainstorming |
| `vf-issue-create` | `/vf-issue-create` | Generate GitHub Issues from design |
| `vf-execute` | `/vf-execute` | Execute issues with tmux + worktree |
| `vf-monitor` | `/vf-monitor` | Monitor PRs and auto-respond to reviews |
| `vf-merge` | `/vf-merge` | Cleanup after merge |
| `vf-flow` | `/vf-flow` | End-to-end orchestration |

## Dependencies

- [Claude Code](https://claude.com/claude-code) with plugin support
- GitHub MCP Server configured
- tmux
- git

## Installation

```bash
claude plugin add ensekitt/github-vibe-flow
```

## Execution Notation

Define issue execution order with lists `[]` (serial) and sets `()` (parallel):

- `[#1, #2, #3]` — execute sequentially
- `[#1, (#2, #3), #4]` — #1 first, then #2 and #3 in parallel, then #4
```

**Step 3: Commit**

```bash
git add .claude-plugin/plugin.json README.md
git commit -m "feat: initialize plugin scaffold with plugin.json and README"
```

---

### Task 2: `/vf-design` Skill + Command

**Files:**
- Create: `skills/vf-design/SKILL.md`
- Create: `commands/vf-design.md`

**Step 1: Create the skill**

Create `skills/vf-design/SKILL.md`:

```markdown
---
name: vf-design
description: "Use to start a new feature design. Wraps superpowers:brainstorming with vibe-flow extensions (issue breakdown, execution notation, acceptance criteria)."
---

# Vibe Flow Design

## Overview

Start a collaborative design session to turn an idea into an actionable plan with GitHub Issues.

Wraps superpowers:brainstorming and extends the output with issue breakdown, execution notation, and acceptance criteria.

**Announce at start:** "I'm using the vf-design skill to start a collaborative design session."

## The Process

### 1. Invoke Brainstorming

Use superpowers:brainstorming to explore the idea and create a design document.

Follow all brainstorming guidelines: one question at a time, multiple choice preferred, incremental validation.

### 2. Extend the Design

After the design is validated, add these sections to the design document:

**Issue Breakdown:**

Break the design into implementable units. Each issue should be:
- A single, well-scoped unit of work
- Independently testable
- Clear about what "done" looks like

Format:

```
## Issue Breakdown

### Issue 1: [Title]
**Description:** [What to implement]
**Acceptance Criteria:**
- [ ] [Criterion 1]
- [ ] [Criterion 2]

### Issue 2: [Title]
...
```

**Execution Notation:**

Define the execution order using list/set notation:
- `[]` = serial (sequential execution)
- `()` = parallel (concurrent execution)
- Numbers are temporary IDs (replaced with GitHub Issue numbers by `/vf-issue-create`)

```
## Execution Order

`[1, (2, 3), 4]`
```

### 3. Save the Design

Save to `docs/plans/YYYY-MM-DD-<topic>-design.md` and commit.

### 4. Handoff

- If invoked from `/vf-flow`: automatically proceed to `/vf-issue-create` with the design document path
- If invoked standalone: report the saved path and suggest running `/vf-issue-create <path>`

## Integration

**Calls:**
- superpowers:brainstorming — core design process

**Called by:**
- vf-flow — as the first step in the E2E flow
```

**Step 2: Create the command**

Create `commands/vf-design.md`:

```markdown
---
description: "Start a collaborative design session for a new feature. Creates design documents with issue breakdown and execution notation."
disable-model-invocation: true
---

Invoke the github-vibe-flow:vf-design skill and follow it exactly as presented to you
```

**Step 3: Commit**

```bash
git add skills/vf-design/SKILL.md commands/vf-design.md
git commit -m "feat: add vf-design skill and command"
```

---

### Task 3: `/vf-issue-create` Skill + Command

**Files:**
- Create: `skills/vf-issue-create/SKILL.md`
- Create: `commands/vf-issue-create.md`

**Step 1: Create the skill**

Create `skills/vf-issue-create/SKILL.md`:

```markdown
---
name: vf-issue-create
description: "Use to generate GitHub Issues from a design document. Parses issue breakdown, creates issues via MCP, and updates execution notation with real issue numbers."
---

# Vibe Flow Issue Create

## Overview

Generate GitHub Issues from a design document created by `/vf-design`.

**Announce at start:** "I'm using the vf-issue-create skill to generate GitHub Issues from the design document."

## The Process

### 1. Read the Design Document

Accept the design document path as argument or ask for it interactively.

```bash
# Verify the file exists
ls <design-doc-path>
```

Read the document and extract:
- Issue breakdown (titles, descriptions, acceptance criteria)
- Execution notation (temporary IDs)

### 2. Create GitHub Issues

For each issue in the breakdown, create a GitHub Issue via MCP:

**Title:** From the issue breakdown title

**Body format:**
```markdown
## Description

[Description from design document]

## Acceptance Criteria

- [ ] [Criterion 1]
- [ ] [Criterion 2]

## Design Document

[Link or path to the design document]

---
*Generated by github-vibe-flow*
```

**Labels:** Add `vibe-flow` label (create if it doesn't exist)

### 3. Update Execution Notation

Map temporary IDs to real GitHub Issue numbers and update the design document:

- Before: `[1, (2, 3), 4]`
- After: `[#12, (#13, #14), #15]`

Write the updated notation back to the design document and commit.

### 4. Confirm with Human

Display the created issues and updated execution notation:

```
Created 4 GitHub Issues:
  #12: [Title 1]
  #13: [Title 2]
  #14: [Title 3]
  #15: [Title 4]

Execution order: [#12, (#13, #14), #15]

Does this look correct?
```

### 5. Handoff

- If invoked from `/vf-flow`: after human confirmation, proceed to `/vf-execute` with the execution notation
- If invoked standalone: done

## Error Handling

- If MCP is not configured, inform the user and suggest configuring GitHub MCP Server
- If label creation fails, continue without labels and warn
- If any issue creation fails, report which ones succeeded and which failed

## Integration

**Called by:**
- vf-design (handoff) — when invoked from `/vf-flow`
- vf-flow — as the second step

**Calls:**
- GitHub MCP Server — issue creation, label management
```

**Step 2: Create the command**

Create `commands/vf-issue-create.md`:

```markdown
---
description: "Generate GitHub Issues from a design document. Creates issues via MCP and updates execution notation with real issue numbers."
disable-model-invocation: true
---

Invoke the github-vibe-flow:vf-issue-create skill and follow it exactly as presented to you
```

**Step 3: Commit**

```bash
git add skills/vf-issue-create/SKILL.md commands/vf-issue-create.md
git commit -m "feat: add vf-issue-create skill and command"
```

---

### Task 4: `/vf-execute` Skill + Command + Agent Prompt

**Files:**
- Create: `skills/vf-execute/SKILL.md`
- Create: `skills/vf-execute/agent-prompt.md`
- Create: `commands/vf-execute.md`

**Step 1: Create the agent prompt template**

Create `skills/vf-execute/agent-prompt.md`:

```markdown
# Issue Agent Prompt Template

Use this template when launching a Claude Code instance in a tmux pane to work on an issue.

```
claude --print --dangerously-skip-permissions -p "
You are working on GitHub Issue ISSUE_NUMBER: ISSUE_TITLE

## Issue Description

ISSUE_BODY

## Acceptance Criteria

ACCEPTANCE_CRITERIA

## Your Working Directory

WORKTREE_PATH

## Your Job

1. Implement exactly what the issue specifies
2. Write tests
3. Verify all tests pass
4. Commit your work with descriptive messages referencing the issue (e.g., 'feat: ... closes #N')
5. Create a Pull Request via gh CLI
6. Self-review and improve (see below)
7. When done, exit

## After Creating the PR

### Self-Review & Improvement Loop

1. Review your own PR diff:
   - Run: gh pr diff BRANCH_NAME
   - Evaluate: code quality, test coverage, edge cases, acceptance criteria

2. Write your review findings as a PR comment:
   - Run: gh pr comment BRANCH_NAME --body 'YOUR_REVIEW'
   - Be specific about what could be improved

3. Implement improvements based on your review:
   - Make changes, commit, push

4. Comment on what you changed:
   - Run: gh pr comment BRANCH_NAME --body 'YOUR_CHANGES'

5. Judge if the PR is ready for human review:
   - Are all acceptance criteria met?
   - Is code quality high?
   - Are tests comprehensive?
   - If NOT ready: go back to step 1
   - If ready: add a final comment stating the PR is ready for review

## Labels

Add the 'vibe-flow' label to the PR:
- Run: gh pr edit BRANCH_NAME --add-label vibe-flow

## Important

- Stay focused on this single issue
- Do not modify files unrelated to this issue
- If you encounter blockers, document them in a PR comment and exit
"
```
```

**Step 2: Create the skill**

Create `skills/vf-execute/SKILL.md`:

```markdown
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

For each issue:

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

The agent prompt is built from `./agent-prompt.md` template with issue details substituted.

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
```

**Step 3: Create the command**

Create `commands/vf-execute.md`:

```markdown
---
description: "Execute GitHub Issues using tmux + git worktree. Supports serial/parallel execution with notation like [#1, (#2, #3), #4]."
disable-model-invocation: true
---

Invoke the github-vibe-flow:vf-execute skill and follow it exactly as presented to you
```

**Step 4: Commit**

```bash
git add skills/vf-execute/SKILL.md skills/vf-execute/agent-prompt.md commands/vf-execute.md
git commit -m "feat: add vf-execute skill with agent prompt template and command"
```

---

### Task 5: `/vf-monitor` Skill + Command

**Files:**
- Create: `skills/vf-monitor/SKILL.md`
- Create: `skills/vf-monitor/review-responder-prompt.md`
- Create: `commands/vf-monitor.md`

**Step 1: Create the review responder prompt template**

Create `skills/vf-monitor/review-responder-prompt.md`:

```markdown
# Review Responder Agent Prompt Template

Use this template when launching a Claude Code instance to respond to human review comments on a PR.

```
claude --print --dangerously-skip-permissions -p "
You are responding to human review comments on PR #PR_NUMBER (branch: BRANCH_NAME).

## Review Comments to Address

REVIEW_COMMENTS

## Your Working Directory

WORKTREE_PATH

## Your Job

1. Read and understand each review comment
2. Implement the requested changes
3. Commit with descriptive messages
4. Push your changes
5. Reply to each review comment explaining what you changed:
   - Run: gh pr comment PR_NUMBER --body 'YOUR_RESPONSE'
6. Self-review your changes (same loop as initial implementation)
7. Judge if the PR is ready for human review again
   - If NOT ready: continue improving
   - If ready: add a final comment stating the PR is ready for re-review
8. Exit when done
"
```
```

**Step 2: Create the skill**

Create `skills/vf-monitor/SKILL.md`:

```markdown
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
```

**Step 3: Create the command**

Create `commands/vf-monitor.md`:

```markdown
---
description: "Monitor vibe-flow PRs for human review comments and automatically dispatch agents to respond. Runs as a polling loop."
disable-model-invocation: true
---

Invoke the github-vibe-flow:vf-monitor skill and follow it exactly as presented to you
```

**Step 4: Commit**

```bash
git add skills/vf-monitor/SKILL.md skills/vf-monitor/review-responder-prompt.md commands/vf-monitor.md
git commit -m "feat: add vf-monitor skill with review responder prompt and command"
```

---

### Task 6: `/vf-merge` Skill + Command

**Files:**
- Create: `skills/vf-merge/SKILL.md`
- Create: `commands/vf-merge.md`

**Step 1: Create the skill**

Create `skills/vf-merge/SKILL.md`:

```markdown
---
name: vf-merge
description: "Use after PRs are merged to clean up worktrees, branches, design documents, and sync local environment to the latest default branch."
---

# Vibe Flow Merge (Cleanup)

## Overview

Clean up all resources created during a vibe-flow cycle and sync the local environment to the latest default branch state.

**Announce at start:** "I'm using the vf-merge skill to clean up after the completed flow."

## The Process

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

# Delete remote branches
git push origin --delete vf/12-add-auth
git push origin --delete vf/13-add-api

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
```

**Step 2: Create the command**

Create `commands/vf-merge.md`:

```markdown
---
description: "Clean up after merged PRs: remove worktrees, delete branches, handle design documents, and sync local environment to latest default branch."
disable-model-invocation: true
---

Invoke the github-vibe-flow:vf-merge skill and follow it exactly as presented to you
```

**Step 3: Commit**

```bash
git add skills/vf-merge/SKILL.md commands/vf-merge.md
git commit -m "feat: add vf-merge skill and command"
```

---

### Task 7: `/vf-flow` Skill + Command

**Files:**
- Create: `skills/vf-flow/SKILL.md`
- Create: `commands/vf-flow.md`

**Step 1: Create the skill**

Create `skills/vf-flow/SKILL.md`:

```markdown
---
name: vf-flow
description: "Use to run the full vibe-flow cycle end-to-end: design → issue creation → execution → monitoring → cleanup."
---

# Vibe Flow (End-to-End)

## Overview

Orchestrate the complete vibe-flow cycle from design through cleanup. This is a convenience skill that chains all other vf-* skills in sequence.

**Announce at start:** "I'm using the vf-flow skill to run the full vibe-flow cycle."

## The Process

### Step 1: Design

Invoke github-vibe-flow:vf-design with the user's input.

Wait for design completion. The design document path is passed to the next step.

### Step 2: Issue Creation

Invoke github-vibe-flow:vf-issue-create with the design document path.

Wait for human confirmation of the execution notation.

### Step 3: Execution

Invoke github-vibe-flow:vf-execute with the confirmed execution notation.

Wait for all agents to complete their work (PRs created, self-reviewed, marked ready).

### Step 4: Monitoring

Invoke github-vibe-flow:vf-monitor to watch for human review comments.

The monitor runs until all PRs are merged or closed.

### Step 5: Cleanup

When all PRs are merged, invoke github-vibe-flow:vf-merge.

Wait for human confirmation before executing cleanup.

## Human Checkpoints

| After Step | Human Action |
|---|---|
| 1. Design | Validated during brainstorming |
| 2. Issue Creation | Confirm execution notation |
| 3. Execution | Review PRs when marked ready |
| 4. Monitoring | Approve PRs |
| 5. Cleanup | Confirm cleanup targets |

## Resumability

If the flow is interrupted at any point, the user can:
- Resume from the current step using the individual skill
- Re-run `/vf-flow` (will need to re-specify the starting point)

Each skill is independent and can be invoked standalone.

## Integration

**Calls (in order):**
1. github-vibe-flow:vf-design
2. github-vibe-flow:vf-issue-create
3. github-vibe-flow:vf-execute
4. github-vibe-flow:vf-monitor
5. github-vibe-flow:vf-merge
```

**Step 2: Create the command**

Create `commands/vf-flow.md`:

```markdown
---
description: "Run the full vibe-flow cycle: design → issue creation → execution → monitoring → cleanup."
disable-model-invocation: true
---

Invoke the github-vibe-flow:vf-flow skill and follow it exactly as presented to you
```

**Step 3: Commit**

```bash
git add skills/vf-flow/SKILL.md commands/vf-flow.md
git commit -m "feat: add vf-flow orchestration skill and command"
```

---

### Task 8: Verify Plugin Structure

**Step 1: Verify directory structure**

```bash
find . -not -path './.git/*' -not -path './.git' | sort
```

Expected output:
```
.
./.claude-plugin
./.claude-plugin/plugin.json
./README.md
./commands
./commands/vf-design.md
./commands/vf-execute.md
./commands/vf-flow.md
./commands/vf-issue-create.md
./commands/vf-merge.md
./commands/vf-monitor.md
./docs
./docs/plans
./docs/plans/2026-03-15-vibe-flow-implementation.md
./docs/plans/2026-03-15-vibe-flow-plugin-design.md
./skills
./skills/vf-design
./skills/vf-design/SKILL.md
./skills/vf-execute
./skills/vf-execute/SKILL.md
./skills/vf-execute/agent-prompt.md
./skills/vf-flow
./skills/vf-flow/SKILL.md
./skills/vf-issue-create
./skills/vf-issue-create/SKILL.md
./skills/vf-merge
./skills/vf-merge/SKILL.md
./skills/vf-monitor
./skills/vf-monitor/SKILL.md
./skills/vf-monitor/review-responder-prompt.md
```

**Step 2: Verify all SKILL.md files have valid frontmatter**

```bash
for f in skills/*/SKILL.md; do
    echo "=== $f ==="
    head -5 "$f"
    echo
done
```

Expected: Each file starts with `---`, has `name:` and `description:` fields, ends with `---`.

**Step 3: Verify all commands reference correct skills**

```bash
for f in commands/*.md; do
    echo "=== $f ==="
    cat "$f"
    echo
done
```

Expected: Each command has frontmatter with `description` and `disable-model-invocation: true`, and body invoking the corresponding skill.

**Step 4: Commit verification (no changes expected)**

```bash
git status
```

Expected: clean working tree.
