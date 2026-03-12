---
name: gsd:add-tests
description: UAT基準と実装に基づいて完了したフェーズのテストを生成する
argument-hint: "<phase> [additional instructions]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
argument-instructions: |
  引数をフェーズ番号（整数、小数、または文字サフィックス）とオプションのフリーテキスト指示として解析する。
  例: /gsd:add-tests 12
  例: /gsd:add-tests 12 focus on edge cases in the pricing module
---
<objective>
完了したフェーズのユニットテストとE2Eテストを、SUMMARY.md、CONTEXT.md、VERIFICATION.mdを仕様として使用して生成する。

実装ファイルを分析し、TDD（ユニット）、E2E（ブラウザ）、またはスキップカテゴリに分類し、ユーザー承認のためにテスト計画を提示し、RED-GREEN規約に従ってテストを生成する。

出力: `test(phase-{N}): add unit and E2E tests from add-tests command` というメッセージでコミットされたテストファイル
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/add-tests.md
</execution_context>

<context>
フェーズ: $ARGUMENTS

@.planning/STATE.md
@.planning/ROADMAP.md
</context>

<process>
@~/.claude/get-shit-done/workflows/add-tests.md のadd-testsワークフローをエンドツーエンドで実行する。
すべてのワークフローゲートを保持する（分類承認、テスト計画承認、RED-GREEN検証、ギャップ報告）。
</process>

