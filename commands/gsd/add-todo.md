---
name: gsd:add-todo
description: 現在の会話コンテキストからアイデアやタスクをTodoとしてキャプチャする
argument-hint: [optional description]
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---

<objective>
GSDセッション中に浮上したアイデア、タスク、または問題を、後で作業するための構造化されたTodoとしてキャプチャする。

add-todoワークフローにルーティングし、以下を処理する:
- ディレクトリ構造の作成
- 引数または会話からのコンテンツ抽出
- ファイルパスからのエリア推論
- 重複検出と解決
- フロントマター付きTodoファイルの作成
- STATE.mdの更新
- Gitコミット
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/add-todo.md
</execution_context>

<context>
引数: $ARGUMENTS (省略可能なTodoの説明)

状態はワークフロー内で `init todos` と対象を絞った読み取りを通じて解決される。
</context>

<process>
`@~/.claude/get-shit-done/workflows/add-todo.md` の **add-todoワークフローに従う**。

ワークフローは以下を含むすべてのロジックを処理する:
1. ディレクトリの確認
2. 既存エリアの確認
3. コンテンツの抽出（引数または会話から）
4. エリアの推論
5. 重複チェック
6. スラグ生成によるファイル作成
7. STATE.mdの更新
8. Gitコミット
</process>
</output>
