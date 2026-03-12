---
name: gsd:new-project
description: 深いコンテキスト収集とPROJECT.mdによる新規プロジェクトの初期化
argument-hint: "[--auto]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---
<context>
**フラグ:**
- `--auto` — 自動モード。設定の質問後、追加のやり取りなしでリサーチ → 要件定義 → ロードマップを実行します。@参照でアイデアドキュメントを指定してください。
</context>

<objective>
統合フローによる新規プロジェクトの初期化: 質問 → リサーチ（任意） → 要件定義 → ロードマップ。

**作成されるファイル:**
- `.planning/PROJECT.md` — プロジェクトコンテキスト
- `.planning/config.json` — ワークフロー設定
- `.planning/research/` — ドメインリサーチ（任意）
- `.planning/REQUIREMENTS.md` — スコープされた要件
- `.planning/ROADMAP.md` — フェーズ構成
- `.planning/STATE.md` — プロジェクトメモリ

**このコマンドの後:** `/gsd:plan-phase 1` を実行して実行を開始してください。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/new-project.md
@~/.claude/get-shit-done/references/questioning.md
@~/.claude/get-shit-done/references/ui-brand.md
@~/.claude/get-shit-done/templates/project.md
@~/.claude/get-shit-done/templates/requirements.md
</execution_context>

<process>
@~/.claude/get-shit-done/workflows/new-project.md のnew-projectワークフローをエンドツーエンドで実行します。
すべてのワークフローゲート（バリデーション、承認、commit、ルーティング）を維持してください。
</process>
</output>
