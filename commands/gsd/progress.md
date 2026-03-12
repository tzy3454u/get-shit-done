---
name: gsd:progress
description: プロジェクトの進捗確認、コンテキスト表示、次のアクションへのルーティング（実行または計画）
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
  - SlashCommand
---
<objective>
プロジェクトの進捗を確認し、最近の作業とこれからの作業を要約した後、次のアクション（既存の計画の実行または次の計画の作成）にインテリジェントにルーティングします。

作業を継続する前に状況認識を提供します。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/progress.md
</execution_context>

<process>
@~/.claude/get-shit-done/workflows/progress.md のprogressワークフローをエンドツーエンドで実行します。
すべてのルーティングロジック（ルートAからF）とエッジケースの処理を維持してください。
</process>
</output>
