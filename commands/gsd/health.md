---
name: gsd:health
description: プランニングディレクトリの健全性を診断し、オプションで問題を修復する
argument-hint: [--repair]
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
---
<objective>
`.planning/` ディレクトリの整合性を検証し、対処可能な問題を報告します。ファイルの欠落、無効な設定、状態の不整合、孤立した計画をチェックします。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/health.md
</execution_context>

<process>
@~/.claude/get-shit-done/workflows/health.md のヘルスワークフローをエンドツーエンドで実行します。
引数から --repair フラグを解析し、ワークフローに渡します。
</process>
