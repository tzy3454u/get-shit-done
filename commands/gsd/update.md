---
name: gsd:update
description: GSD を最新バージョンに更新し、変更履歴を表示する
allowed-tools:
  - Bash
  - AskUserQuestion
---

<objective>
GSD のアップデートを確認し、利用可能であればインストールし、変更内容を表示します。

update ワークフローにルーティングし、以下を処理します：
- バージョン検出（ローカル vs グローバルインストール）
- npm バージョンチェック
- 変更履歴の取得と表示
- クリーンインストール警告付きのユーザー確認
- アップデートの実行とキャッシュクリア
- 再起動リマインダー
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/update.md
</execution_context>

<process>
`@~/.claude/get-shit-done/workflows/update.md` の **update ワークフローに従ってください**。

ワークフローは以下のすべてのロジックを処理します：
1. インストール済みバージョンの検出（ローカル/グローバル）
2. npm 経由での最新バージョンチェック
3. バージョン比較
4. 変更履歴の取得と抽出
5. クリーンインストール警告の表示
6. ユーザー確認
7. アップデートの実行
8. キャッシュクリア
</process>
