---
name: gsd:verify-work
description: 対話型UATを通じてビルドされた機能を検証する
argument-hint: "[phase number, e.g., '4']"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Edit
  - Write
  - Task
---
<objective>
永続的な状態を持つ対話型テストを通じて、ビルドされた機能を検証する。

目的: Claudeがビルドしたものがユーザーの視点から実際に動作することを確認する。一度に1つのテスト、プレーンテキストの応答、尋問なし。問題が見つかった場合、自動的に診断し、修正を計画し、実行の準備をする。

出力: すべてのテスト結果を追跡する{phase_num}-UAT.md。問題が見つかった場合: 診断されたギャップ、/gsd:execute-phase用の検証済み修正計画
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/verify-work.md
@~/.claude/get-shit-done/templates/UAT.md
</execution_context>

<context>
フェーズ: $ARGUMENTS (省略可)
- 指定された場合: 特定のフェーズをテスト (例: "4")
- 指定されない場合: アクティブなセッションを確認するか、フェーズの入力を求める

コンテキストファイルはワークフロー内で解決され (`init verify-work`)、`<files_to_read>` ブロックを通じて委譲される。
</context>

<process>
@~/.claude/get-shit-done/workflows/verify-work.md のverify-workワークフローをエンドツーエンドで実行する。
すべてのワークフローゲートを保持する（セッション管理、テスト提示、診断、修正計画、ルーティング）。
</process>

