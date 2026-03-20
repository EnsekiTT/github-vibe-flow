---
name: vf-monitor
description: "Use to monitor PRs for human review comments and automatically dispatch agents to respond. Runs as a persistent polling loop in a tmux pane."
---

# Vibe Flow Monitor

## 概要

vibe-flow PRに対する人間のレビューコメントを監視し、自動的にAgentをディスパッチして対応する永続的なポーリングループを実行する。

**開始時の宣言:** 「vf-monitor スキルを使って、PRのレビューコメント監視を開始します。」

## プロセス

### 1. セットアップ

```bash
# tmux内にいることを確認
if [ -z "$TMUX" ]; then
    echo "エラー: vf-monitor はtmux内で実行する必要があります"
    exit 1
fi
```

監視対象の特定:
- GitHub MCP経由で `vibe-flow` ラベルが付いたオープンPRを全て取得する
- 各PRの最新コメント/レビューのタイムスタンプを記録する（新規検出用）

### 2. ポーリングループ

設定可能な間隔でループを実行する（デフォルト: 30秒、引数で上書き可能）:

```bash
INTERVAL=${1:-30}  # 秒
```

各イテレーションで:

1. **PR更新の取得** — 監視対象の各PRについてGitHub MCP経由で取得:
   - 新しいレビュー（特に `changes_requested`）
   - 新しいレビューコメント
   - 新しいIssueコメント

2. **Agentコメントの除外:**
   - Bot/Agentユーザーからのコメントは無視する
   - 人間のユーザーからのコメントのみ処理する

3. **新しい人間のコメントを検出した場合:**
   - PRとブランチを特定する
   - ブランチのworktreeが存在するか確認し、なければ作成する:
     ```bash
     REPO_ROOT=$(git rev-parse --show-toplevel)
     WORKTREE_PATH="${REPO_ROOT}/.worktrees/${BRANCH_NAME}"
     if [ ! -d "$WORKTREE_PATH" ]; then
         git worktree add "$WORKTREE_PATH" "${BRANCH_NAME}"
     fi
     ```
   - `./review-responder-prompt.md` テンプレートからプロンプトを構築する
   - 新しいtmuxウィンドウでClaude Codeを起動する:
     ```bash
     tmux new-window -n "vf-review-${PR_NUMBER}" "cd ${WORKTREE_PATH} && claude --dangerously-skip-permissions -p 'RESPONDER_PROMPT'"
     ```
     **注意:** `--print` は使用しない。TUIのリアルタイム表示でユーザーが進捗を確認できるようにする。
   - このPRの最終確認タイムスタンプを更新する

4. **終了条件の確認:**
   - 監視対象の全PRがマージまたはクローズされた場合: ループを終了する
   - ログ出力: 「監視対象のPRがすべて解決されました。モニターを停止します。」

### 3. 手動制御

モニターの停止方法:
- tmuxペインを終了する
- 全PRが解決された場合は自動停止する

## 設定

| パラメータ | デフォルト | 説明 |
|---|---|---|
| interval | 30 | ポーリング間隔（秒） |

## 連携

**呼び出され方:**
- vf-execute — 全Issue完了後に提案される
- vf-flow — E2Eフローの第4ステップとして（Readツールで読み込まれる）

**利用するもの:**
- GitHub MCP Server — PRコメントのポーリング
- tmux — レスポンダーAgentのペイン管理
- git worktree — レスポンダーAgentの作業領域
