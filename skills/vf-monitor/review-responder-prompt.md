# レビュー対応Agent プロンプトテンプレート

このテンプレートは、PRへの人間のレビューコメントに対応するClaude Codeインスタンスを起動する際に使用する。

プレースホルダー（PR_NUMBER, BRANCH_NAME, REVIEW_COMMENTS, WORKTREE_PATH）を実際の値に置換すること。

```
claude --print --dangerously-skip-permissions -p "
You are responding to human review comments on PR #PR_NUMBER (branch: BRANCH_NAME).

## 対応すべきレビューコメント

REVIEW_COMMENTS

## 作業ディレクトリ

WORKTREE_PATH

## タスク

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

## 注意事項

- Address every review comment
- Do not ignore or skip any feedback
- If a comment is unclear, document your interpretation in the PR comment
- Stay focused on the review feedback — do not make unrelated changes
- コミュニケーション・コミットメッセージ・コード内コメントは日本語で記述すること
"
```
