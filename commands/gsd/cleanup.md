---
name: gsd:cleanup
description: 完了したマイルストーンから蓄積されたフェーズディレクトリをアーカイブする
---
<objective>
完了したマイルストーンのフェーズディレクトリを `.planning/milestones/v{X.Y}-phases/` にアーカイブする。

`.planning/phases/` に過去のマイルストーンのディレクトリが蓄積されている場合に使用する。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/cleanup.md
</execution_context>

<process>
@~/.claude/get-shit-done/workflows/cleanup.md のクリーンアップワークフローに従う。
完了したマイルストーンを特定し、ドライラン概要を表示し、確認後にアーカイブを実行する。
</process>
