<purpose>
ロードマップフェーズの実行可能なフェーズプロンプト（PLAN.mdファイル）を、統合されたリサーチと検証とともに作成する。デフォルトフロー：リサーチ（必要な場合）→ 計画 → 検証 → 完了。gsd-phase-researcher、gsd-planner、gsd-plan-checkerエージェントをリビジョンループ（最大3回）でオーケストレーションする。
</purpose>

<required_reading>
開始前に、呼び出し元プロンプトのexecution_contextで参照されているすべてのファイルを読み込むこと。

@~/.claude/get-shit-done/references/ui-brand.md
</required_reading>

<process>

## 1. 初期化

1回の呼び出しで全コンテキストを読み込む（オーケストレーターのコンテキストを最小化するためパスのみ）：

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init plan-phase "$PHASE")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

JSONをパース：`researcher_model`, `planner_model`, `checker_model`, `research_enabled`, `plan_checker_enabled`, `nyquist_validation_enabled`, `commit_docs`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_research`, `has_context`, `has_plans`, `plan_count`, `planning_exists`, `roadmap_exists`, `phase_req_ids`。

**`planning_exists`がfalseの場合：** エラー — まず`/gsd:new-project`を実行すること。
**`phase_found`がfalseの場合：** ROADMAP.mdでフェーズが存在するか検証。有効な場合、initから`phase_slug`と`padded_phase`を使用してディレクトリを作成する：

## 2. 引数のパースと正規化

$ARGUMENTSから抽出：フェーズ番号（整数または`2.1`のような小数）、フラグ（`--research`, `--skip-research`, `--gaps`, `--skip-verify`, `--prd <filepath>`）。

$ARGUMENTSから`--prd <filepath>`を抽出。存在する場合、PRD_FILEにファイルパスを設定。

**フェーズ番号がない場合：** ロードマップから次の未計画フェーズを検出。

**`phase_found`がfalseの場合：** ROADMAP.mdでフェーズが存在するか検証。有効な場合、initから`phase_slug`と`padded_phase`を使用してディレクトリを作成：
```bash
mkdir -p ".planning/phases/${padded_phase}-${phase_slug}"
```
## 3. フェーズの検証

AskUserQuestionを使用する：
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
```
**`has_research`がtrue（initから）かつ`--research`フラグがない場合：** 既存のものを使用し、ステップ6にスキップする。
**`found`がfalseの場合：** 利用可能なフェーズとともにエラー。**`found`がtrueの場合：** JSONから`phase_number`, `phase_name`, `goal`を抽出。

## 3.5. PRDエクスプレスパスの処理

**スキップ条件：** 引数に`--prd`フラグがない場合。

**`--prd <filepath>`が提供された場合：**

1. PRDファイルを読み込む：
```bash
PRD_CONTENT=$(cat "$PRD_FILE" 2>/dev/null)
**`## VERIFICATION PASSED`：** 確認を表示し、ステップ13に進む。
  echo "Error: PRD file not found: $PRD_FILE"
  exit 1
提案：1) 強制続行、2) ガイダンスを提供してリトライ、3) 中止する。
```

2. バナーを表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PRD EXPRESS PATH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

使用中のPRD: {PRD_FILE}
要件からCONTEXT.mdを生成中...
```

3. PRDコンテンツをパースしてCONTEXT.mdを生成する。オーケストレーターは以下を行う：
   - PRDからすべての要件、ユーザーストーリー、受け入れ基準、制約を抽出
   - 各項目をロックされた決定にマッピング（PRD内のすべてはロックされた決定として扱う）
   - PRDがカバーしていない領域を特定し「Claude's Discretion」としてマーク
   - フェーズディレクトリにCONTEXT.mdを作成

