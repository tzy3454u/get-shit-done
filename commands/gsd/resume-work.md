---
name: gsd:resume-work
description: 前回のセッションから完全なコンテキスト復元で作業を再開する
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
  - SlashCommand
---

<objective>
プロジェクトの完全なコンテキストを復元し、前回のセッションからシームレスに作業を再開します。

resume-project ワークフローにルーティングし、以下を処理します：

- STATE.md の読み込み（欠落時は再構築）
- チェックポイントの検出（.continue-here ファイル）
- 未完了作業の検出（SUMMARY のない PLAN）
- ステータスの表示
- コンテキストに応じた次のアクションへのルーティング
  </objective>

<execution_context>
@~/.claude/get-shit-done/workflows/resume-project.md
</execution_context>

<process>
`@~/.claude/get-shit-done/workflows/resume-project.md` の **resume-project ワークフローに従ってください**。

ワークフローは以下のすべての再開ロジックを処理します：

1. プロジェクトの存在確認
2. STATE.md の読み込みまたは再構築
3. チェックポイントと未完了作業の検出
4. ビジュアルなステータス表示
5. コンテキストに応じたオプション提示（plan vs discuss を提案する前に CONTEXT.md を確認）
6. 適切な次のコマンドへのルーティング
7. セッション継続性の更新
   </process>
