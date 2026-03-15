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
