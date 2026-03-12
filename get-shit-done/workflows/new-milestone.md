<purpose>

既存プロジェクトの新しいマイルストーンサイクルを開始する。プロジェクトコンテキストを読み込み、マイルストーン目標を収集し（MILESTONE-CONTEXT.mdまたは会話から）、PROJECT.mdとSTATE.mdを更新し、オプションで並列リサーチを実行し、REQ-ID付きのスコープ付き要件を定義し、ロードマッパーを生成してフェーズ分けされた実行計画を作成し、すべてのアーティファクトをコミットする。new-projectのブラウンフィールド版。

</purpose>

<required_reading>

開始前に、呼び出し元プロンプトのexecution_contextで参照されているすべてのファイルを読み込むこと。

</required_reading>

<process>

## 1. コンテキストの読み込み

- PROJECT.md を読み込む（既存プロジェクト、検証済み要件、決定事項）
- MILESTONES.md を読み込む（以前の出荷内容）
- STATE.md を読み込む（保留中のTODO、ブロッカー）
- MILESTONE-CONTEXT.md を確認する（/gsd:discuss-milestone から）

## 2. マイルストーン目標の収集

**MILESTONE-CONTEXT.md が存在する場合：**
- discuss-milestoneからの機能とスコープを使用する
- サマリーを確認のために表示する

**コンテキストファイルがない場合：**
- 前回のマイルストーンで出荷された内容を表示する
- インラインで確認する（フリーフォーム、AskUserQuestionではなく）："次に何を構築しますか？"
- 返信を待ち、AskUserQuestionを使って詳細を掘り下げる
- ユーザーがいずれかの時点で "Other" を選択してフリーフォーム入力した場合、フォローアップはプレーンテキストで質問する — 別のAskUserQuestionではなく

## 3. マイルストーンバージョンの決定

- MILESTONES.mdから最新バージョンをパースする
- 次のバージョンを提案する（v1.0 → v1.1、またはメジャーの場合はv2.0）
- ユーザーに確認する

## 4. PROJECT.mdの更新

追加/更新する：

```markdown
## Current Milestone: v[X.Y] [名前]

**Goal:** [マイルストーンのフォーカスを1文で記述]

**Target features:**
- [機能1]
- [機能2]
- [機能3]
```

Active要件セクションと "Last updated" フッターを更新する。

## 5. STATE.mdの更新

```markdown
## Current Position

Phase: Not started (defining requirements)
Plan: —
Status: Defining requirements
Last activity: [today] — Milestone v[X.Y] started
```

前回のマイルストーンのAccumulated Contextセクションを保持する。

## 6. クリーンアップとコミット

MILESTONE-CONTEXT.mdが存在する場合は削除する（消費済み）。

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: start milestone v[X.Y] [Name]" --files .planning/PROJECT.md .planning/STATE.md
```

## 7. コンテキストの読み込みとモデルの解決

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init new-milestone)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから抽出する：`researcher_model`、`synthesizer_model`、`roadmapper_model`、`commit_docs`、`research_enabled`、`current_milestone`、`project_exists`、`roadmap_exists`。

## 8. リサーチの判断

AskUserQuestion: "要件定義の前に新機能のドメインエコシステムをリサーチしますか？"
- "先にリサーチ（推奨）" — 新しい機能のパターン、特徴、アーキテクチャを調査する
- "リサーチをスキップ" — 直接要件に進む

**選択を設定に保存する**（今後の `/gsd:plan-phase` がそれを尊重するように）：

```bash
# "先にリサーチ" の場合：trueを保存
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow.research true

# "リサーチをスキップ" の場合：falseを保存
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow.research false
```

**"先にリサーチ" の場合：**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► リサーチ中
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ 4つのリサーチャーを並列で生成中...
  → Stack, Features, Architecture, Pitfalls
```

```bash
mkdir -p .planning/research
```

4つの並列gsd-project-researcherエージェントを生成する。各エージェントはディメンション固有のフィールドを持つこのテンプレートを使用する：

**4つのリサーチャー共通の構造：**
```
Task(prompt="
<research_type>プロジェクトリサーチ — [新機能]のための{DIMENSION}。</research_type>

<milestone_context>
後続マイルストーン — 既存アプリに[対象機能]を追加。
{EXISTING_CONTEXT}
新機能に必要なもののみにフォーカス。
</milestone_context>

<question>{QUESTION}</question>

<files_to_read>
- .planning/PROJECT.md (プロジェクトコンテキスト)
</files_to_read>

<downstream_consumer>{CONSUMER}</downstream_consumer>

<quality_gate>{GATES}</quality_gate>

<output>
Write to: .planning/research/{FILE}
Use template: ~/.claude/get-shit-done/templates/research-project/{FILE}
</output>
", subagent_type="gsd-project-researcher", model="{researcher_model}", description="{DIMENSION} research")
```

**ディメンション固有のフィールド：**

