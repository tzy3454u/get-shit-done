# モデルプロファイル

モデルプロファイルは、各GSDエージェントが使用するClaudeモデルを制御します。品質とトークン消費のバランスを調整できます。

## プロファイル定義

| Agent | `quality` | `balanced` | `budget` |
|-------|-----------|------------|----------|
| gsd-planner | opus | opus | sonnet |
| gsd-roadmapper | opus | sonnet | sonnet |
| gsd-executor | opus | sonnet | sonnet |
| gsd-phase-researcher | opus | sonnet | haiku |
| gsd-project-researcher | opus | sonnet | haiku |
| gsd-research-synthesizer | sonnet | sonnet | haiku |
| gsd-debugger | opus | sonnet | sonnet |
| gsd-codebase-mapper | sonnet | haiku | haiku |
| gsd-verifier | sonnet | sonnet | haiku |
| gsd-plan-checker | sonnet | sonnet | haiku |
| gsd-integration-checker | sonnet | sonnet | haiku |
| gsd-nyquist-auditor | sonnet | sonnet | haiku |

## プロファイルの思想

**quality** - 最大の推論能力
- すべての意思決定エージェントにOpusを使用
- 読み取り専用の検証にSonnetを使用
- 使用タイミング: クォータに余裕がある場合、重要なアーキテクチャ作業

**balanced**（デフォルト）- スマートな配分
- 計画にのみOpus（アーキテクチャの意思決定が行われる場所）
- 実行とリサーチにSonnet（明示的な指示に従う）
- 検証にSonnet（パターンマッチングだけでなく推論が必要）
- 使用タイミング: 通常の開発、品質とコストの良いバランス

**budget** - 最小限のOpus使用
- コードを書くすべてにSonnet
- リサーチと検証にHaiku
- 使用タイミング: クォータの節約、大量の作業、重要度の低いフェーズ

## 解決ロジック

オーケストレーターはスポーン前にモデルを解決します:

```
1. .planning/config.jsonを読み取る
2. エージェント固有のオーバーライドのmodel_overridesを確認
3. オーバーライドがなければ、プロファイルテーブルでエージェントを検索
4. Task呼び出しにmodelパラメータを渡す
```

## エージェントごとのオーバーライド

プロファイル全体を変更せずに特定のエージェントをオーバーライドします:

```json
{
  "model_profile": "balanced",
  "model_overrides": {
    "gsd-executor": "opus",
    "gsd-planner": "haiku"
  }
}
```

オーバーライドはプロファイルよりも優先されます。有効な値: `opus`, `sonnet`, `haiku`。

## プロファイルの切り替え

ランタイム: `/gsd:set-profile <profile>`

プロジェクトごとのデフォルト: `.planning/config.json`で設定:
```json
{
  "model_profile": "balanced"
}
```

## 設計の根拠

**なぜgsd-plannerにOpus?**
計画にはアーキテクチャの意思決定、目標の分解、タスク設計が含まれます。ここがモデル品質の影響が最も大きい場所です。

**なぜgsd-executorにSonnet?**
エグゼキューターは明示的なPLAN.mdの指示に従います。プランには既に推論が含まれており、実行は実装です。

**なぜbalancedで検証にSonnet（Haikuではない）?**
検証にはゴール逆引き推論が必要です - コードがフェーズの約束した内容を本当に実現しているかを確認する、単なるパターンマッチングではありません。Sonnetはこれを適切に処理しますが、Haikuは微妙なギャップを見逃す可能性があります。

**なぜgsd-codebase-mapperにHaiku?**
読み取り専用の探索とパターン抽出。推論は不要で、ファイル内容からの構造化された出力のみ。

**なぜ`opus`を直接渡すのではなく`inherit`?**
Claude Codeの`"opus"`エイリアスは特定のモデルバージョンにマッピングされます。組織は古いopusバージョンをブロックしながら新しいバージョンを許可する場合があります。GSDはopusティアのエージェントに`"inherit"`を返し、ユーザーのセッションで設定されているopusバージョンを使用させます。これによりバージョンの競合やSonnetへのサイレントフォールバックを回避します。
