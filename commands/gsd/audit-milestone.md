---
name: gsd:audit-milestone
description: アーカイブ前にマイルストーンの完了状況を元の意図と照合して監査する
argument-hint: "[version]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Task
  - Write
---
<objective>
マイルストーンが完了定義を達成したことを検証する。要件カバレッジ、クロスフェーズ統合、エンドツーエンドフローを確認する。

**このコマンドがオーケストレーターである。** 既存のVERIFICATION.mdファイル（execute-phase中に既に検証済みのフェーズ）を読み取り、技術的負債と先送りされたギャップを集約し、クロスフェーズの接続を確認するための統合チェッカーを生成する。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/audit-milestone.md
</execution_context>

<context>
バージョン: $ARGUMENTS (省略可 — デフォルトは現在のマイルストーン)

コア計画ファイルはワークフロー内で解決され (`init milestone-op`)、必要に応じてのみ読み込まれる。

**完了した作業:**
Glob: .planning/phases/*/*-SUMMARY.md
Glob: .planning/phases/*/*-VERIFICATION.md
</context>

<process>
@~/.claude/get-shit-done/workflows/audit-milestone.md のaudit-milestoneワークフローをエンドツーエンドで実行する。
すべてのワークフローゲートを保持する（スコープ決定、検証読み取り、統合チェック、要件カバレッジ、ルーティング）。
</process>