| Field | Stack | Features | Architecture | Pitfalls |
|-------|-------|----------|-------------|----------|
| EXISTING_CONTEXT | 既存の検証済み機能（再リサーチ不要）：[PROJECT.mdから] | 既存の機能（構築済み）：[PROJECT.mdから] | 既存のアーキテクチャ：[PROJECT.mdまたはコードベースマップから] | 既存システムにこれらの機能を追加する際の一般的なミスにフォーカス |
| QUESTION | [新機能]にどのようなスタック追加/変更が必要か？ | [対象機能]は通常どのように機能するか？期待される動作は？ | [対象機能]は既存のアーキテクチャとどのように統合するか？ | [ドメイン]に[対象機能]を追加する際の一般的なミスは？ |
| CONSUMER | 新機能用の具体的なライブラリとバージョン、統合ポイント、追加すべきでないもの | テーブルステークス vs 差別化要因 vs アンチフィーチャー、複雑さの注記、既存への依存関係 | 統合ポイント、新コンポーネント、データフローの変更、推奨ビルド順序 | 警告サイン、防止戦略、どのフェーズで対処すべきか |
| GATES | バージョンが最新（Context7で検証）、WHYの根拠を説明、統合が考慮されている | カテゴリが明確、複雑さが注記されている、依存関係が識別されている | 統合ポイントが識別されている、新規vs修正が明示的、ビルド順序が依存関係を考慮 | これらの機能追加に固有の落とし穴、統合の落とし穴がカバーされている、防止策が実行可能 |
| FILE | STACK.md | FEATURES.md | ARCHITECTURE.md | PITFALLS.md |

4つすべて完了後、シンセサイザーを生成する：

```
Task(prompt="
リサーチ出力をSUMMARY.mdに統合する。

<files_to_read>
- .planning/research/STACK.md
- .planning/research/FEATURES.md
- .planning/research/ARCHITECTURE.md
- .planning/research/PITFALLS.md
</files_to_read>

Write to: .planning/research/SUMMARY.md
Use template: ~/.claude/get-shit-done/templates/research-project/SUMMARY.md
書き込み後にコミット。
", subagent_type="gsd-research-synthesizer", model="{synthesizer_model}", description="Synthesize research")
```

SUMMARY.mdから主な発見を表示する：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► リサーチ完了 ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**スタック追加:** [SUMMARY.mdから]
**機能のテーブルステークス:** [SUMMARY.mdから]
**注意すべき点:** [SUMMARY.mdから]
```

**"リサーチをスキップ" の場合：** ステップ9に進む。

## 9. 要件の定義

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► 要件を定義中
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

PROJECT.mdを読み込む：core value、現在のマイルストーン目標、検証済み要件（既存のもの）。

**リサーチが存在する場合：** FEATURES.mdを読み込み、機能カテゴリを抽出する。

カテゴリ別に機能を表示する：
```
## [カテゴリ1]
**テーブルステークス:** 機能A、機能B
**差別化要因:** 機能C、機能D
**リサーチメモ:** [関連メモ]
```

**リサーチがない場合：** 会話で要件を収集する。質問する："[新機能]でユーザーが行う必要がある主なことは何ですか？" 明確化し、関連する機能を調査し、カテゴリにグループ化する。

**各カテゴリのスコープを設定する** AskUserQuestion経由（multiSelect: true、ヘッダー最大12文字）：
- "[機能1]" — [簡潔な説明]
- "[機能2]" — [簡潔な説明]
- "このマイルストーンでは含めない" — カテゴリ全体を延期

追跡する：選択済み → このマイルストーン。未選択のテーブルステークス → 将来。未選択の差別化要因 → スコープ外。

**ギャップを識別する** AskUserQuestion経由：
- "いいえ、リサーチでカバーされています" — 続行
- "はい、追加します" — 追加分を収集

**REQUIREMENTS.mdを生成する：**
- カテゴリ別にグループ化されたv1要件（チェックボックス、REQ-ID）
- 将来の要件（延期）
- スコープ外（理由付きの明示的な除外）
- トレーサビリティセクション（空、ロードマップで記入）

**REQ-IDフォーマット：** `[CATEGORY]-[NUMBER]` (AUTH-01, NOTIF-02)。既存の番号から続ける。

**要件の品質基準：**

良い要件の条件：
- **具体的でテスト可能：** "ユーザーはメールリンクでパスワードをリセットできる" ("パスワードリセットを処理する" ではない)
- **ユーザー中心：** "ユーザーはXできる" ("システムがYを行う" ではない)
- **アトミック：** 1要件に1機能 ("ユーザーはログインしてプロフィールを管理できる" ではない)
- **独立：** 他の要件への依存を最小限に

完全な要件リストを確認のために表示する：

```
## マイルストーン v[X.Y] 要件

### [カテゴリ1]
- [ ] **CAT1-01**: ユーザーはXできる
- [ ] **CAT1-02**: ユーザーはYできる