4. CONTEXT.mdを書き込む：
```markdown
# Phase [X]: [Name] - Context

**Gathered:** [date]
**Status:** Ready for planning
**Source:** PRD Express Path ({PRD_FILE})

<domain>
## Phase Boundary

[PRDから抽出 — このフェーズが提供するもの]

</domain>

<decisions>
## Implementation Decisions

{PRD内の各要件/ストーリー/基準について：}
### [コンテンツから導出されたカテゴリ]
- [ロックされた決定としての要件]

### Claude's Discretion
[PRDでカバーされていない領域 — 実装の詳細、技術的選択]

</decisions>

<specifics>
## Specific Ideas

[PRDからの特定の参照、例、具体的な要件]

</specifics>

<deferred>
## Deferred Ideas

[PRDで明示的に将来/v2/スコープ外とマークされた項目]
[該当なしの場合: "None — PRD covers phase scope"]

</deferred>

---

*Phase: XX-name*
*Context gathered: [date] via PRD Express Path*
```

5. コミット：
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): generate context from PRD" --files "${phase_dir}/${padded_phase}-CONTEXT.md"
```

6. `context_content`に生成されたCONTEXT.mdの内容を設定し、ステップ5（リサーチの処理）に続行。

**効果：** これはステップ4（CONTEXT.mdの読み込み）を完全にバイパスする（既に作成済みのため）。残りのワークフロー（リサーチ、計画、検証）はPRD由来のコンテキストで通常通り進行する。

## 4. CONTEXT.mdの読み込み

**スキップ条件：** PRDエクスプレスパスが使用された場合（CONTEXT.mdはステップ3.5で既に作成済み）。

initのJSONから`context_path`を確認。

`context_path`がnullでない場合、表示：`Using phase context from: ${context_path}`

**`context_path`がnullの場合（CONTEXT.mdが存在しない）：**

AskUserQuestionを使用：
- header: "No context"
- question: "フェーズ{X}のCONTEXT.mdが見つかりません。プランはリサーチと要件のみを使用します — あなたの設計上の好みは含まれません。続行しますか、それとも先にコンテキストを収集しますか？"
- options:
  - "Continue without context" — リサーチ＋要件のみで計画
  - "Run discuss-phase first" — 計画前に設計上の決定を収集

"Continue without context"の場合：ステップ5に進む。
"Run discuss-phase first"の場合：`/gsd:discuss-phase {X}`を表示してワークフローを終了。

## 5. リサーチの処理

**スキップ条件：** `--gaps`フラグ、`--skip-research`フラグ、または`--research`オーバーライドなしで`research_enabled`がfalse（initから）。

**`has_research`がtrue（initから）かつ`--research`フラグがない場合：** 既存のものを使用し、ステップ6にスキップ。

**RESEARCH.mdが見つからない場合 または `--research`フラグがある場合：**

バナーを表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► RESEARCHING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ リサーチャーを生成中...
```

### gsd-phase-researcherを生成

```bash
PHASE_DESC=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}" | jq -r '.section')
```

リサーチプロンプト：

```markdown
<objective>
フェーズ{phase_number}: {phase_name}の実装方法をリサーチ
回答すべき問い：「このフェーズを適切に計画するために知っておくべきことは何か？」
</objective>

<files_to_read>
- {context_path} (/gsd:discuss-phaseからのユーザー決定)
- {requirements_path} (プロジェクト要件)
- {state_path} (プロジェクトの決定と履歴)
</files_to_read>

<additional_context>
**フェーズ説明:** {phase_description}
**フェーズ要件ID（必ず対処すること）:** {phase_req_ids}

**プロジェクト指示:** ./CLAUDE.mdが存在する場合読み込む — プロジェクト固有のガイドラインに従う
**プロジェクトスキル:** .claude/skills/または.agents/skills/ディレクトリ（いずれかが存在する場合）を確認 — SKILL.mdファイルを読み、リサーチはプロジェクトスキルパターンを考慮すべき
</additional_context>

<output>
Write to: {phase_dir}/{phase_num}-RESEARCH.md
</output>
```

```
Task(
  prompt=research_prompt,
  subagent_type="gsd-phase-researcher",
  model="{researcher_model}",
  description="Research Phase {phase}"
)
```

### リサーチャーの戻り値を処理

- **`## RESEARCH COMPLETE`：** 確認を表示し、ステップ6に続行
- **`## RESEARCH BLOCKED`：** ブロッカーを表示し、提案：1) コンテキストを提供、2) リサーチをスキップ、3) 中止

