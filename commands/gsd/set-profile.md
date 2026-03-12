---
name: gsd:set-profile
description: GSD エージェントのモデルプロファイルを切り替える（quality/balanced/budget）
argument-hint: <profile>
allowed-tools:
  - Read
  - Write
  - Bash
---

<objective>
GSD エージェントが使用するモデルプロファイルを切り替える。各エージェントが使用する Claude モデルを制御し、品質とトークンコストのバランスを調整する。

set-profile ワークフローにルーティングし、以下を処理する：
- 引数のバリデーション（quality/balanced/budget）
- 設定ファイルが存在しない場合の作成
- config.json のプロファイル更新
- モデルテーブル表示による確認
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/set-profile.md
</execution_context>

<process>
`@~/.claude/get-shit-done/workflows/set-profile.md` の **set-profile ワークフローに従うこと**。

ワークフローは以下のすべてのロジックを処理する：
1. プロファイル引数のバリデーション
2. 設定ファイルの確認
3. 設定の読み取りと更新
4. MODEL_PROFILES からのモデルテーブル生成
5. 確認表示
</process>
