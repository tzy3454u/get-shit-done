---
phase: XX-name
plan: YY
subsystem: [主要カテゴリ]
tags: [検索可能な技術]
requires:
  - phase: [先行フェーズ]
    provides: [そのフェーズが構築したもの]
provides:
  - [構築/提供されたもののリスト]
affects: [フェーズ名やキーワードのリスト]
tech-stack:
  added: [ライブラリ/ツール]
  patterns: [アーキテクチャ/コードパターン]
key-files:
  created: [作成した重要ファイル]
  modified: [変更した重要ファイル]
key-decisions:
  - "Decision 1"
patterns-established:
  - "Pattern 1: description"
duration: Xmin
completed: YYYY-MM-DD
---

# Phase [X]: [名前] サマリー（複雑）

**[成果を説明する実質的な一行]**

## パフォーマンス
- **所要時間:** [時間]
- **タスク:** [完了数]
- **変更ファイル:** [数]

## 成果
- [主要な成果 1]
- [主要な成果 2]

## タスクコミット
1. **Task 1: [タスク名]** - `hash`
2. **Task 2: [タスク名]** - `hash`
3. **Task 3: [タスク名]** - `hash`

## 作成/変更ファイル
- `path/to/file.ts` - 役割
- `path/to/another.ts` - 役割

## 行われた決定
[重要な決定と簡単な根拠]

## プランからの逸脱（自動修正）
[GSD逸脱ルールに従った詳細な自動修正記録]

## 発生した問題
[計画された作業中の問題とその解決方法]

## 次フェーズの準備状況
[次のフェーズに向けて準備できていること]
[ブロッカーや懸念事項]
