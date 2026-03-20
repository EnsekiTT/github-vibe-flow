---
name: vf-execute
description: "Use to execute GitHub Issues with tmux + git worktree. Parses execution notation [#1, (#2, #3), #4] and manages parallel/serial agent dispatch."
---

# Vibe Flow Execute

## 概要

tmuxペインとgit worktreeを使ってGitHub Issueを実行する。実行記法を解析し、シリアル/パラレルの実行順序を決定する。

**開始時の宣言:** 「vf-execute スキルを使って、Issueを実行します。」

## プロセス

### 1. 入力の解析

引数として実行記法を受け取るか、設計ドキュメントから読み取る:
- 引数: `[#12, (#13, #14), #15]`
- 設計ドキュメントパス: ドキュメントから記法を抽出

**解析ルール:**
- `[...]` = シリアルグループ（左から右へ順次実行、各完了を待つ）
- `(...)` = パラレルグループ（全アイテムを同時実行、全完了を待つ）
- `#N` = GitHub Issue番号
- ネストに対応: `[#1, (#2, [#3, #4]), #5]`

### 2. tmux環境の確認

```bash
# tmux内かどうかを確認
if [ -n "$TMUX" ]; then
    echo "tmuxセッション内です"
else
    echo "tmux外です — 新しいセッションを作成します"
    tmux new-session -d -s vibe-flow
    # 注意: ユーザーにアタッチを案内する: tmux attach -t vibe-flow
fi
```

### 3. Issue詳細の取得

記法内の各Issue番号について、GitHub MCP経由で詳細を取得する:
- タイトル
- 本文（説明 + 受入基準）

### 4. DAGに従って実行

実行記法をDAGとして処理する:

**シリアルステップごとに:**
1. 単一Issueの場合: そのIssueを実行（下記「Agentの起動」参照）
2. パラレルグループの場合: 全Issueを同時に起動

**パラレルグループごとに:**
1. 全Issueを同時に起動
2. 全完了を待ってから次に進む

### 5. Agentの起動（Issue単位）

各Issueについて、`./agent-prompt.md` テンプレートにIssue詳細を埋め込んでAgentプロンプトを構築する:

```bash
# リポジトリルートとIssueスラグを取得
REPO_ROOT=$(git rev-parse --show-toplevel)
ISSUE_NUM=12
ISSUE_TITLE_SLUG="add-auth-endpoint"  # タイトルをスラグ化
BRANCH_NAME="vf/${ISSUE_NUM}-${ISSUE_TITLE_SLUG}"

# worktreeを作成
git worktree add "${REPO_ROOT}/.worktrees/${BRANCH_NAME}" -b "${BRANCH_NAME}"

# 新しいtmuxペインでClaude Codeを起動
tmux split-window -t vibe-flow -h
tmux send-keys -t vibe-flow "cd ${REPO_ROOT}/.worktrees/${BRANCH_NAME} && claude --print --dangerously-skip-permissions -p 'AGENT_PROMPT_HERE'" Enter

# 識別用にペインの名前を設定
tmux select-pane -T "vf-${ISSUE_NUM}"
```

### 6. 進捗の監視

tmuxペインのアクティビティを確認して各Agentの状態を追跡する:

```bash
# ペインのプロセスが実行中かどうかを確認
tmux list-panes -t vibe-flow -F '#{pane_title} #{pane_pid} #{pane_current_command}'
```

シリアルステップの完了（またはパラレルグループの全アイテム完了）後、次のステップに進む。

### 7. 完了

全Issueの処理が終わったら:
- 作成されたPRのサマリを報告
- `/vf-monitor` の開始を提案し、人間のレビューコメントを監視する

## エラーハンドリング

- worktree作成の失敗（ブランチが既に存在）: 警告を表示し、スキップするか確認する
- tmuxペイン作成の失敗: エラーを報告する
- Agentがエラーで終了: 報告し、リトライするかスキップするか確認する

## 連携

**呼び出し元:**
- vf-flow — 3番目のステップとして

**呼び出し先:**
- GitHub MCP Server — Issue詳細の取得
- tmux — ペイン管理
- git worktree — 作業領域の隔離