## 5.5. バリデーション戦略の作成

`nyquist_validation_enabled`がfalseでない限り必須。

```bash
grep -l "## Validation Architecture" "${PHASE_DIR}"/*-RESEARCH.md 2>/dev/null
```

**見つかった場合：**
1. テンプレートを読む：`~/.claude/get-shit-done/templates/VALIDATION.md`
2. `${PHASE_DIR}/${PADDED_PHASE}-VALIDATION.md`に書き込む（Writeツールを使用）
3. フロントマターを記入：`{N}` → フェーズ番号、`{phase-slug}` → スラグ、`{date}` → 現在の日付
4. 検証：
```bash
test -f "${PHASE_DIR}/${PADDED_PHASE}-VALIDATION.md" && echo "VALIDATION_CREATED=true" || echo "VALIDATION_CREATED=false"
```
5. `VALIDATION_CREATED=false`の場合：停止 — ステップ6に進まない
6. `commit_docs`の場合：`commit-docs "docs(phase-${PHASE}): add validation strategy"`

**見つからなかった場合：** 警告して続行 — プランはDimension 8で失敗する可能性がある。

## 6. 既存プランの確認

```bash
ls "${PHASE_DIR}"/*-PLAN.md 2>/dev/null
```

**存在する場合：** 提案：1) プランを追加、2) 既存を表示、3) ゼロから再計画。

## 7. INITからのコンテキストパスの使用

INIT JSONから抽出：

```bash
STATE_PATH=$(printf '%s\n' "$INIT" | jq -r '.state_path // empty')
ROADMAP_PATH=$(printf '%s\n' "$INIT" | jq -r '.roadmap_path // empty')
REQUIREMENTS_PATH=$(printf '%s\n' "$INIT" | jq -r '.requirements_path // empty')
RESEARCH_PATH=$(printf '%s\n' "$INIT" | jq -r '.research_path // empty')
VERIFICATION_PATH=$(printf '%s\n' "$INIT" | jq -r '.verification_path // empty')
UAT_PATH=$(printf '%s\n' "$INIT" | jq -r '.uat_path // empty')
CONTEXT_PATH=$(printf '%s\n' "$INIT" | jq -r '.context_path // empty')
```

## 7.5. Nyquistアーティファクトの検証

`nyquist_validation_enabled`がfalseの場合スキップ。

```bash
VALIDATION_EXISTS=$(ls "${PHASE_DIR}"/*-VALIDATION.md 2>/dev/null | head -1)
```

見つからずNyquistが有効な場合 — ユーザーに質問：
1. 再実行：`/gsd:plan-phase {PHASE} --research`
2. 設定でNyquistを無効化
3. そのまま続行（プランはDimension 8で失敗する）

ユーザーが2または3を選択した場合のみステップ8に進む。

## 8. gsd-plannerエージェントを生成

バナーを表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PLANNING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ プランナーを生成中...
```

プランナープロンプト：

```markdown
<planning_context>
**Phase:** {phase_number}
**Mode:** {standard | gap_closure}

<files_to_read>
- {state_path} (プロジェクト状態)
- {roadmap_path} (ロードマップ)
- {requirements_path} (要件)
- {context_path} (/gsd:discuss-phaseからのユーザー決定)
- {research_path} (技術リサーチ)
- {verification_path} (検証ギャップ - --gapsの場合)
- {uat_path} (UATギャップ - --gapsの場合)
</files_to_read>

**フェーズ要件ID（すべてのIDがプランの`requirements`フィールドに含まれなければならない）:** {phase_req_ids}

**プロジェクト指示:** ./CLAUDE.mdが存在する場合読み込む — プロジェクト固有のガイドラインに従う
**プロジェクトスキル:** .claude/skills/または.agents/skills/ディレクトリ（いずれかが存在する場合）を確認 — SKILL.mdファイルを読み、プランはプロジェクトスキルルールを考慮すべき
</planning_context>

<downstream_consumer>
出力は/gsd:execute-phaseによって消費される。プランに必要なもの：
- フロントマター（wave、depends_on、files_modified、autonomous）
- XML形式のタスク
- 検証基準
- ゴール逆算検証用のmust_haves
</downstream_consumer>

