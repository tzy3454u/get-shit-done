---
name: gsd:add-phase
description: 現在のマイルストーンのロードマップ末尾にフェーズを追加する
argument-hint: <description>
allowed-tools:
  - Read
  - Write
  - Bash
---

<objective>
ロードマップの現在のマイルストーン末尾に新しい整数フェーズを追加する。

add-phaseワークフローにルーティングし、以下を処理する:
- フェーズ番号の計算（次の連番整数）
- スラグ生成によるディレクトリ作成
- ロードマップ構造の更新
- STATE.mdのロードマップ進化追跡
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/add-phase.md
</execution_context>

<context>
引数: $ARGUMENTS (フェーズの説明)

ロードマップと状態はワークフロー内で `init phase-op` と対象を絞ったツール呼び出しを通じて解決される。
</context>

<process>
`@~/.claude/get-shit-done/workflows/add-phase.md` の **add-phaseワークフローに従う**。

ワークフローは以下を含むすべてのロジックを処理する:
1. 引数の解析とバリデーション
2. ロードマップの存在確認
3. 現在のマイルストーンの特定
4. 次のフェーズ番号の計算（小数を無視）
5. 説明からのスラグ生成
6. フェーズディレクトリの作成
7. ロードマップエントリの挿入
8. STATE.mdの更新
</process>
</output>
