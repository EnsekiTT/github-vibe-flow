---
name: vf-execute
description: "Use to execute GitHub Issues with tmux + git worktree. Parses execution notation [#1, (#2, #3), #4] and manages parallel/serial agent dispatch."
---

# Vibe Flow Execute

## 概要

tmuxペインとgit worktreeを使ってGitHub Issueを実行する。実行記法を解析してシリアル/パラレルの実行順序を制御する。

**開始時の宣言:** 「vf-execute スキルを使って、Issueの実行を開始します。」

## プロセス

### 1. 入力の解析

引数として実行記法を受け取るか、設計ドキュメントから抽出する:
- 引数: `[#12, (#13, #14), #15]`
- 設計ドキュメントパス: ドキュメントから記法を抽出する

**解析ルール:**
- `[...]` = シリアルグループ（左から右へ順次実行、各完了を待つ）
- `(...)` = パラレルグループ（全項目を同時実行、全完了を待つ）
- `#N` = GitHub Issue番号
- ネストに対応: `[#1, (#2, [#3, #4]), #5]`

### 2. tmux環境の確認

```bash
# tmux内にいるか確認
if [ -n "$TMUX" ]; then
    echo "tmuxセッション内です"
else
    echo "tmux外です — 新しいセッションを作成します"
    tmux new-session -d -s vibe-flow
    # 注意: ユーザーにアタッチを案内する: tmux attach -t vibe-flow
fi
```

### 3. Issue詳細の取得

実行記法内の各Issue番号について、GitHub MCP経由で詳細を取得する:
- タイトル
- 本文（説明 + 受入基準）

### 4. DAGに従って実行

実行記法をDAGとして処理する:

**シリアルステップごとに:**
1. 単一Issueの場合: 実行する（下記「Agentの起動」参照）
2. パラレルグループの場合: 全Issueを同時に起動する

**パラレルグループごとに:**
1. 全Issueを同時に起動する
2. 全完了を待ってから次のステップに進む

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

# 新しいtmuxウィンドウでClaude Codeを起動（ペインではなくウィンドウを使い、各Agentの進捗を個別に確認可能にする）
tmux new-window -n "vf-${ISSUE_NUM}" "cd ${REPO_ROOT}/.worktrees/${BRANCH_NAME} && claude --dangerously-skip-permissions -p 'AGENT_PROMPT_HERE'"
```

**注意:** `--print` は使用しない。`--print` は全出力をバッファして完了時に一括表示するため、実行中の進捗が見えなくなる。`--print` なしであればTUIがリアルタイムで表示され、ユーザーが `tmux select-window` で各Agentの進捗を確認できる。

### 6. 進捗のモニタリング

各Agentのウィンドウが残っているかで完了を検知する:

```bash
# 各Agentウィンドウの生存確認（ウィンドウが閉じれば完了）
tmux list-windows -F '#{window_name} #{window_active}'
```

シリアルステップの完了（またはパラレルグループの全項目完了）を検知したら、次のステップに進む。

### 7. 完了

全Issueの処理が完了したら:
- 作成されたPRのサマリを報告する
- `/vf-monitor` の開始を提案し、レビューコメントの監視を促す

## エラーハンドリング

- worktree作成に失敗した場合（ブランチが既存）: 警告し、スキップするか確認する
- tmuxペイン作成に失敗した場合: エラーを報告する
- Agentがエラーで終了した場合: 報告し、リトライかスキップかを確認する

## 連携

**呼び出され方:**
- vf-flow — E2Eフローの第3ステップとして（Readツールで読み込まれる）
- vf-plan — 短縮フローからの実行開始時

**利用するもの:**
- GitHub MCP Server — Issue詳細の取得
- tmux — ペイン管理
- git worktree — 作業領域の隔離
