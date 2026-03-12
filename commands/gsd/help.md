---
name: gsd:help
description: 利用可能なGSDコマンドと使い方ガイドを表示
---
<objective>
完全なGSDコマンドリファレンスを表示する。

以下の内容のみを出力すること。以下を追加しないこと:
- プロジェクト固有の分析
- Gitステータスやファイルコンテキスト
- 次のステップの提案
- リファレンス以外のコメント
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/help.md
</execution_context>

<process>
@~/.claude/get-shit-done/workflows/help.md の完全なGSDコマンドリファレンスを出力する。
リファレンスの内容をそのまま表示すること — 追加や変更はしないこと。
</process>

