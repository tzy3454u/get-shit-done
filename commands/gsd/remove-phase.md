---
name: gsd:remove-phase
description: ロードマップから未来のフェーズを削除し、後続フェーズを再番号付けする
argument-hint: <phase-number>
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---
<objective>
ロードマップから未開始の未来のフェーズを削除し、クリーンで連続的な番号順を維持するためにすべての後続フェーズを再番号付けする。

目的：実施しないと決定した作業を、キャンセル済み/延期マーカーでコンテキストを汚すことなくクリーンに削除する。
出力：フェーズが削除され、すべての後続フェーズが再番号付けされ、履歴記録として Git コミットされる。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/remove-phase.md
</execution_context>

<context>
フェーズ: $ARGUMENTS

ロードマップと状態はワークフロー内で `init phase-op` と対象ファイルの読み取りにより解決される。
</context>

<process>
@~/.claude/get-shit-done/workflows/remove-phase.md の remove-phase ワークフローをエンドツーエンドで実行すること。
すべてのバリデーションゲート（未来フェーズチェック、作業チェック）、再番号付けロジック、およびコミットを維持すること。
</process>
