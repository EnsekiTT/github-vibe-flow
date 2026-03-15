# Issue Agent Prompt Template

Use this template when launching a Claude Code instance in a tmux pane to work on an issue.

Substitute placeholders (ISSUE_NUMBER, ISSUE_TITLE, ISSUE_BODY, ACCEPTANCE_CRITERIA, WORKTREE_PATH, BRANCH_NAME) with actual values.

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
