# Issue Agent プロンプトテンプレート

このテンプレートは、tmuxペインでIssueに取り組むClaude Codeインスタンスを起動する際に使用する。

プレースホルダー（ISSUE_NUMBER, ISSUE_TITLE, ISSUE_BODY, ACCEPTANCE_CRITERIA, WORKTREE_PATH, BRANCH_NAME）を実際の値に置換すること。

```
claude --print --dangerously-skip-permissions -p "
You are working on GitHub Issue ISSUE_NUMBER: ISSUE_TITLE

## Issue の説明

ISSUE_BODY

## 受入基準

ACCEPTANCE_CRITERIA

## 作業ディレクトリ

WORKTREE_PATH

## タスク

1. Implement exactly what the issue specifies
2. Write tests
3. Verify all tests pass
4. Commit your work with descriptive messages referencing the issue (e.g., 'feat: ... closes #N')
5. Create a Pull Request via gh CLI
6. Self-review and improve (see below)
7. When done, exit

## PR作成後

### セルフレビュー・改善ループ

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

## ラベル

Add the 'vibe-flow' label to the PR:
- Run: gh pr edit BRANCH_NAME --add-label vibe-flow

## 注意事項

- Stay focused on this single issue
- Do not modify files unrelated to this issue
- If you encounter blockers, document them in a PR comment and exit
- コミュニケーション・コミットメッセージ・コード内コメントは日本語で記述すること
"
```
