# リポジトリ基本整理 設計ドキュメント

## 概要

作りたてのgithub-vibe-flowプラグインリポジトリを整理する。
`.gitignore`の追加、初期開発時の設計ドキュメント削除、空ディレクトリの維持を行う。

## 方針

- 最小限の変更でリポジトリをクリーンにする
- 今後のvf-flowサイクルで使う`docs/plans/`ディレクトリは維持する
- git履歴に設計書は残るため、ファイル削除で問題なし

## Issue Breakdown

### Issue 1: .gitignoreの追加とdocs/plans整理
**Description:** `.gitignore`ファイルを新規作成し、`.worktrees/`、`.DS_Store`等を除外対象に設定。初期設計ドキュメント2ファイルを削除し、`docs/plans/.gitkeep`で空ディレクトリを維持する。
**Acceptance Criteria:**
- [ ] `.gitignore`が作成され、`.worktrees/`、`.DS_Store`が除外されている
- [ ] `docs/plans/2026-03-15-vibe-flow-plugin-design.md`が削除されている
- [ ] `docs/plans/2026-03-15-vibe-flow-implementation.md`が削除されている
- [ ] `docs/plans/.gitkeep`が存在し、ディレクトリが維持されている

## Execution Order

`[#1]`
