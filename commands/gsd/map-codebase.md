---
name: gsd:map-codebase
description: 並列マッパーエージェントでコードベースを分析し、.planning/codebase/ ドキュメントを生成する
argument-hint: "[オプション: マッピングする特定の領域、例: 'api' や 'auth']"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Write
  - Task
---

<objective>
並列の gsd-codebase-mapper エージェントを使用して既存のコードベースを分析し、構造化されたコードベースドキュメントを生成します。

各マッパーエージェントは担当領域を調査し、`.planning/codebase/` に**ドキュメントを直接書き込みます**。オーケストレーターは確認のみを受け取り、コンテキスト使用量を最小限に抑えます。

出力: .planning/codebase/ フォルダーにコードベースの状態に関する7つの構造化ドキュメント。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/map-codebase.md
</execution_context>

<context>
対象領域: $ARGUMENTS（オプション — 指定された場合、エージェントに特定のサブシステムに集中するよう指示）

**既存のプロジェクト状態を読み込む:**
.planning/STATE.md の存在を確認 — プロジェクトが既に初期化されている場合はコンテキストを読み込む

**このコマンドは以下のタイミングで実行可能:**
- /gsd:new-project の前（ブラウンフィールドコードベース） — 先にコードベースマップを作成
- /gsd:new-project の後（グリーンフィールドコードベース） — コード進化に合わせてコードベースマップを更新
- いつでもコードベースの理解を更新
</context>

<when_to_use>
**map-codebase を使用する場合:**
- 初期化前のブラウンフィールドプロジェクト（既存コードを先に理解する）
- 大きな変更後にコードベースマップを更新する
- 不慣れなコードベースへのオンボーディング
- 大規模リファクタリング前（現在の状態を理解する）
- STATE.md が古いコードベース情報を参照している場合

**map-codebase をスキップする場合:**
- コードがまだないグリーンフィールドプロジェクト（マッピング対象がない）
- 些細なコードベース（5ファイル未満）
</when_to_use>

<process>
1. .planning/codebase/ が既に存在するか確認（更新またはスキップを提案）
2. .planning/codebase/ ディレクトリ構造を作成
3. 4つの並列 gsd-codebase-mapper エージェントを起動:
   - エージェント1: tech フォーカス → STACK.md、INTEGRATIONS.md を作成
   - エージェント2: arch フォーカス → ARCHITECTURE.md、STRUCTURE.md を作成
   - エージェント3: quality フォーカス → CONVENTIONS.md、TESTING.md を作成
   - エージェント4: concerns フォーカス → CONCERNS.md を作成
4. エージェントの完了を待ち、確認を収集（ドキュメント内容ではない）
5. 7つのドキュメントすべてが行数とともに存在することを検証
6. コードベースマップをコミット
7. 次のステップを提案（通常: /gsd:new-project または /gsd:plan-phase）
</process>

<success_criteria>
- [ ] .planning/codebase/ ディレクトリが作成されている
- [ ] マッパーエージェントにより7つのコードベースドキュメントすべてが書き込まれている
- [ ] ドキュメントがテンプレート構造に従っている
- [ ] 並列エージェントがエラーなく完了している
- [ ] ユーザーが次のステップを把握している
</success_criteria>
