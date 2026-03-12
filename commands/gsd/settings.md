---
name: gsd:settings
description: GSD ワークフローのトグルとモデルプロファイルを設定する
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---

<objective>
複数の質問プロンプトによる GSD ワークフローエージェントとモデルプロファイルのインタラクティブな設定。

settings ワークフローにルーティングし、以下を処理する：
- 設定ファイルの存在確認
- 現在の設定の読み取りと解析
- インタラクティブな5問のプロンプト（model、research、plan_check、verifier、branching）
- 設定のマージと書き込み
- クイックコマンドリファレンス付きの確認表示
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/settings.md
</execution_context>

<process>
`@~/.claude/get-shit-done/workflows/settings.md` の **settings ワークフローに従うこと**。

ワークフローは以下のすべてのロジックを処理する：
1. デフォルト値での設定ファイル作成（存在しない場合）
2. 現在の設定の読み取り
3. 事前選択付きのインタラクティブな設定表示
4. 回答の解析と設定のマージ
5. ファイルの書き込み
6. 確認表示
</process>
