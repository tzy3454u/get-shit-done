---
name: gsd:quick
description: GSD保証（アトミックコミット、状態追跡）付きでクイックタスクを実行し、オプションのエージェントをスキップする
argument-hint: "[--full] [--discuss]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - AskUserQuestion
---
<objective>
GSD保証（アトミックコミット、STATE.md追跡）付きで小規模なアドホックタスクを実行する。

クイックモードは同じシステムのショートパス:
- gsd-planner（クイックモード）+ gsd-executor(s)を生成
- クイックタスクは計画済みフェーズとは別に `.planning/quick/` に保存
- STATE.mdの「Quick Tasks Completed」テーブルを更新（ROADMAP.mdではない）

**デフォルト:** リサーチ、ディスカッション、プランチェッカー、ベリファイアをスキップ。何をすべきか正確に分かっている場合に使用。

**`--discuss` フラグ:** 計画前の軽量ディスカッションフェーズ。前提を明らかにし、曖昧な部分を明確にし、決定事項をCONTEXT.mdに記録する。事前に解消すべき曖昧さがあるタスクに使用。

**`--full` フラグ:** プランチェック（最大2回の反復）と実行後の検証を有効にする。完全なマイルストーンセレモニーなしで品質保証が必要な場合に使用。

フラグは組み合わせ可能: `--discuss --full` でディスカッション + プランチェック + 検証が得られる。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/quick.md
</execution_context>

<context>
$ARGUMENTS

コンテキストファイルはワークフロー内で解決され (`init quick`)、`<files_to_read>` ブロックを通じて委譲される。
</context>

<process>
@~/.claude/get-shit-done/workflows/quick.md のクイックワークフローをエンドツーエンドで実行する。
すべてのワークフローゲートを保持する（バリデーション、タスク説明、計画、実行、状態更新、コミット）。
</process>
</output>
