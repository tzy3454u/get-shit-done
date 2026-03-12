---
name: gsd:plan-phase
description: 検証ループ付きの詳細フェーズ計画（PLAN.md）を作成
argument-hint: "[phase] [--auto] [--research] [--skip-research] [--gaps] [--skip-verify] [--prd <file>]"
agent: gsd-planner
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - WebFetch
  - mcp__context7__*
---
<objective>
ロードマップフェーズに対して、リサーチと検証を統合した実行可能なフェーズプロンプト（PLAN.mdファイル）を作成する。

**デフォルトフロー:** リサーチ（必要な場合） → 計画 → 検証 → 完了

**オーケストレーターの役割:** 引数の解析、フェーズの検証、ドメインリサーチ（スキップされない場合）、gsd-plannerの起動、gsd-plan-checkerによる検証、合格または最大反復回数まで繰り返し、結果の提示。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/plan-phase.md
@~/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
フェーズ番号: $ARGUMENTS（任意 — 省略時は次の未計画フェーズを自動検出）

**フラグ:**
- `--research` — RESEARCH.mdが存在していても強制的にリサーチを再実行
- `--skip-research` — リサーチをスキップし、直接計画に進む
- `--gaps` — ギャップ解消モード（VERIFICATION.mdを読み込み、リサーチをスキップ）
- `--skip-verify` — 検証ループをスキップ
- `--prd <file>` — discuss-phaseの代わりにPRD/受入基準ファイルを使用。要件を自動的にCONTEXT.mdに解析する。discuss-phaseを完全にスキップする。

ステップ2でディレクトリ検索を行う前にフェーズ入力を正規化すること。
</context>

<process>
@~/.claude/get-shit-done/workflows/plan-phase.md のplan-phaseワークフローをエンドツーエンドで実行する。
すべてのワークフローゲート（バリデーション、リサーチ、計画、検証ループ、ルーティング）を維持すること。
</process>