<quality_gate>
- [ ] PLAN.mdファイルがフェーズディレクトリに作成された
- [ ] 各プランに有効なフロントマターがある
- [ ] タスクが具体的で実行可能
- [ ] 依存関係が正しく特定されている
- [ ] 並列実行用のWaveが割り当てられている
- [ ] フェーズ目標から導出されたmust_haves
</quality_gate>
```

```
Task(
  prompt=filled_prompt,
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Plan Phase {phase}"
)
```

## 9. プランナーの戻り値を処理

- **`## PLANNING COMPLETE`：** プラン数を表示。`--skip-verify`または`plan_checker_enabled`がfalse（initから）の場合：ステップ13にスキップ。それ以外：ステップ10。
- **`## CHECKPOINT REACHED`：** ユーザーに提示し、応答を取得し、継続を生成（ステップ12）
- **`## PLANNING INCONCLUSIVE`：** 試行回数を表示し、提案：コンテキストを追加 / リトライ / 手動

## 10. gsd-plan-checkerエージェントを生成

バナーを表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFYING PLANS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ プランチェッカーを生成中...
```

チェッカープロンプト：

```markdown
<verification_context>
**Phase:** {phase_number}
**Phase Goal:** {ROADMAPからの目標}

<files_to_read>
- {PHASE_DIR}/*-PLAN.md (検証するプラン)
- {roadmap_path} (ロードマップ)
- {requirements_path} (要件)
- {context_path} (/gsd:discuss-phaseからのユーザー決定)
- {research_path} (技術リサーチ — バリデーションアーキテクチャを含む)
</files_to_read>

**フェーズ要件ID（すべてカバーされなければならない）:** {phase_req_ids}

**プロジェクト指示:** ./CLAUDE.mdが存在する場合読み込む — プランがプロジェクトガイドラインに準拠しているか検証
**プロジェクトスキル:** .claude/skills/または.agents/skills/ディレクトリ（いずれかが存在する場合）を確認 — プランがプロジェクトスキルルールを考慮しているか検証
</verification_context>

<expected_output>
- ## VERIFICATION PASSED — すべてのチェックに合格
- ## ISSUES FOUND — 構造化された問題リスト
</expected_output>
```

```
Task(
  prompt=checker_prompt,
  subagent_type="gsd-plan-checker",
  model="{checker_model}",
  description="Verify Phase {phase} plans"
)
```

## 11. チェッカーの戻り値を処理

- **`## VERIFICATION PASSED`：** 確認を表示し、ステップ13に進む。
- **`## ISSUES FOUND`：** 問題を表示し、イテレーション数を確認し、ステップ12に進む。

## 12. リビジョンループ（最大3回）

`iteration_count`を追跡（初期プラン＋チェック後に1から開始）。

**iteration_count < 3の場合：**

表示：`修正のためプランナーに返送中...（イテレーション {N}/3）`

リビジョンプロンプト：

```markdown
<revision_context>
**Phase:** {phase_number}
**Mode:** revision

<files_to_read>
- {PHASE_DIR}/*-PLAN.md (既存プラン)
- {context_path} (/gsd:discuss-phaseからのユーザー決定)
</files_to_read>

**チェッカーの問題点:** {structured_issues_from_checker}
</revision_context>

<instructions>
チェッカーの問題に対処するための的確な更新を行う。
問題が根本的でない限り、ゼロから再計画しないこと。
変更内容を返す。
</instructions>
```

```
Task(
  prompt=revision_prompt,
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Revise Phase {phase} plans"
)
```

プランナーが返した後 → チェッカーを再度生成（ステップ10）、iteration_countをインクリメント。

**iteration_count >= 3の場合：**

表示：`最大イテレーション数に達した。{N}個の問題が残っている：` ＋ 問題リスト

提案：1) 強制続行、2) ガイダンスを提供してリトライ、3) 中止

## 13. 最終ステータスの提示

フラグ/設定に応じて`<offer_next>`または`auto_advance`にルーティング。

## 14. 自動進行チェック

自動進行トリガーを確認：

1. $ARGUMENTSから`--auto`フラグをパース
2. **チェーンフラグをインテントと同期** — ユーザーが手動で呼び出した場合（`--auto`なし）、以前の中断された`--auto`チェーンからのエフェメラルチェーンフラグをクリアする。これは`workflow.auto_advance`（ユーザーの永続設定）には触れない：
   ```bash
   if [[ ! "$ARGUMENTS" =~ --auto ]]; then
     node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false 2>/dev/null
   fi
   ```
3. チェーンフラグとユーザー設定の両方を読む：
   ```bash
   AUTO_CHAIN=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
   AUTO_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**`--auto`フラグが存在する場合 または `AUTO_CHAIN`がtrue または `AUTO_CFG`がtrueの場合：**

バナーを表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTO-ADVANCING TO EXECUTE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

プラン準備完了。execute-phaseを起動中...
```

ネストされたTaskセッション（深いエージェントネスティングによるランタイムフリーズを引き起こす）を避けるため、Skillツールを使用してexecute-phaseを起動：
```
Skill(skill="gsd:execute-phase", args="${PHASE} --auto --no-transition")
```

`--no-transition`フラグは、execute-phaseに検証後にさらにチェーンせずステータスを返すよう指示する。これにより自動進行チェーンがフラット化される — 各フェーズが深いTaskエージェントを生成するのではなく、同じネスティングレベルで実行される。

**execute-phaseの戻り値を処理：**
- **PHASE COMPLETE** → 最終サマリーを表示：
  ```
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   GSD ► PHASE ${PHASE} COMPLETE ✓
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  自動進行パイプライン完了。

  次: /gsd:discuss-phase ${NEXT_PHASE} --auto
  ```
- **GAPS FOUND / VERIFICATION FAILED** → 結果を表示し、チェーンを停止：
  ```
  自動進行停止：実行にはレビューが必要。

  上記の出力を確認し、手動で続行すること：
  /gsd:execute-phase ${PHASE}
  ```

**`--auto`も設定も有効でない場合：**
`<offer_next>`にルーティング（既存の動作）。

</process>

<offer_next>
このmarkdownを直接出力する（コードブロックとしてではない）：

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PHASE {X} PLANNED ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {X}: {Name}** — {N}プラン、{M}ウェーブ

| Wave | Plans | 構築内容 |
|------|-------|----------------|
| 1    | 01, 02 | [objectives] |
| 2    | 03     | [objective]  |

リサーチ: {完了 | 既存を使用 | スキップ}
検証: {合格 | オーバーライドで合格 | スキップ}

───────────────────────────────────────────────────────────────

## ▶ Next Up

**Execute Phase {X}** — 全{N}プランを実行

/gsd:execute-phase {X}

<sub>/clear first → 新しいコンテキストウィンドウ</sub>

───────────────────────────────────────────────────────────────

**その他のオプション：**
- cat .planning/phases/{phase-dir}/*-PLAN.md — プランを確認
- /gsd:plan-phase {X} --research — 先にリサーチを再実行

───────────────────────────────────────────────────────────────
</offer_next>

<success_criteria>
- [ ] .planning/ディレクトリが検証された
- [ ] フェーズがロードマップに対して検証された
- [ ] 必要に応じてフェーズディレクトリが作成された
- [ ] CONTEXT.mdが早期（ステップ4）に読み込まれ、すべてのエージェントに渡された
- [ ] リサーチが完了した（--skip-researchまたは--gapsまたは既存でない限り）
- [ ] CONTEXT.md付きでgsd-phase-researcherが生成された
- [ ] 既存プランが確認された
- [ ] CONTEXT.md + RESEARCH.md付きでgsd-plannerが生成された
- [ ] プランが作成された（PLANNING COMPLETEまたはCHECKPOINTを処理）
- [ ] CONTEXT.md付きでgsd-plan-checkerが生成された
- [ ] 検証に合格 または ユーザーオーバーライド または 最大イテレーションでユーザー判断
- [ ] エージェント生成間にユーザーにステータスが表示される
- [ ] ユーザーが次のステップを認識している
</success_criteria>
