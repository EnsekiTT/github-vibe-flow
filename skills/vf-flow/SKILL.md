---
name: vf-flow
description: "Use to run the full vibe-flow cycle end-to-end: design → issue creation → execution → monitoring → cleanup."
---

# Vibe Flow (End-to-End)

## Overview

vibe-flow サイクル全体を設計からクリーンアップまで統括する。

**開始時の宣言:** 「vf-flow スキルを使って、vibe-flow サイクル全体を実行します。」

## パスの解決方法

本スキル内のパス（例: `skills/vf-design/SKILL.md`）はリポジトリルートからの相対パスで記述している。Read ツールは絶対パスを要求するため、実行時にはリポジトリルートを特定してパスを結合すること。

```bash
git rev-parse --show-toplevel
```

例: リポジトリルートが `/Users/user/dev/github-vibe-flow` の場合、`skills/vf-design/SKILL.md` は `/Users/user/dev/github-vibe-flow/skills/vf-design/SKILL.md` として Read ツールに渡す。

## プロセス

### Step 1: 設計

`skills/vf-design/SKILL.md` を Read ツールで読み込み、その指示に従って設計セッションを実行する。

設計が完了したら、設計ドキュメントのパスを次のステップに引き継ぐ。

### Step 2: Issue 作成

`skills/vf-issue-create/SKILL.md` を Read ツールで読み込み、設計ドキュメントのパスを入力として実行する。

人間による実行記法の確認を待つ。

### Step 3: 実行

`skills/vf-execute/SKILL.md` を Read ツールで読み込み、確認済みの実行記法を入力として実行する。

すべてのAgentが作業を完了するまで待つ（PR作成・セルフレビュー・Ready状態）。

### Step 4: モニタリング

`skills/vf-monitor/SKILL.md` を Read ツールで読み込み、PRの監視を開始する。

すべてのPRがマージまたはクローズされるまで継続する。

### Step 5: クリーンアップ

すべてのPRがマージされたら、`skills/vf-merge/SKILL.md` を Read ツールで読み込み、クリーンアップを実行する。

人間による確認を待ってから実行する。

## ヒューマンチェックポイント

| ステップ後 | 人間のアクション |
|---|---|
| 1. 設計 | 設計セッション中に対話的に承認 |
| 2. Issue 作成 | 実行記法を確認 |
| 3. 実行 | PRがReady状態になったらレビュー |
| 4. モニタリング | PRを承認 |
| 5. クリーンアップ | クリーンアップ対象を確認 |

## 再開方法

フローが中断された場合:
- 個別のスキルを使って現在のステップから再開できる
- `/vf-flow` を再実行する場合は、開始ポイントを再指定する必要がある

各スキルは独立しており、単体で呼び出し可能。

## 短縮フロー（既存Issue活用）

すでにGitHub Issueが作成済みの場合、設計・Issue作成（Step 1-2）をスキップして `/vf-plan` から開始できる。
`vf-plan` は既存Issueから対話的に実行対象を選定し、実行順序を決定して `vf-execute` に繋げる独立したエントリーポイント。

短縮フローの流れ: `vf-plan → vf-execute → vf-monitor → vf-merge`

## Integration

**Reads (in order):**
1. skills/vf-design/SKILL.md
2. skills/vf-issue-create/SKILL.md
3. skills/vf-execute/SKILL.md
4. skills/vf-monitor/SKILL.md
5. skills/vf-merge/SKILL.md
