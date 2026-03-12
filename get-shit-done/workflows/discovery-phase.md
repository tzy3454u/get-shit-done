<purpose>
適切な深度レベルでディスカバリーを実行する。
PLAN.md作成に情報を提供するDISCOVERY.md（レベル2-3用）を生成する。

plan-phase.mdのmandatory_discoveryステップから深度パラメーター付きで呼び出される。

注意：包括的なエコシステムリサーチ（「専門家がこれをどう構築するか」）には、RESEARCH.mdを生成する/gsd:research-phaseを代わりに使用すること。
</purpose>

<depth_levels>
**このワークフローは3つの深度レベルをサポートする：**

| レベル | 名前         | 時間      | 出力                                       | 使用場面                                      |
| ----- | ------------ | --------- | -------------------------------------------- | ----------------------------------------- |
| 1     | Quick Verify | 2-5分   | ファイルなし、検証済み知識で続行     | 単一ライブラリ、現在の構文の確認 |
| 2     | Standard     | 15-30分 | DISCOVERY.md                                 | オプション間の選択、新しい統合 |
| 3     | Deep Dive    | 1時間以上   | バリデーションゲート付きの詳細なDISCOVERY.md  | アーキテクチャ上の決定、新規の問題   |

**深度はここにルーティングされる前にplan-phase.mdによって決定される。**
</depth_levels>

<source_hierarchy>
**必須：WebSearchの前にContext7**

Claudeのトレーニングデータは6-18ヶ月古い。常に検証すること。

1. **Context7 MCPを最初に** - 最新のドキュメント、ハルシネーションなし
2. **公式ドキュメント** - Context7がカバーしていない場合
3. **WebSearchは最後** - 比較とトレンドのみ

完全なプロトコルについては~/.claude/get-shit-done/templates/discovery.md `<discovery_protocol>`を参照。
</source_hierarchy>

<process>

<step name="determine_depth">
plan-phase.mdから渡された深度パラメーターを確認：
- `depth=verify` → レベル1（クイック検証）
- `depth=standard` → レベル2（標準ディスカバリー）
- `depth=deep` → レベル3（ディープダイブ）

以下の適切なレベルのワークフローにルーティング。
</step>

<step name="level_1_quick_verify">
**レベル1：クイック検証（2-5分）**

対象：単一の既知ライブラリ、構文/バージョンがまだ正しいか確認。

**プロセス：**

1. Context7でライブラリを解決：

   ```
   mcp__context7__resolve-library-id with libraryName: "[library]"
   ```

2. 関連するドキュメントを取得：

   ```
   mcp__context7__get-library-docs with:
   - context7CompatibleLibraryID: [from step 1]
   - topic: [specific concern]
   ```

3. 検証：

   - 現在のバージョンが期待と一致
   - API構文が変更なし
   - 最近のバージョンで破壊的変更なし

4. **検証済みの場合：** 確認とともにplan-phase.mdに戻る。DISCOVERY.mdは不要。

5. **懸念が見つかった場合：** レベル2にエスカレート。

**出力：** 続行の口頭確認、またはレベル2へのエスカレーション。
</step>

<step name="level_2_standard">
**レベル2：標準ディスカバリー（15-30分）**

対象：オプション間の選択、新しい外部統合。

**プロセス：**

1. **何を発見すべきか特定：**

   - どんなオプションが存在するか？
   - 主要な比較基準は何か？
   - 具体的なユースケースは何か？

2. **各オプションにContext7：**

   ```
   各ライブラリ/フレームワークについて：
   - mcp__context7__resolve-library-id
   - mcp__context7__get-library-docs (mode: "code" for API, "info" for concepts)
   ```

3. Context7が不足している部分は**公式ドキュメント**。

4. 比較に**WebSearch**：

   - "[option A] vs [option B] {current_year}"
   - "[option] known issues"
   - "[option] with [our stack]"

5. **クロス検証：** WebSearchの発見 → Context7/公式ドキュメントで確認。

6. ~/.claude/get-shit-done/templates/discovery.mdの構造で**DISCOVERY.mdを作成**：

   - 推奨付きのサマリー
   - オプションごとの主要な発見
   - Context7からのコード例
   - 信頼度レベル（レベル2ではMEDIUM-HIGHであるべき）

7. plan-phase.mdに戻る。

**出力：** `.planning/phases/XX-name/DISCOVERY.md`
</step>

<step name="level_3_deep_dive">
**レベル3：ディープダイブ（1時間以上）**

対象：アーキテクチャ上の決定、新規の問題、高リスクの選択。

**プロセス：**

1. ~/.claude/get-shit-done/templates/discovery.mdを使用して**ディスカバリーのスコープを定義**：

   - 明確なスコープを定義
   - 含む/除外の境界を定義
   - 回答すべき具体的な質問をリスト

2. **網羅的なContext7リサーチ：**

   - すべての関連ライブラリ
   - 関連するパターンとコンセプト
   - 必要に応じてライブラリごとに複数トピック

