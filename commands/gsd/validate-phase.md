---
name: gsd:validate-phase
description: 完了したフェーズのNyquistバリデーションギャップを遡及的に監査・補完する
argument-hint: "[phase number]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---
<objective>
完了したフェーズのNyquistバリデーションカバレッジを監査する。3つの状態:
- (A) VALIDATION.mdが存在する — 監査してギャップを補完
- (B) VALIDATION.mdなし、SUMMARY.mdが存在する — 成果物から再構築
- (C) フェーズ未実行 — ガイダンスを表示して終了

出力: 更新されたVALIDATION.md + 生成されたテストファイル。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/validate-phase.md
</execution_context>

<context>
フェーズ: $ARGUMENTS — 省略可、デフォルトは最後に完了したフェーズ。
</context>

<process>
@~/.claude/get-shit-done/workflows/validate-phase.md を実行する。
すべてのワークフローゲートを保持する。
</process>
</output>
