---
name: gsd:check-todos
description: 保留中のTodoを一覧表示し、作業するものを選択する
argument-hint: [area filter]
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---

<objective>
すべての保留中のTodoを一覧表示し、選択を許可し、選択されたTodoの完全なコンテキストを読み込み、適切なアクションにルーティングする。

check-todosワークフローにルーティングし、以下を処理する:
- エリアフィルタリング付きのTodoカウントとリスト表示
- 完全なコンテキスト読み込み付きのインタラクティブ選択
- ロードマップ相関チェック
- アクションルーティング（今すぐ作業、フェーズに追加、ブレインストーム、フェーズ作成）
- STATE.mdの更新とGitコミット
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/check-todos.md
</execution_context>

<context>
引数: $ARGUMENTS (省略可能なエリアフィルタ)

Todoの状態とロードマップの相関は、`init todos` と対象を絞った読み取りを使用してワークフロー内で読み込まれる。
</context>

<process>
`@~/.claude/get-shit-done/workflows/check-todos.md` の **check-todosワークフローに従う**。

ワークフローは以下を含むすべてのロジックを処理する:
1. Todoの存在確認
2. エリアフィルタリング
3. インタラクティブなリスト表示と選択
4. ファイルサマリー付きの完全なコンテキスト読み込み
5. ロードマップ相関チェック
6. アクションの提示と実行
7. STATE.mdの更新
8. Gitコミット
</process>

