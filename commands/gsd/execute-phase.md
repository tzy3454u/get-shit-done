---
name: gsd:execute-phase
description: ウェーブベースの並列化によるフェーズ内全計画の実行
argument-hint: "<phase-number> [--gaps-only]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - TodoWrite
  - AskUserQuestion
---
<objective>
ウェーブベースの並列実行を使用して、フェーズ内のすべての計画を実行する。

オーケストレーターはスリムに保つ: 計画の検出、依存関係の分析、ウェーブへのグループ化、サブエージェントの起動、結果の収集。各サブエージェントは完全なexecute-planコンテキストをロードし、自身の計画を処理する。

コンテキスト予算: オーケストレーター約15%、サブエージェントごとに100%フレッシュ。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/execute-phase.md
@~/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
フェーズ: $ARGUMENTS

**フラグ:**
- `--gaps-only` — ギャップ解消計画のみを実行（frontmatterに`gap_closure: true`がある計画）。verify-workが修正計画を作成した後に使用する。

コンテキストファイルはワークフロー内で`gsd-tools init execute-phase`およびサブエージェントごとの`<files_to_read>`ブロックを通じて解決される。
</context>

<process>
@~/.claude/get-shit-done/workflows/execute-phase.md のexecute-phaseワークフローをエンドツーエンドで実行する。
すべてのワークフローゲート（ウェーブ実行、チェックポイント処理、検証、状態更新、ルーティング）を維持すること。
</process>

