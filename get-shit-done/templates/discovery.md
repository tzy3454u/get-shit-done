# ディスカバリーテンプレート

`.planning/phases/XX-name/DISCOVERY.md` 用テンプレート - ライブラリ/オプション選定のための浅いリサーチ。

**目的:** plan-phaseでの必須ディスカバリー中に「どのライブラリ/オプションを使用すべきか」の質問に答えます。

深いエコシステムリサーチ（「専門家はこれをどう構築するか」）には、RESEARCH.mdを生成する`/gsd:research-phase`を使用してください。

---

## ファイルテンプレート

```markdown
---
phase: XX-name
type: discovery
topic: [discovery-topic]
---

<session_initialization>
ディスカバリー開始前に、今日の日付を確認してください:
!`date +%Y-%m-%d`

「現在」や「最新」の情報を検索する際にこの日付を使用してください。
例: 今日が2025-11-22の場合、「2024」ではなく「2025」で検索してください。
</session_initialization>

<discovery_objective>
[フェーズ名]の実装に向けて[トピック]をディスカバリーします。

Purpose: [この判断/実装が何を可能にするか]
Scope: [境界]
Output: 推奨事項付きDISCOVERY.md
</discovery_objective>

<discovery_scope>
<include>
- [回答すべき質問]
- [調査すべき領域]
- [必要な場合の具体的な比較]
</include>

<exclude>
- [このディスカバリーのスコープ外]
- [実装フェーズに先送り]
</exclude>
</discovery_scope>

<discovery_protocol>

**ソースの優先順位:**
1. **Context7 MCP** - ライブラリ/フレームワークのドキュメント用（最新、権威ある）
2. **公式ドキュメント** - プラットフォーム固有またはインデックスされていないライブラリ用
3. **WebSearch** - 比較、トレンド、コミュニティパターン用（すべての発見を検証）

**品質チェックリスト:**
ディスカバリー完了前に確認:
- [ ] すべての主張に権威あるソース（Context7または公式ドキュメント）がある
- [ ] 否定的な主張（「Xは不可能」）が公式ドキュメントで検証されている
- [ ] API構文/設定がContext7または公式ドキュメントからのもの（WebSearch単独ではない）
- [ ] WebSearchの発見が権威あるソースでクロスチェックされている
- [ ] 最近の更新/変更ログで破壊的変更が確認されている
- [ ] 代替アプローチが検討されている（最初に見つかった解決策だけではない）

**信頼度レベル:**
- HIGH: Context7または公式ドキュメントが確認
- MEDIUM: WebSearch + Context7/公式ドキュメントが確認
- LOW: WebSearchのみまたはトレーニング知識のみ（検証のためにマーク）

</discovery_protocol>


<output_structure>
`.planning/phases/XX-name/DISCOVERY.md`を作成:

```markdown
# [トピック] ディスカバリー

## サマリー
[2〜3段落のエグゼクティブサマリー - 何を調査し、何を発見し、何を推奨するか]

## 主な推奨事項
[何をすべきか、なぜか - 具体的で実行可能に]

## 検討した代替案
[他に何を評価し、なぜ選ばなかったか]

## 主な発見

### [カテゴリ1]
- [ソースURLと自分たちのケースへの関連性を含む発見]

### [カテゴリ2]
- [ソースURLと関連性を含む発見]

## コード例
[関連する実装パターン（該当する場合）]

## メタデータ

<metadata>
<confidence level="high|medium|low">
[なぜこの信頼度レベルか - ソースの品質と検証に基づく]
</confidence>

<sources>
- [使用した主な権威あるソース]
</sources>

<open_questions>
[判断できなかったこと、実装中に検証が必要なこと]
</open_questions>

<validation_checkpoints>
[信頼度がLOWまたはMEDIUMの場合、実装中に検証すべき具体的な項目をリスト]
</validation_checkpoints>
</metadata>
```
</output_structure>

<success_criteria>
- すべてのスコープの質問が権威あるソースで回答されている
- 品質チェックリスト項目が完了している
- 明確な主な推奨事項がある
- 低信頼度の発見に検証チェックポイントが付いている
- PLAN.md作成に情報提供する準備ができている
</success_criteria>

<guidelines>
**ディスカバリーを使用するタイミング:**
- 技術選択が不明確な場合（ライブラリAとBの比較）
- 不慣れな統合のベストプラクティスが必要な場合
- API/ライブラリの調査が必要な場合
- 単一の判断が保留中の場合

**使用しないタイミング:**
- 確立されたパターン（CRUD、既知のライブラリによる認証）
- 実装の詳細（実行時に先送り）
- 既存のプロジェクトコンテキストで回答可能な質問

**代わりにRESEARCH.mdを使用するタイミング:**
- ニッチ/複雑なドメイン（3D、ゲーム、オーディオ、シェーダー）
- ライブラリの選択だけでなくエコシステムの知識が必要な場合
- 「専門家はこれをどう構築するか」の質問
- これらには`/gsd:research-phase`を使用してください
</guidelines>
</output>