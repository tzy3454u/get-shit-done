---
name: gsd:list-phase-assumptions
description: 計画開始前にClaudeのフェーズに対する前提を明らかにする
argument-hint: "[phase]"
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
---

<objective>
フェーズを分析し、技術的アプローチ、実装順序、スコープ境界、リスク領域、依存関係についてClaudeの前提を提示する。

目的: 計画開始前にClaudeがどう考えているかをユーザーに見せ、前提が間違っている場合に早期に軌道修正できるようにする。
出力: 会話形式の出力のみ（ファイル作成なし）— 「どう思いますか？」のプロンプトで終了
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/list-phase-assumptions.md
</execution_context>

<context>
フェーズ番号: $ARGUMENTS（必須）

プロジェクトの状態とロードマップはワークフロー内で対象読み込みを使用してロードされる。
</context>

<process>
1. フェーズ番号の引数を検証（欠落または無効な場合はエラー）
2. フェーズがロードマップに存在するか確認
3. list-phase-assumptions.md ワークフローに従う:
   - ロードマップの説明を分析
   - 前提を明らかにする: 技術的アプローチ、実装順序、スコープ、リスク、依存関係
   - 前提を明確に提示
   - 「どう思いますか？」とプロンプトを表示
4. フィードバックを収集し、次のステップを提案
</process>

<success_criteria>

- フェーズがロードマップに対して検証されている
- 5つの領域にわたる前提が明らかにされている
- ユーザーにフィードバックが求められている
- ユーザーが次のステップを把握している（コンテキストの議論、フェーズの計画、または前提の修正）
  </success_criteria>
