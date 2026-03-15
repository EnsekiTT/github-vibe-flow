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
