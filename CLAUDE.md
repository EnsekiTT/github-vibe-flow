# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## コミュニケーション言語

- ユーザーとのコミュニケーションは**日本語**で行うこと
- コード内のコメント、ドキュメント、コミットメッセージも**日本語**で記述すること
- SKILL.mdファイル内の説明文やフロー記述も日本語で書くこと
- ただし、YAML frontmatter の `name` / `description` フィールド、plugin.json、コマンドの技術的識別子は英語のまま維持

## プロジェクト概要

GitHub Issue駆動でCoding Agentと人間が協働するためのClaude Code Plugin。人間は「起点（きっかけ）」と「関門（レビュー承認）」に集中し、実行はAgentが自律的に担う。

## アーキテクチャ

このリポジトリはClaude Code Pluginであり、実行可能なコードではなく**スキル定義（Markdown）とメタデータ**で構成される。

- `.claude-plugin/plugin.json` — プラグインメタデータ
- `skills/<skill-name>/SKILL.md` — スキル定義（YAML frontmatter + Markdown）
- `skills/<skill-name>/*.md` — プロンプトテンプレート（agent-prompt.md, review-responder-prompt.md）
- `commands/<command-name>.md` — slash commandラッパー（`disable-model-invocation: true` で対応するスキルを呼び出すだけ）
- `docs/plans/` — 設計ドキュメントと実装計画

## スキル構成

6つのスキルがE2Eフローを構成する:

1. `vf-design` → 対話的設計セッションで設計ドキュメント作成
2. `vf-issue-create` → 設計からGitHub Issue生成（MCP経由）
3. `vf-execute` → tmux + git worktreeで複数Agent並列実行
4. `vf-monitor` → PRコメントをポーリングし自動レビュー対応
5. `vf-merge` → worktree/ブランチ/設計ドキュメントのクリーンアップ
6. `vf-flow` → 上記1-5を統括するメタフロー

## 実行記法

Issue実行順序を `[]`（シリアル）と `()`（パラレル）で定義:
- `[#1, (#2, #3), #4]` → #1完了後に#2,#3を並列実行、両方完了後に#4

## 依存関係

- GitHub MCP Server（Issue・PR操作）
- tmux（複数Agent実行環境）
- git worktree（Agent作業領域の隔離）

## スキル間連携の規約

- スキルから他のスキルを Skill ツールで呼び出すことは**禁止**（Claude Code の制約）
- 同一プラグイン内のスキルを参照する場合: `skills/<name>/SKILL.md` を Read ツールで読み込み、指示に従う
- 外部プラグインのスキルに依存する場合: 必要なロジックを自スキルにインライン化する

## スキルファイルの規約

- フロントマター: `name`（kebab-case）と `description`（英語、トリガー条件を記述）が必須
- 本文構成: Overview → プロセス → Integration の順
- コマンドファイル: `disable-model-invocation: true` を必ず付与し、対応するスキルを呼び出す1行のみ
