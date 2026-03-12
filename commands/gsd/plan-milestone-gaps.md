---
name: gsd:plan-milestone-gaps
description: マイルストーン監査で特定されたすべてのギャップを埋めるフェーズを作成する
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---
<objective>
`/gsd:audit-milestone` で特定されたギャップを埋めるために必要なすべてのフェーズを作成する。

MILESTONE-AUDIT.md を読み取り、ギャップを論理的なフェーズにグループ化し、ROADMAP.md にフェーズエントリを作成し、各フェーズの計画を提案する。

1つのコマンドで、すべての修正フェーズを作成する — ギャップごとに手動で `/gsd:add-phase` を実行する必要はない。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/plan-milestone-gaps.md
</execution_context>

<context>
**監査結果：**
Glob: .planning/v*-MILESTONE-AUDIT.md（最新のものを使用）

元の意図と現在の計画状態はワークフロー内でオンデマンドで読み込まれる。
</context>

<process>
@~/.claude/get-shit-done/workflows/plan-milestone-gaps.md の plan-milestone-gaps ワークフローをエンドツーエンドで実行すること。
すべてのワークフローゲート（監査の読み込み、優先順位付け、フェーズのグループ化、ユーザー確認、ロードマップ更新）を維持すること。
</process>