### [カテゴリ2]
- [ ] **CAT2-01**: ユーザーはZできる

構築する内容をこれで網羅していますか？ (yes / adjust)
```

"adjust" の場合：スコープ設定に戻る。

**要件をコミットする：**
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: define milestone v[X.Y] requirements" --files .planning/REQUIREMENTS.md
```

## 10. ロードマップの作成

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► ロードマップを作成中
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ ロードマッパーを生成中...
```

**開始フェーズ番号：** MILESTONES.mdから最後のフェーズ番号を読み取る。そこから続ける（v1.0がフェーズ5で終了 → v1.1はフェーズ6から開始）。

```
Task(prompt="
<planning_context>
<files_to_read>
- .planning/PROJECT.md
- .planning/REQUIREMENTS.md
- .planning/research/SUMMARY.md (if exists)
- .planning/config.json
- .planning/MILESTONES.md
</files_to_read>
</planning_context>

<instructions>
マイルストーン v[X.Y] のロードマップを作成する：
1. フェーズ番号を [N] から開始
2. このマイルストーンの要件のみからフェーズを導出
3. すべての要件を正確に1つのフェーズにマッピング
4. フェーズごとに2-5個の成功基準を導出（観察可能なユーザー行動）
5. 100%のカバレッジを検証
6. ファイルを即座に書き込む（ROADMAP.md、STATE.md、REQUIREMENTS.mdのトレーサビリティを更新）
7. ROADMAP CREATEDをサマリー付きで返す

先にファイルを書き込み、その後返す。
</instructions>
", subagent_type="gsd-roadmapper", model="{roadmapper_model}", description="Create roadmap")
```

**戻り値の処理：**

**`## ROADMAP BLOCKED` の場合：** ブロッカーを表示し、ユーザーと解決し、再生成する。

**`## ROADMAP CREATED` の場合：** ROADMAP.mdを読み込み、インラインで表示する：

```
## 提案されたロードマップ

**[N] フェーズ** | **[X] 要件マッピング済み** | すべてカバー ✓

| # | Phase | Goal | Requirements | Success Criteria |
|---|-------|------|--------------|------------------|
| [N] | [名前] | [目標] | [REQ-ID] | [数] |

### フェーズ詳細

**Phase [N]: [名前]**
目標: [目標]
要件: [REQ-ID]
成功基準:
1. [基準]
2. [基準]
```

**承認を求める** AskUserQuestion経由：
- "承認" — コミットして続行
- "フェーズを調整" — 変更内容を伝えてください
- "完全なファイルを確認" — 生のROADMAP.mdを表示

**"調整" の場合：** メモを取得し、修正コンテキスト付きでロードマッパーを再生成し、承認されるまでループする。
**"確認" の場合：** 生のROADMAP.mdを表示し、再度質問する。

**ロードマップをコミットする**（承認後）：
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: create milestone v[X.Y] roadmap ([N] phases)" --files .planning/ROADMAP.md .planning/STATE.md .planning/REQUIREMENTS.md
```

## 11. 完了

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► マイルストーン初期化完了 ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**マイルストーン v[X.Y]: [名前]**

| Artifact       | Location                    |
|----------------|-----------------------------|
| Project        | `.planning/PROJECT.md`      |
| Research       | `.planning/research/`       |
| Requirements   | `.planning/REQUIREMENTS.md` |
| Roadmap        | `.planning/ROADMAP.md`      |

**[N] フェーズ** | **[X] 要件** | ビルド準備完了 ✓

## ▶ Next Up

**Phase [N]: [フェーズ名]** — [目標]

`/gsd:discuss-phase [N]` — コンテキストを収集しアプローチを明確化

<sub>`/clear` first → fresh context window</sub>

その他: `/gsd:plan-phase [N]` — ディスカッションをスキップして直接計画
```

</process>

<success_criteria>
- [ ] PROJECT.mdがCurrent Milestoneセクションで更新されている
- [ ] STATE.mdが新しいマイルストーン用にリセットされている
- [ ] MILESTONE-CONTEXT.mdが消費されて削除されている（存在した場合）
- [ ] リサーチが完了している（選択された場合） — 4つの並列エージェント、マイルストーン対応
- [ ] 要件がカテゴリごとに収集されスコープ設定されている
- [ ] REQUIREMENTS.mdがREQ-ID付きで作成されている
- [ ] gsd-roadmapperがフェーズ番号コンテキスト付きで生成されている
- [ ] ロードマップファイルが即座に書き込まれている（ドラフトではない）
- [ ] ユーザーのフィードバックが反映されている（ある場合）
- [ ] ROADMAP.mdのフェーズが前回のマイルストーンから続いている
- [ ] すべてのコミットが作成されている（計画ドキュメントがコミットされる場合）
- [ ] ユーザーが次のステップを知っている：`/gsd:discuss-phase [N]`

**アトミックコミット：** 各フェーズはアーティファクトを即座にコミットする。
</success_criteria>
</output>
