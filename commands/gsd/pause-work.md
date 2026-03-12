---
name: gsd:pause-work
description: フェーズ作業中断時のコンテキスト引き継ぎファイルを作成する
allowed-tools:
  - Read
  - Write
  - Bash
---

<objective>
セッション間で完全な作業状態を保持するための `.continue-here.md` 引き継ぎファイルを作成します。

pause-work ワークフローにルーティングし、以下を処理します：
- 最近のファイルから現在のフェーズを検出
- 完全な状態の収集（位置、完了済み作業、残作業、決定事項、ブロッカー）
- すべてのコンテキストセクションを含む引き継ぎファイルの作成
- WIP として Git コミット
- 再開手順
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/pause-work.md
</execution_context>

<context>
状態とフェーズの進捗はワークフロー内で対象ファイルの読み取りにより収集されます。
</context>

<process>
`@~/.claude/get-shit-done/workflows/pause-work.md` の **pause-work ワークフローに従ってください**。

ワークフローは以下のすべてのロジックを処理します：
1. フェーズディレクトリの検出
2. ユーザーへの確認を含む状態収集
3. タイムスタンプ付き引き継ぎファイルの書き込み
4. Git コミット
5. 再開手順の確認表示
</process>
