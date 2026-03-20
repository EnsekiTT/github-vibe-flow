---
name: vf-merge
description: "Use after PRs are merged to clean up worktrees, branches, design documents, and sync local environment to the latest default branch."
---

# Vibe Flow Merge（クリーンアップ）

## 概要

vibe-flowサイクルで作成された全リソースをクリーンアップし、ローカル環境をデフォルトブランチの最新状態に同期する。

**開始時の宣言:** 「vf-merge スキルを使って、完了したフローのクリーンアップを実行します。」

## プロセス

### 0. マージ依頼

クリーンアップの前に、マージが必要なオープンPRを確認する。

```bash
# vibe-flowラベル付きのオープンPRを一覧表示
gh pr list --label vibe-flow --state open --json number,title,headRefName
```

**オープンPRなし:** ステップ1に進む。

**オープンPRあり:**

各オープンPRについて、人間に個別に承認を確認する:

```
オープンなvibe-flow PR:

PR #15: Add auth endpoint (ブランチ: vf/12-add-auth)
  → このPRをマージしますか？ [はい / いいえ / スキップ]
```

承認されたPRごとに、gh CLIでマージする:

```bash
gh pr merge <number> --merge --delete-branch
```

- `--merge`: マージコミット戦略
- `--delete-branch`: マージ後にリモートブランチを自動削除

全承認済みPRのマージ（またはスキップ）完了後、ステップ1に進む。

**エラーハンドリング:**
- コンフリクトによるマージ失敗: コンフリクトを報告し、人間に手動解決を依頼
- 必須チェック未通過によるマージ失敗: ステータスを報告し、待機を提案

### 1. クリーンアップ対象の特定

マージ済みのvibe-flow PRと関連リソースを検索する:

```bash
# vibe-flowラベル付きのマージ済みPRを一覧表示
gh pr list --label vibe-flow --state merged --json number,title,headRefName
```

各マージ済みPRについて、以下を特定する:
- worktreeパス: `.worktrees/vf/<issue>-<slug>`
- ローカルブランチ: `vf/<issue>-<slug>`
- リモートブランチ: `origin/vf/<issue>-<slug>`

また、以下も検索する:
- PRに参照されている `docs/plans/` 内の設計ドキュメント
- `vf-*` という名前の残存tmuxペイン

### 2. クリーンアップ計画の提示

クリーンアップ対象リソースの全リストを表示する:

```
クリーンアップ対象:

worktree:
  - .worktrees/vf/12-add-auth（マージ済みPR #15）
  - .worktrees/vf/13-add-api（マージ済みPR #16）

ブランチ（ローカル＋リモート）:
  - vf/12-add-auth
  - vf/13-add-api

設計ドキュメント:
  - docs/plans/2026-03-15-auth-design.md

tmuxペイン:
  - vf-12, vf-13, vf-monitor

設計ドキュメントの処理:
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

# リモートブランチの削除（ステップ0の--delete-branchで削除済みの場合はスキップ）
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
# デフォルトブランチの検出
DEFAULT_BRANCH=$(git remote show origin | grep 'HEAD branch' | awk '{print $NF}')

# デフォルトブランチに切り替えて最新を取得
git checkout "${DEFAULT_BRANCH}"
git pull origin "${DEFAULT_BRANCH}"
```

### 5. 結果の報告

```
クリーンアップ完了:
  - worktree 2件を削除
  - ローカルブランチ 2件を削除
  - リモートブランチ 2件を削除
  - 設計ドキュメント 1件をアーカイブ
  - tmuxペイン 3件を終了
  - mainに同期完了（最新: abc1234）
```

## 連携

**呼び出し元:**
- vf-flow — 全PRマージ後の最終ステップとして

**関連スキル:**
- vf-execute — vf-executeが作成したリソースをクリーンアップ
- vf-monitor — モニターペインを終了
