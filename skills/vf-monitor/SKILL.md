---
name: vf-monitor
description: "Use to monitor PRs for human review comments and automatically dispatch agents to respond. Runs as a persistent polling loop in a tmux pane."
---

# Vibe Flow Monitor

## 概要

vibe-flow PRへの人間のレビューコメントを監視し、自動的にAgentをディスパッチして対応する持続的なポーリングループを実行する。

**開始時の宣言:** 「vf-monitor スキルを使って、PRのレビューコメント監視を開始します。」

## プロセス

### 1. セットアップ

```bash
# tmux内かどうかを確認
if [ -z "$TMUX" ]; then
    echo "エラー: vf-monitorはtmux内で実行する必要があります"
    exit 1
fi
```

監視対象の特定:
- GitHub MCP経由で `vibe-flow` ラベル付きのオープンPRをすべて取得
- 各PRの最新コメント/レビューのタイムスタンプを記録（新着検出用）

### 2. ポーリングループ

設定可能なインターバル（デフォルト: 30秒、引数で上書き可能）でループを実行する:

```bash
INTERVAL=${1:-30}  # 秒
```

各イテレーション:

1. **PR更新の取得** — 監視対象の各PRについてGitHub MCP経由で取得:
   - 新しいレビュー（特に `changes_requested`）
   - 新しいレビューコメント
   - 新しいIssueコメント

2. **Agentコメントのフィルタリング:**
   - bot/Agentユーザーからのコメントは無視
   - 人間ユーザーからのコメントのみ処理

3. **新しい人間のコメントを検出した場合:**
   - PRとブランチを特定
   - worktreeの存在を確認し、必要なら作成:
     ```bash
     REPO_ROOT=$(git rev-parse --show-toplevel)
     WORKTREE_PATH="${REPO_ROOT}/.worktrees/${BRANCH_NAME}"
     if [ ! -d "$WORKTREE_PATH" ]; then
         git worktree add "$WORKTREE_PATH" "${BRANCH_NAME}"
     fi
     ```
   - `./review-responder-prompt.md` テンプレートからプロンプトを構築
   - 新しいtmuxペインでClaude Codeを起動:
     ```bash
     tmux split-window -t vibe-flow -h
     tmux send-keys -t vibe-flow "cd ${WORKTREE_PATH} && claude --print --dangerously-skip-permissions -p 'RESPONDER_PROMPT'" Enter
     tmux select-pane -T "vf-review-${PR_NUMBER}"
     ```
   - このPRの最終確認タイムスタンプを更新

4. **終了条件の確認:**
   - 監視対象の全PRがマージまたはクローズされた場合: ループを終了
   - ログ出力: 「監視対象の全PRが解決されました。モニターを停止します。」

### 3. 手動制御

モニターの停止方法:
- tmuxペインを終了する
- 全PRが解決される（自動停止）

## 設定

| パラメータ | デフォルト | 説明 |
|---|---|---|
| interval | 30 | ポーリング間隔（秒） |

## 連携

**呼び出し元:**
- vf-execute — 全Issue完了後に提案される
- vf-flow — 5番目のステップとして

**呼び出し先:**
- GitHub MCP Server — PRコメントのポーリング
- tmux — レスポンダーAgentのペイン管理
- git worktree — レスポンダーAgentの作業領域
