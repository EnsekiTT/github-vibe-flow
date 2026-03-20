---
name: vf-plan
description: "Use to select and prioritize existing GitHub Issues for execution. Interactively filters, analyzes dependencies, and builds execution notation through dialogue with the user."
---

# Vibe Flow Plan

## Overview

リポジトリの既存GitHub Issueから対話的に実行対象を選定し、実行順序を決めて vf-execute に繋げる。vf-design を経由しない「既存Issue活用」の独立したエントリーポイント。

**開始時の宣言:** 「vf-plan スキルを使って、既存Issueから実行計画を作成します。」

## プロセス

### 1. フィルタ条件の対話的決定

人間に以下のフィルタ条件を対話的に確認する：

- **ラベル** — 特定のラベルで絞るか？（例: `bug`, `enhancement`, `vibe-flow`）
- **マイルストーン** — 特定のマイルストーンに限定するか？
- **アサイン** — 特定のアサイン先で絞るか？
- **フリーテキスト** — キーワード検索するか？

条件なしの場合はopen Issue全件を取得する。複数条件の組み合わせも可。

### 2. Issue取得・一覧表示

GitHub MCP Serverを使ってIssueを取得し、一覧表示する：

- Issue番号、タイトル、ラベル、作成日
- 件数が多い場合はカテゴリ別（ラベルなど）にグルーピングして見やすく整理する

### 3. 依存関係・概要の整理

各Issueの本文を読み、以下を分析・提示する：

- 各Issueの概要（1行サマリ）
- Issue間の依存関係（本文中の参照や内容から推定）
- 関連するIssue同士のグルーピング

この分析結果を整理して人間に提示する。

### 4. スコープ絞り込み

依存関係の整理を踏まえて、人間と対話的に「今回実行するIssue」を決める：

- Claudeが「この組み合わせが良さそうです」と推奨を提示し、理由を説明する
- 人間が追加・除外を指示
- 最終的な対象Issueリストを人間と確認する

### 5. 実行順序の提案

選定されたIssueに対して、実行記法を提案する：

- 依存関係があるものはシリアル `[]`
- 独立しているものはパラレル `()`
- 提案理由を簡潔に説明する
- 人間が修正可能（例:「#3は#2の後にして」）
- 修正があれば反映して再提示する

### 6. 出力とハンドオフ

確定した実行記法を表示する。

人間に「ドキュメントとして保存しますか？」と確認する：
- 希望する場合: `docs/plans/YYYY-MM-DD-<topic>-plan.md` に保存し、コミットする
- 希望しない場合: 実行記法のみ出力する

最後に `/vf-execute <実行記法>` の実行を提案する。

## エラーハンドリング

- GitHub MCP Serverが未設定の場合: 設定方法を案内する
- Issueが0件の場合: フィルタ条件の変更を提案する
- 依存関係の推定に自信がない場合: 不確実であることを明示し、人間に確認する

## 連携

**独立したエントリーポイントとして機能する。**

短縮フロー: `vf-plan → vf-execute → vf-monitor → vf-merge`

**呼び出し先:**
- GitHub MCP Server — Issue取得・詳細読み込み

**ハンドオフ先:**
- vf-execute — 実行記法を渡して実行を開始