3. **公式ドキュメントの深い読み込み：**

   - アーキテクチャガイド
   - ベストプラクティスセクション
   - マイグレーション/アップグレードガイド
   - 既知の制限

4. **エコシステムコンテキストのためのWebSearch：**

   - 他の人が同様の問題をどう解決したか
   - 本番環境での経験
   - 落とし穴とアンチパターン
   - 最近の変更/アナウンス

5. **すべての発見をクロス検証：**

   - すべてのWebSearchの主張 → 権威あるソースで検証
   - 検証済みと仮定をマーク
   - 矛盾にフラグを立てる

6. **包括的なDISCOVERY.mdを作成：**

   - ~/.claude/get-shit-done/templates/discovery.mdからの完全な構造
   - ソース帰属付きの品質レポート
   - 発見ごとの信頼度
   - いずれかのクリティカルな発見でLOW信頼度の場合 → バリデーションチェックポイントを追加

7. **信頼度ゲート：** 全体の信頼度がLOWの場合、続行前にオプションを提示。

8. plan-phase.mdに戻る。

**出力：** `.planning/phases/XX-name/DISCOVERY.md`（包括的）
</step>

<step name="identify_unknowns">
**レベル2-3用：** 何を学ぶ必要があるかを定義する。

問い：このフェーズを計画する前に何を学ぶ必要があるか？

- 技術選択？
- ベストプラクティス？
- APIパターン？
- アーキテクチャアプローチ？
  </step>

<step name="create_discovery_scope">
~/.claude/get-shit-done/templates/discovery.mdを使用する。

含める内容：

- 明確なディスカバリー目標
- スコープ付きの含む/除外リスト
- ソース優先度（公式ドキュメント、Context7、今年度）
- DISCOVERY.mdの出力構造
  </step>

<step name="execute_discovery">
ディスカバリーを実行する：
- 最新情報にWeb検索を使用
- ライブラリドキュメントにContext7 MCPを使用
- 今年度のソースを優先
- テンプレートに従って発見を構造化
</step>

<step name="create_discovery_output">
`.planning/phases/XX-name/DISCOVERY.md`を書き込む：
- 推奨付きのサマリー
- ソース付きの主要な発見
- 該当する場合はコード例
- メタデータ（信頼度、依存関係、オープンクエスチョン、前提）
</step>

<step name="confidence_gate">
DISCOVERY.md作成後、信頼度レベルを確認する。

信頼度がLOWの場合：
AskUserQuestionを使用：

- header: "Low Conf."
- question: "ディスカバリーの信頼度がLOWです：[理由]。どのように進めますか？"
- options:
  - "Dig deeper" - 計画前にさらにリサーチする
  - "Proceed anyway" - 不確実性を受け入れ、注意事項付きで計画
  - "Pause" - 考える必要がある

信頼度がMEDIUMの場合：
インライン：「ディスカバリー完了（中程度の信頼度）。[簡単な理由]。計画に進みますか？」

信頼度がHIGHの場合：
直接続行、記載のみ：「ディスカバリー完了（高信頼度）。」
</step>

<step name="open_questions_gate">
DISCOVERY.mdにopen_questionsがある場合：

インラインで提示：
「ディスカバリーからのオープンクエスチョン：

- [質問1]
- [質問2]

これらは実装に影響する可能性があります。確認して進みますか？ (yes / address first)」

"address first"の場合：質問に対するユーザー入力を収集し、ディスカバリーを更新。
</step>

<step name="offer_next">
```
ディスカバリー完了: .planning/phases/XX-name/DISCOVERY.md
推奨: [一行サマリー]
信頼度: [レベル]

次のステップ：

1. フェーズコンテキストについてディスカッション (/gsd:discuss-phase [current-phase])
2. フェーズプランを作成 (/gsd:plan-phase [current-phase])
3. ディスカバリーを精緻化（さらに深掘り）
4. ディスカバリーを確認

```

注意：DISCOVERY.mdは個別にコミットされない。フェーズ完了時にコミットされる。
</step>

</process>

<success_criteria>
**レベル1（クイック検証）：**
- ライブラリ/トピックについてContext7を参照
- 現在の状態が検証されたか、懸念がエスカレートされた
- 続行の口頭確認（ファイルなし）

**レベル2（標準）：**
- すべてのオプションについてContext7を参照
- WebSearchの発見がクロス検証された
- 推奨付きのDISCOVERY.mdが作成された
- 信頼度レベルがMEDIUM以上
- PLAN.md作成に情報提供の準備完了

**レベル3（ディープダイブ）：**
- ディスカバリーのスコープが定義された
- Context7が網羅的に参照された
- すべてのWebSearchの発見が権威あるソースで検証された
- 包括的な分析付きのDISCOVERY.mdが作成された
- ソース帰属付きの品質レポート
- LOW信頼度の発見がある場合 → バリデーションチェックポイントが定義された
- 信頼度ゲートに合格
- PLAN.md作成に情報提供の準備完了
</success_criteria>
