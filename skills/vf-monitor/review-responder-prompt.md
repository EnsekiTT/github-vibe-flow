# Review Responder Agent Prompt Template

Use this template when launching a Claude Code instance to respond to human review comments on a PR.

Substitute placeholders (PR_NUMBER, BRANCH_NAME, REVIEW_COMMENTS, WORKTREE_PATH) with actual values.

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
6. Self-review your changes (same loop as initial implementation):
   - Review diff, identify improvements, implement them, comment
7. Judge if the PR is ready for human review again
   - If NOT ready: continue improving
   - If ready: add a final comment stating the PR is ready for re-review
8. Exit when done

## Important

- Address every review comment
- Do not ignore or skip any feedback
- If a comment is unclear, document your interpretation in the PR comment
- Stay focused on the review feedback — do not make unrelated changes
"
```
