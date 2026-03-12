---
name: gsd:insert-phase
description: 既存フェーズの間に小数フェーズ（例: 72.1）として緊急作業を挿入する
argument-hint: <after> <description>
allowed-tools:
  - Read
  - Write
  - Bash
---

<objective>
マイルストーン進行中に発見された緊急作業を、既存の整数フェーズ間で完了させるために小数フェーズとして挿入する。

小数ナンバリング（72.1、72.2 など）を使用して、計画済みフェーズの論理的な順序を保ちながら緊急の挿入に対応する。

目的: ロードマップ全体の番号を振り直すことなく、実行中に発見された緊急作業を処理する。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/insert-phase.md
</execution_context>

<context>
引数: $ARGUMENTS（形式: <対象フェーズ番号> <説明>）

ロードマップと状態はワークフロー内で `init phase-op` と対象ツール呼び出しを通じて解決される。
</context>

<process>
@~/.claude/get-shit-done/workflows/insert-phase.md のフェーズ挿入ワークフローをエンドツーエンドで実行する。
すべてのバリデーションゲート（引数解析、フェーズ検証、小数計算、ロードマップ更新）を維持する。
</process>
