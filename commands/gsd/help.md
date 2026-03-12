---
name: gsd:help
description: 利用可能なGSDコマンドと使い方ガイドを表示
---
<objective>
完全なGSDコマンドリファレンスを表示します。

以下の内容のみを出力してください。以下を追加しないでください:
- プロジェクト固有の分析
- Gitステータスやファイルコンテキスト
- 次のステップの提案
- リファレンス以外のコメント
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/help.md
</execution_context>

<process>
@~/.claude/get-shit-done/workflows/help.md の完全なGSDコマンドリファレンスを出力します。
リファレンスの内容をそのまま表示してください — 追加や変更はしないでください。
</process>
</output>
