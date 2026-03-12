---
name: gsd:new-milestone
description: 新しいマイルストーンサイクルを開始する — PROJECT.md を更新し要件定義に進む
argument-hint: "[マイルストーン名、例: 'v1.1 Notifications']"
allowed-tools:
  - Read
  - Write
  - Bash
  - Task
  - AskUserQuestion
---
<objective>
新しいマイルストーンを開始: 質問 → 調査（オプション） → 要件 → ロードマップ。

new-project のブラウンフィールド版。プロジェクトは存在し、PROJECT.md に履歴がある状態。「次に何をするか」を収集し、PROJECT.md を更新してから、要件 → ロードマップサイクルを実行する。

**作成/更新するもの:**
- `.planning/PROJECT.md` — 新しいマイルストーン目標で更新
- `.planning/research/` — ドメインリサーチ（オプション、新機能の場合のみ）
- `.planning/REQUIREMENTS.md` — このマイルストーンのスコープ付き要件
- `.planning/ROADMAP.md` — フェーズ構造（番号を継続）
- `.planning/STATE.md` — 新しいマイルストーン用にリセット

**完了後:** `/gsd:plan-phase [N]` で実行を開始。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/new-milestone.md
@~/.claude/get-shit-done/references/questioning.md
@~/.claude/get-shit-done/references/ui-brand.md
@~/.claude/get-shit-done/templates/project.md
@~/.claude/get-shit-done/templates/requirements.md
</execution_context>

<context>
マイルストーン名: $ARGUMENTS（オプション — 未指定の場合はプロンプトで確認）

プロジェクトとマイルストーンのコンテキストファイルはワークフロー内（`init new-milestone`）で解決され、サブエージェントが使用される場合は `<files_to_read>` ブロックを通じて委任される。
</context>

<process>
@~/.claude/get-shit-done/workflows/new-milestone.md の new-milestone ワークフローをエンドツーエンドで実行する。
すべてのワークフローゲート（バリデーション、質問、調査、要件、ロードマップ承認、コミット）を維持する。
</process>
