---
name: vf-merge
description: "Use after PRs are merged to clean up worktrees, branches, design documents, and sync local environment to the latest default branch."
---

# Vibe Flow Merge (Cleanup)

## 概要

vibe-flowサイクルで作成されたすべてのリソースをクリーンアップし、ローカル環境をデフォルトブランチの最新状態に同期する。

**開始時の宣言:** 「vf-merge スキルを使って、完了したフローのクリーンアップを行います。」

## プロセス

### 0. マージリクエスト

クリーンアップの前に、マージが必要なオープンのvibe-flow PRを確認する。

```bash
# vibe-flowラベルのオープンPRを一覧表示
gh pr list --label vibe-flow --state open --json number,title,headRefName
```

**オープンPRがない場合:** ステップ1にスキップする。

**オープンPRがある場合:**

各オープンPRについて、個別に人間の承認を求める:

```
オープンのvibe-flow PRが見つかりました:

PR #15: Add auth endpoint (ブランチ: vf/12-add-auth)
  → このPRをマージしますか？ [はい / いいえ / スキップ]
```

承認されたPRごとに、gh CLI経由でマージする:

```bash
gh pr merge <number> --merge --delete-branch
```

- `--merge`: マージコミット戦略
- `--delete-branch`: マージ後にリモートブランチを自動削除

すべての承認済みPRがマージ（またはスキップ）されたら、ステップ1に進む。

**エラーハンドリング:**
- コンフリクトでマージに失敗した場合: コンフリクトを報告し、手動での解決を提案する
- 必須チェックが未完了でマージに失敗した場合: ステータスを報告し、待機を提案する

### 1. クリーンアップ対象の特定

マージ済みのvibe-flow PRと関連リソースを特定する:

```bash
# vibe-flowラベルのマージ済みPRを一覧表示
gh pr list --label vibe-flow --state merged --json number,title,headRefName
```

各マージ済みPRについて以下を特定する:
- worktreeパス: `.worktrees/vf/<issue>-<slug>`
- ローカルブランチ: `vf/<issue>-<slug>`
- リモートブランチ: `origin/vf/<issue>-<slug>`

また以下も検出する:
- PRが参照する `docs/plans/` 内の設計ドキュメント
- `vf-*` という名前の残存tmuxペイン

### 2. クリーンアップ計画の提示

クリーンアップ対象の全リストを表示する:

```
クリーンアップ対象:

worktree:
  - .worktrees/vf/12-add-auth (マージ済みPR #15)
  - .worktrees/vf/13-add-api (マージ済みPR #16)

ブランチ（ローカル + リモート）:
  - vf/12-add-auth
  - vf/13-add-api

設計ドキュメント:
  - docs/plans/2026-03-15-auth-design.md

tmuxペイン:
  - vf-12, vf-13, vf-monitor

設計ドキュメントの処理方法:
  1. 削除
  2. docs/plans/archive/ に移動

クリーンアップを実行しますか？
```

人間の確認を待つ。

### 3. クリーンアップの実行

確認後:

```bash
REPO_ROOT=$(git rev-parse --show-toplevel)

# worktreeの削除
git worktree remove "${REPO_ROOT}/.worktrees/vf/12-add-auth" --force
git worktree remove "${REPO_ROOT}/.worktrees/vf/13-add-api" --force

# ローカルブランチの削除
git branch -d vf/12-add-auth
git branch -d vf/13-add-api

# リモートブランチの削除（ステップ0の --delete-branch で削除済みの場合はスキップ）
git push origin --delete vf/12-add-auth 2>/dev/null || true
git push origin --delete vf/13-add-api 2>/dev/null || true

# 設計ドキュメントの処理（ユーザーの選択に基づく）
# 選択肢1: 削除
rm docs/plans/2026-03-15-auth-design.md

# 選択肢2: アーカイブ
mkdir -p docs/plans/archive
mv docs/plans/2026-03-15-auth-design.md docs/plans/archive/

# tmuxペインの終了
tmux kill-pane -t vf-12 2>/dev/null
tmux kill-pane -t vf-13 2>/dev/null
tmux kill-pane -t vf-monitor 2>/dev/null
```

### 4. ローカル環境の同期

```bash
# デフォルトブランチを検出
DEFAULT_BRANCH=$(git remote show origin | grep 'HEAD branch' | awk '{print $NF}')

# デフォルトブランチに切り替えて最新を取得
git checkout "${DEFAULT_BRANCH}"
git pull origin "${DEFAULT_BRANCH}"
```

### 5. 結果の報告

```
クリーンアップ完了:
  - worktreeを2件削除
  - ローカルブランチを2件削除
  - リモートブランチを2件削除
  - 設計ドキュメントを1件アーカイブ
  - tmuxペインを3件終了
  - mainに同期 (最新: abc1234)
```

## 連携

**呼び出され方:**
- vf-flow — E2Eフローの最終ステップとして、全PRマージ後に実行（Readツールで読み込まれる）

**関連スキル:**
- vf-execute — vf-executeが作成したリソースをクリーンアップする
- vf-monitor — モニターペインを終了する
