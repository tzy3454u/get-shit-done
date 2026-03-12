---
name: gsd:research-phase
description: フェーズの実装方法をリサーチ（スタンドアロン — 通常は/gsd:plan-phaseを使用）
argument-hint: "[phase]"
allowed-tools:
  - Read
  - Bash
  - Task
---

<objective>
フェーズの実装方法をリサーチする。フェーズコンテキスト付きでgsd-phase-researcherエージェントを起動する。

**注意:** これはスタンドアロンのリサーチコマンド。ほとんどのワークフローでは、リサーチを自動的に統合する`/gsd:plan-phase`を使用すること。

**このコマンドを使用する場合:**
- 計画を立てる前にリサーチだけしたい場合
- 計画完了後にリサーチをやり直したい場合
- フェーズが実現可能かどうか判断する前に調査が必要な場合

**オーケストレーターの役割:** フェーズの解析、ロードマップに対する検証、既存リサーチの確認、コンテキストの収集、リサーチャーエージェントの起動、結果の提示。

**サブエージェントを使う理由:** リサーチはコンテキストを急速に消費する（WebSearch、Context7クエリ、ソース検証）。調査にフレッシュな200kコンテキストを確保。メインコンテキストはユーザーとのやり取りのためにスリムに保つ。
</objective>

<context>
フェーズ番号: $ARGUMENTS（必須）

ステップ1でディレクトリ検索を行う前にフェーズ入力を正規化すること。
</context>

<process>

## 0. コンテキストの初期化

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "$ARGUMENTS")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから抽出: `phase_dir`、`phase_number`、`phase_name`、`phase_found`、`commit_docs`、`has_research`、`state_path`、`requirements_path`、`context_path`、`research_path`。

リサーチャーモデルを解決:
```bash
RESEARCHER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-phase-researcher --raw)
```

## 1. フェーズの検証

```bash
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${phase_number}")
```

**`found`がfalseの場合:** エラーを出して終了。 **`found`がtrueの場合:** JSONから`phase_number`、`phase_name`、`goal`を抽出。

## 2. 既存リサーチの確認

```bash
ls .planning/phases/${PHASE}-*/RESEARCH.md 2>/dev/null
```

**存在する場合:** 提示: 1) リサーチを更新、2) 既存を表示、3) スキップ。応答を待つ。

**存在しない場合:** 続行。

## 3. フェーズコンテキストの収集

INITからのパスを使用（オーケストレーターコンテキストにファイル内容をインラインしない）:
- `requirements_path`
- `context_path`
- `state_path`

フェーズの説明とリサーチャーがロードするファイルの概要を提示。

## 4. gsd-phase-researcherエージェントの起動

リサーチモード: ecosystem（デフォルト）、feasibility、implementation、comparison。

```markdown
<research_type>
フェーズリサーチ — 特定のフェーズをうまく実装する方法を調査。
</research_type>

<key_insight>
問題は「どのライブラリを使うべきか？」ではありません。

問題は: 「自分が知らないことで、知らないことは何か？」。

このフェーズについて発見すべきこと:
- 確立されたアーキテクチャパターンは何か？
- 標準スタックを構成するライブラリは何か？
- 一般的にどのような問題に遭遇するか？
- SOTAは何か vs Claudeの学習データが考えるSOTAは何か？
- 自前で実装すべきでないものは何か？
</key_insight>

<objective>
フェーズ {phase_number}: {phase_name} の実装アプローチをリサーチ
モード: ecosystem
</objective>

<files_to_read>
- {requirements_path} (要件)
- {context_path} (discuss-phaseからのフェーズコンテキスト、存在する場合)
- {state_path} (過去のプロジェクト決定事項とブロッカー)
</files_to_read>

<additional_context>
**フェーズの説明:** {phase_description}
</additional_context>

<downstream_consumer>
あなたのRESEARCH.mdは`/gsd:plan-phase`によってロードされ、特定のセクションが使用される:
- `## Standard Stack` → 計画はこれらのライブラリを使用
- `## Architecture Patterns` → タスク構造はこれらに従う
- `## Don't Hand-Roll` → タスクはリストされた問題に対してカスタムソリューションを構築しない
- `## Common Pitfalls` → 検証ステップでこれらを確認
- `## Code Examples` → タスクアクションはこれらのパターンを参照

探索的ではなく処方的に。「Xを検討する」ではなく「Xを使用する」。
</downstream_consumer>

<quality_gate>
完了を宣言する前に検証:
- [ ] すべてのドメインを調査済み（一部だけでなく）
- [ ] 否定的な主張を公式ドキュメントで検証済み
- [ ] 重要な主張に複数のソースを確認
- [ ] 信頼度レベルを正直に割り当て済み
- [ ] セクション名がplan-phaseの期待と一致
</quality_gate>

<output>
書き出し先: .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md
</output>
```

```
Task(
  prompt=filled_prompt,
  subagent_type="gsd-phase-researcher",
  model="{researcher_model}",
  description="Research Phase {phase}"
)
```

## 5. エージェントの返却処理

**`## RESEARCH COMPLETE`:** 概要を表示、提示: フェーズを計画、さらに深掘り、全体をレビュー、完了。

**`## CHECKPOINT REACHED`:** ユーザーに提示、応答を取得、継続を起動。

**`## RESEARCH INCONCLUSIVE`:** 試行した内容を表示、提示: コンテキストを追加、別のモードを試行、手動。

## 6. 継続エージェントの起動

```markdown
<objective>
フェーズ {phase_number}: {phase_name} のリサーチを継続
</objective>

<prior_state>
<files_to_read>
- .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md (既存リサーチ)
</files_to_read>
</prior_state>

<checkpoint_response>
**Type:** {checkpoint_type}
**Response:** {user_response}
</checkpoint_response>
```

```
Task(
  prompt=continuation_prompt,
  subagent_type="gsd-phase-researcher",
  model="{researcher_model}",
  description="Continue research Phase {phase}"
)
```

</process>

<success_criteria>
- [ ] フェーズをロードマップに対して検証済み
- [ ] 既存リサーチを確認済み
- [ ] gsd-phase-researcherをコンテキスト付きで起動済み
- [ ] チェックポイントを正しく処理済み
- [ ] ユーザーが次のステップを把握済み
</success_criteria>

