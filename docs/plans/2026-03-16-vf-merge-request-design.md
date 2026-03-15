# vf-merge マージリクエスト機能追加 設計ドキュメント

## 概要

vf-mergeスキルに「マージリクエスト」ステップを追加する。
クリーンアップの前に、未マージのvibe-flow PRを検出し、人間の個別承認を経て`gh pr merge`でマージする。

## 変更対象

- `skills/vf-merge/SKILL.md`

## 設計

### 新ステップ: Step 0 — マージリクエスト

既存のStep 1（クリーンアップ対象の特定）の前に挿入する。

**フロー:**
1. `gh pr list --label vibe-flow --state open`でオープン中のvibe-flow PRを取得
2. オープンPRがない場合はスキップして既存フローへ
3. 各PRについて人間に個別確認:「PR #N: [タイトル] をマージしますか？」
4. 承認されたPRを`gh pr merge <number> --merge --delete-branch`でマージ
5. 全マージ完了後、既存のクリーンアップフロー（Step 1〜5）に進む

### 既存フローへの影響

- Step 1〜5は変更なし
- マージ時に`--delete-branch`でリモートブランチが削除されるため、Step 3のリモートブランチ削除は重複チェックが必要

## Issue Breakdown

### Issue 1: vf-merge SKILL.mdにマージリクエストステップを追加
**Description:** vf-mergeスキルのThe Processセクションに「Step 0: マージリクエスト」を追加。オープン中のvibe-flow PRを検出し、個別承認を経てgh pr mergeで自動マージする機能を記述する。
**Acceptance Criteria:**
- [ ] SKILL.mdに「Step 0: マージリクエスト」セクションが追加されている
- [ ] オープンPRの検出手順が記載されている
- [ ] 個別承認フローが記載されている
- [ ] `gh pr merge --merge --delete-branch`でのマージ手順が記載されている
- [ ] オープンPRがない場合のスキップ条件が記載されている

## Execution Order

`[#3]`
