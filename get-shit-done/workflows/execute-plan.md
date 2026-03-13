<purpose>
フェーズプロンプト（PLAN.md）を実行し、結果サマリー（SUMMARY.md）を作成する。
</purpose>

<required_reading>
操作前にSTATE.mdを読み、プロジェクトコンテキストを読み込むこと。
計画動作の設定についてconfig.jsonを読むこと。

@~/.claude/get-shit-done/references/git-integration.md
</required_reading>

<process>

<step name="init_context" priority="first">
実行コンテキストを読み込む（オーケストレーターのコンテキストを最小化するためパスのみ）：

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init execute-phase "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから抽出：`executor_model`, `commit_docs`, `phase_dir`, `phase_number`, `plans`, `summaries`, `incomplete_plans`, `state_path`, `config_path`。

`.planning/`がない場合：エラー。
</step>

<step name="identify_plan">
```bash
# INIT JSONからのplans/summariesを使用するか、ファイルをリスト
ls .planning/phases/XX-name/*-PLAN.md 2>/dev/null | sort
ls .planning/phases/XX-name/*-SUMMARY.md 2>/dev/null | sort
```

対応するSUMMARYがない最初のPLANを見つける。小数フェーズ対応（`01.1-hotfix/`）：

```bash
PHASE=$(echo "$PLAN_PATH" | grep -oE '[0-9]+(\.[0-9]+)?-[0-9]+')
# 必要に応じてgsd-tools config-getで設定を取得可能
```

<if mode="yolo">
自動承認：`⚡ Execute {phase}-{plan}-PLAN.md [Plan X of Y for Phase Z]` → parse_segments。
</if>

<if mode="interactive" OR="custom with gates.execute_next_plan true">
プラン識別を提示し、確認を待つ。
</if>
</step>

<step name="record_start_time">
```bash
PLAN_START_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
PLAN_START_EPOCH=$(date +%s)
```
</step>

<step name="parse_segments">
```bash
grep -n "type=\"checkpoint" .planning/phases/XX-name/{phase}-{plan}-PLAN.md
```

**チェックポイントタイプ別のルーティング：**

| チェックポイント | パターン | 実行方法 |
|-------------|---------|-----------|
| なし | A（自律） | 単一サブエージェント：完全なプラン + SUMMARY + コミット |
| 検証のみ | B（セグメント化） | チェックポイント間のセグメント。none/human-verifyの後 → サブエージェント。decision/human-actionの後 → メイン |
| 判断 | C（メイン） | 完全にメインコンテキストで実行 |

**パターンA：** init_agent_tracking → Task(subagent_type="gsd-executor", model=executor_model)を生成、プロンプト：[path]でプランを実行、自律、全タスク + SUMMARY + コミット、deviation/authルールに従う、報告：プラン名、タスク、SUMMARYパス、コミットハッシュ → agent_idを追跡 → 待機 → トラッキング更新 → 報告。

**パターンB：** セグメントごとに実行。自律セグメント：割り当てられたタスクのみのサブエージェントを生成（SUMMARY/コミットなし）。チェックポイント：メインコンテキスト。全セグメント後：ファイル/逸脱/決定を集約 → SUMMARY作成 → コミット → セルフチェック：
   - key-files.createdがディスク上に存在するか`[ -f ]`で検証
   - `git log --oneline --all --grep="{phase}-{plan}"`が1つ以上のコミットを返すか確認
   - SUMMARYに`## Self-Check: PASSED`または`## Self-Check: FAILED`を追記

   **既知のClaude Codeバグ（classifyHandoffIfNeeded）：** セグメントエージェントが`classifyHandoffIfNeeded is not defined`で「failed」と報告した場合、これはClaude Codeのランタイムバグ — 実際の失敗ではない。スポットチェックを実行、合格なら成功として扱う。

segment_executionを参照。

**パターンC：** 標準フロー（step name="execute"）を使用してメインで実行。

サブエージェントごとの新鮮なコンテキストがピーク品質を維持する。メインコンテキストは軽量に保たれる。
</step>

<step name="init_agent_tracking">
```bash
if [ ! -f .planning/agent-history.json ]; then
  echo '{"version":"1.0","max_entries":50,"entries":[]}' > .planning/agent-history.json
fi
rm -f .planning/current-agent-id.txt
if [ -f .planning/current-agent-id.txt ]; then
  INTERRUPTED_ID=$(cat .planning/current-agent-id.txt)
  echo "Found interrupted agent: $INTERRUPTED_ID"
fi
```

中断された場合：再開（Taskの`resume`パラメーター）するか、新規開始するかユーザーに確認。

**トラッキングプロトコル：** 生成時：`current-agent-id.txt`にagent_idを書き込み、agent-history.jsonに追加：`{"agent_id":"[id]","task_description":"[desc]","phase":"[phase]","plan":"[plan]","segment":[num|null],"timestamp":"[ISO]","status":"spawned","completion_timestamp":null}`。完了時：status → "completed"、completion_timestampを設定、current-agent-id.txtを削除。プルーニング：entriesがmax_entriesを超えた場合、最も古い"completed"を削除（"spawned"は削除しない）。

パターンA/Bでは生成前に実行。パターンC：スキップ。
</step>

<step name="segment_execution">
パターンBのみ（検証のみチェックポイント）。A/Cではスキップ。

1. セグメントマップをパース：チェックポイントの場所とタイプ
2. セグメントごと：
   - サブエージェントルート：割り当てられたタスクのみのgsd-executorを生成。プロンプト：タスク範囲、プランパス、コンテキスト用に完全なプランを読む、割り当てられたタスクを実行、逸脱を追跡、SUMMARY/コミットなし。エージェントプロトコルで追跡。
   - メインルート：標準フロー（step name="execute"）でタスクを実行
3. 全セグメント後：ファイル/逸脱/決定を集約 → SUMMARY.md作成 → コミット → セルフチェック：
   - key-files.createdがディスク上に存在するか`[ -f ]`で検証
   - `git log --oneline --all --grep="{phase}-{plan}"`が1つ以上のコミットを返すか確認
   - SUMMARYに`## Self-Check: PASSED`または`## Self-Check: FAILED`を追記

   **既知のClaude Codeバグ（classifyHandoffIfNeeded）：** セグメントエージェントが`classifyHandoffIfNeeded is not defined`で「failed」と報告した場合、これはClaude Codeのランタイムバグ — 実際の失敗ではない。スポットチェックを実行、合格なら成功として扱う。




</step>

<step name="load_prompt">
```bash
cat .planning/phases/XX-name/{phase}-{plan}-PLAN.md
```
これが実行指示そのものである。正確に従うこと。プランがCONTEXT.mdを参照する場合：全体を通じてユーザーのビジョンを尊重すること。

**プランに`<interfaces>`ブロックが含まれる場合：** これらは事前抽出された型定義とコントラクト。直接使用すること — 型を発見するためにソースファイルを再度読まないこと。プランナーが必要なものを既に抽出している。
</step>

<step name="previous_phase_check">
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phases list --type summaries --raw
# JSON結果から最後から2番目のサマリーを抽出
```
以前のSUMMARYに未解決の「Issues Encountered」または「Next Phase Readiness」ブロッカーがある場合：AskUserQuestion(header="Previous Issues", options: "Proceed anyway" | "Address first" | "Review previous")。
</step>

<step name="execute">
逸脱は正常 — 以下のルールで処理する。

1. プロンプトから@contextファイルを読む
2. タスクごと：
   - **必須のread_firstゲート：** タスクに`<read_first>`フィールドがある場合、編集を行う前にリストされたすべてのファイルを読まなければならない。これはオプションではない。「既に内容を知っている」からといってファイルをスキップしないこと — 読むこと。read_firstファイルはタスクの真実の基準を確立する。
   - `type="auto"`: `tdd="true"`の場合 → TDD実行。逸脱ルール + 認証ゲート付きで実装。完了基準を検証。コミット（task_commitを参照）。サマリー用にハッシュを追跡。
   - `type="checkpoint:*"`: 停止 → checkpoint_protocol → ユーザーを待つ → 確認後のみ続行。
   - **必須のacceptance_criteriaチェック：** 各タスク完了後、`<acceptance_criteria>`がある場合、次のタスクに進む前にすべての基準を検証すること。grep、ファイル読み取り、CLIコマンドを使用して各基準を確認する。基準が失敗した場合、先に進む前に実装を修正すること。基準をスキップしたり「後で検証する」としてマークしないこと。
3. `<verification>`チェックを実行
4. `<success_criteria>`が満たされていることを確認
5. サマリーに逸脱を文書化
</step>

<authentication_gates>

## 認証ゲート

実行中の認証エラーは失敗ではない — 予想されるインタラクションポイントである。

**指標：** "Not authenticated"、"Unauthorized"、401/403、"Please run {tool} login"、"Set {ENV_VAR}"

**プロトコル：**
1. 認証ゲートを認識する（バグではない）
2. タスク実行を停止
3. 正確な認証ステップ付きの動的checkpoint:human-actionを作成
4. ユーザーの認証を待つ
5. 認証情報が機能するか検証
6. 元のタスクをリトライ
7. 通常通り続行

**例：** `vercel --yes` → "Not authenticated" → ユーザーに`vercel login`を求めるチェックポイント → `vercel whoami`で検証 → デプロイをリトライ → 続行

**サマリー内：** 逸脱としてではなく、「## Authentication Gates」の下の通常フローとして文書化。

</authentication_gates>

<deviation_rules>

## 逸脱ルール

計画外の作業を発見する場合がある。自動的に適用し、サマリーのためにすべて追跡する。

| ルール | トリガー | アクション | 許可 |
|------|---------|--------|------------|
| **1: バグ** | 壊れた動作、エラー、間違ったクエリ、型エラー、セキュリティ脆弱性、レースコンディション、リーク | 修正 → テスト → 検証 → `[Rule 1 - Bug]`として追跡 | 自動 |
| **2: 不足するクリティカル** | 不足している必須項目：エラー処理、バリデーション、認証、CSRF/CORS、レート制限、インデックス、ロギング | 追加 → テスト → 検証 → `[Rule 2 - Missing Critical]`として追跡 | 自動 |
| **3: ブロッキング** | 完了を妨げる：不足する依存関係、間違った型、壊れたインポート、不足するenv/config/ファイル、循環依存 | ブロッカーを修正 → 進行を検証 → `[Rule 3 - Blocking]`として追跡 | 自動 |
| **4: アーキテクチャ** | 構造的変更：新しいDBテーブル、スキーマ変更、新しいサービス、ライブラリ切り替え、API破壊、新しいインフラ | 停止 → 決定を提示（以下）→ `[Rule 4 - Architectural]`として追跡 | ユーザーに確認 |

**ルール4のフォーマット：**
```
⚠️ アーキテクチャ上の決定が必要

現在のタスク: [task name]
発見: [これを引き起こしたもの]
提案する変更: [修正内容]
理由: [根拠]
影響: [これが影響するもの]
代替案: [他のアプローチ]

提案された変更で進めますか？ (yes / different approach / defer)
```

**優先順位：** ルール4（停止）> ルール1-3（自動）> 不明 → ルール4
**エッジケース：** 不足するバリデーション → R2 | nullクラッシュ → R1 | 新しいテーブル → R4 | 新しいカラム → R1/2
**ヒューリスティック：** 正確性/セキュリティ/完了に影響？ → R1-3。かも？ → R4。

</deviation_rules>

<deviation_documentation>

## 逸脱の文書化

サマリーには逸脱セクションを含める必要がある。なし？ → `## Deviations from Plan\n\nNone - plan executed exactly as written.`

逸脱ごと：**[Rule N - Category] Title** — 発見場所：Task X | 問題 | 修正 | 修正されたファイル | 検証 | コミットハッシュ

最後に：**逸脱合計：** N件の自動修正（内訳）。**影響：** 評価。

</deviation_documentation>

<tdd_plan_execution>
## TDD実行

`type: tdd`プランの場合 — RED-GREEN-REFACTOR：

1. **インフラストラクチャ**（最初のTDDプランのみ）：プロジェクトを検出、フレームワークをインストール、設定、空のスイートを検証
2. **RED:** `<behavior>`を読む → 失敗するテスト → 実行（必ず失敗すること）→ コミット：`test({phase}-{plan}): add failing test for [feature]`
3. **GREEN:** `<implementation>`を読む → 最小限のコード → 実行（必ず合格すること）→ コミット：`feat({phase}-{plan}): implement [feature]`
4. **REFACTOR:** クリーンアップ → テストが合格すること → コミット：`refactor({phase}-{plan}): clean up [feature]`

エラー：REDが失敗しない → テスト/既存機能を調査。GREENが合格しない → デバッグ、反復。REFACTORが壊れる → 元に戻す。

構造については`~/.claude/get-shit-done/references/tdd.md`を参照。
</tdd_plan_execution>

<precommit_failure_handling>
## Pre-commit Hook Failure Handling

Your commits may trigger pre-commit hooks. Auto-fix hooks handle themselves transparently — files get fixed and re-staged automatically.

If a commit is BLOCKED by a hook:

1. The `git commit` command fails with hook error output
2. Read the error — it tells you exactly which hook and what failed
3. Fix the issue (type error, lint violation, secret leak, etc.)
4. `git add` the fixed files
5. Retry the commit
6. Do NOT use `--no-verify`

This is normal and expected. Budget 1-2 retry cycles per commit.
</precommit_failure_handling>

<task_commit>
## タスクコミットプロトコル

各タスク（検証合格、完了基準達成）の後、直ちにコミットする。

**1. 確認：** `git status --short`

**2. 個別にステージ**（絶対に`git add .`や`git add -A`を使わない）：
```bash
git add src/api/auth.ts
git add src/types/user.ts
```

**3. コミットタイプ：**

| タイプ | 使用場面 | 例 |
|------|------|---------|
| `feat` | 新機能 | feat(08-02): create user registration endpoint |
| `fix` | バグ修正 | fix(08-02): correct email validation regex |
| `test` | テストのみ（TDD RED） | test(08-02): add failing test for password hashing |
| `refactor` | 動作変更なし（TDD REFACTOR） | refactor(08-02): extract validation to helper |
| `perf` | パフォーマンス | perf(08-02): add database index |
| `docs` | ドキュメント | docs(08-02): add API docs |
| `style` | フォーマット | style(08-02): format auth module |
| `chore` | 設定/依存関係 | chore(08-02): add bcrypt dependency |

**4. フォーマット：** `{type}({phase}-{plan}): {description}`、主要な変更をバレットポイントで。

**5. ハッシュを記録：**
```bash
TASK_COMMIT=$(git rev-parse --short HEAD)
TASK_COMMITS+=("Task ${TASK_NUM}: ${TASK_COMMIT}")
```

</task_commit>

<step name="checkpoint_protocol">
`type="checkpoint:*"`の場合：可能な限りすべてを先に自動化する。チェックポイントは検証/判断のみのため。

表示：`CHECKPOINT: [Type]`ボックス → 進捗 {X}/{Y} → タスク名 → タイプ固有のコンテンツ → `YOUR ACTION: [signal]`

| タイプ | コンテンツ | 再開シグナル |
|------|---------|---------------|
| human-verify (90%) | 構築されたもの + 検証手順（コマンド/URL） | 「approved」または問題を説明 |
| decision (9%) | 必要な決定 + コンテキスト + 長所/短所付きのオプション | 「Select: option-id」 |
| human-action (1%) | 自動化されたもの + 1つの手動ステップ + 検証計画 | 「done」 |

応答後：指定されている場合は検証。合格 → 続行。不合格 → 通知、待機。ユーザーを待つ — 完了をハルシネートしないこと。

詳細は~/.claude/get-shit-done/references/checkpoints.mdを参照。
</step>

<step name="checkpoint_return_for_orchestrator">
Task経由で生成されチェックポイントに到達した場合：構造化された状態を返す（ユーザーと直接やり取りできない）。

**必要な戻り値：** 1) 完了タスクテーブル（ハッシュ + ファイル） 2) 現在のタスク（ブロッキング内容） 3) チェックポイント詳細（ユーザー向けコンテンツ） 4) 待機中（ユーザーから必要なもの）

オーケストレーターがパース → ユーザーに提示 → 完了タスク状態付きで新しい継続を生成。あなたは再開されない。メインコンテキスト内：上記のcheckpoint_protocolを使用。
</step>

<step name="verification_failure_gate">
検証が失敗した場合：

**ノードリペアが有効か確認**（デフォルト: オン）：
```bash
NODE_REPAIR=$(node "./.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.node_repair 2>/dev/null || echo "true")
```

`NODE_REPAIR`が`true`の場合：以下のパラメータで`@./.claude/get-shit-done/workflows/node-repair.md`を呼び出す：
- FAILED_TASK: タスク番号、名前、完了基準
- ERROR: 期待値 vs 実際の結果
- PLAN_CONTEXT: 隣接タスク名 + フェーズゴール
- REPAIR_BUDGET: 設定の`workflow.node_repair_budget`（デフォルト: 2）

ノードリペアはRETRY、DECOMPOSE、PRUNEを自律的に試行する。リペア予算が使い切られた場合のみ（ESCALATE）、このゲートに再到達する。

`NODE_REPAIR`が`false`の場合 または リペアがESCALATEを返した場合：停止。提示：「タスク[X]の検証に失敗：[name]。期待：[criteria]。実際：[result]。リペア試行内容：[試行内容のサマリー]。」オプション：リトライ | スキップ（未完了としてマーク）| 停止（調査）。スキップされた場合 → SUMMARY「Issues Encountered」。
</step>

<step name="record_completion_time">
```bash
PLAN_END_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
PLAN_END_EPOCH=$(date +%s)

DURATION_SEC=$(( PLAN_END_EPOCH - PLAN_START_EPOCH ))
DURATION_MIN=$(( DURATION_SEC / 60 ))

if [[ $DURATION_MIN -ge 60 ]]; then
  HRS=$(( DURATION_MIN / 60 ))
  MIN=$(( DURATION_MIN % 60 ))
  DURATION="${HRS}h ${MIN}m"
else
  DURATION="${DURATION_MIN} min"
fi
```
</step>

<step name="generate_user_setup">
```bash
grep -A 50 "^user_setup:" .planning/phases/XX-name/{phase}-{plan}-PLAN.md | head -50
```

user_setupが存在する場合：テンプレート`~/.claude/get-shit-done/templates/user-setup.md`を使用して`{phase}-USER-SETUP.md`を作成。サービスごと：環境変数テーブル、アカウントセットアップチェックリスト、ダッシュボード設定、ローカル開発メモ、検証コマンド。ステータス「Incomplete」。`USER_SETUP_CREATED=true`を設定。空/見つからない場合：スキップ。
</step>

<step name="create_summary">
`.planning/phases/XX-name/`に`{phase}-{plan}-SUMMARY.md`を作成。`~/.claude/get-shit-done/templates/summary.md`を使用。

**フロントマター：** phase, plan, subsystem, tags | requires/provides/affects | tech-stack.added/patterns | key-files.created/modified | key-decisions | requirements-completed（PLAN.mdフロントマターの`requirements`配列をそのままコピー**しなければならない**） | duration ($DURATION), completed ($PLAN_END_TIME date)。

タイトル：`# Phase [X] Plan [Y]: [Name] Summary`

一行サマリーは実質的に：「jose libraryを使用したJWT authとリフレッシュローテーション」、「Authentication implemented」ではない

含める内容：所要時間、開始/終了時間、タスク数、ファイル数。

次：さらにプランがある → "Ready for {next-plan}" | 最後 → "Phase complete, ready for transition"。
</step>

<step name="update_current_position">
gsd-toolsを使用してSTATE.mdを更新：

```bash
# プランカウンターを進める（最終プランのエッジケースを処理）
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state advance-plan

# ディスク状態からプログレスバーを再計算
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state update-progress

# 実行メトリクスを記録
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state record-metric \
  --phase "${PHASE}" --plan "${PLAN}" --duration "${DURATION}" \
  --tasks "${TASK_COUNT}" --files "${FILE_COUNT}"
```
</step>

<step name="extract_decisions_and_issues">
SUMMARYから：決定を抽出しSTATE.mdに追加：

```bash
# SUMMARYのkey-decisionsから各決定を追加
# シェルセーフなテキスト（`$`、`*`などを正確に保持）のためにファイル入力を優先
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state add-decision \
  --phase "${PHASE}" --summary-file "${DECISION_TEXT_FILE}" --rationale-file "${RATIONALE_FILE}"

# ブロッカーが見つかった場合は追加
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state add-blocker --text-file "${BLOCKER_TEXT_FILE}"
```
</step>

<step name="update_session_continuity">
gsd-toolsを使用してセッション情報を更新：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state record-session \
  --stopped-at "Completed ${PHASE}-${PLAN}-PLAN.md" \
  --resume-file "None"
```

STATE.mdを150行以下に保つ。
</step>

<step name="issues_review_gate">
SUMMARYの「Issues Encountered」が「None」でない場合：yolo → ログして続行。Interactive → 問題を提示し、確認を待つ。
</step>

<step name="update_roadmap">
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap update-plan-progress "${PHASE}"
```
ディスク上のPLAN vs SUMMARYファイルを数える。正しいカウントとステータス（`In Progress`または完了日付付きの`Complete`）でprogressテーブル行を更新。
</step>

<step name="update_requirements">
PLAN.mdフロントマターの`requirements:`フィールドから完了した要件をマーク：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" requirements mark-complete ${REQ_IDS}
```

プランのフロントマターから要件IDを抽出（例：`requirements: [AUTH-01, AUTH-02]`）。requirementsフィールドがない場合、スキップ。
</step>

<step name="git_commit_metadata">
タスクコードはタスクごとに既にコミット済み。プランメタデータをコミット：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs({phase}-{plan}): complete [plan-name] plan" --files .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md .planning/STATE.md .planning/ROADMAP.md .planning/REQUIREMENTS.md
```
</step>

<step name="update_codebase_map">
.planning/codebase/が存在しない場合：スキップ。

```bash
FIRST_TASK=$(git log --oneline --grep="feat({phase}-{plan}):" --grep="fix({phase}-{plan}):" --grep="test({phase}-{plan}):" --reverse | head -1 | cut -d' ' -f1)
git diff --name-only ${FIRST_TASK}^..HEAD 2>/dev/null
```

構造的変更のみ更新：新しいsrc/ディレクトリ → STRUCTURE.md | 依存関係 → STACK.md | ファイルパターン → CONVENTIONS.md | APIクライアント → INTEGRATIONS.md | 設定 → STACK.md | リネーム → パスを更新。コードのみ/バグ修正/コンテンツ変更はスキップ。

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "" --files .planning/codebase/*.md --amend
```
</step>

<step name="offer_next">
`USER_SETUP_CREATED=true`の場合：`⚠️ USER SETUP REQUIRED`をパス + env/configタスク付きで最上部に表示。

```bash
ls -1 .planning/phases/[current-phase-dir]/*-PLAN.md 2>/dev/null | wc -l
ls -1 .planning/phases/[current-phase-dir]/*-SUMMARY.md 2>/dev/null | wc -l
```

| 条件 | ルート | アクション |
|-----------|-------|--------|
| summaries < plans | **A: さらにプラン** | SUMMARYのない次のPLANを見つける。Yolo：自動続行。Interactive：次のプランを表示、`/gsd:execute-phase {phase}` + `/gsd:verify-work`を提案。ここで停止。 |
| summaries = plans、現在 < 最大フェーズ | **B: フェーズ完了** | 完了を表示、`/gsd:plan-phase {Z+1}` + `/gsd:verify-work {Z}` + `/gsd:discuss-phase {Z+1}`を提案 |
| summaries = plans、現在 = 最大フェーズ | **C: マイルストーン完了** | バナーを表示、`/gsd:complete-milestone` + `/gsd:verify-work` + `/gsd:add-phase`を提案 |

すべてのルート：新しいコンテキストのためにまず`/clear`。
</step>

</process>

<success_criteria>

- PLAN.mdのすべてのタスクが完了
- すべての検証に合格
- フロントマターにuser_setupがある場合はUSER-SETUP.mdが生成された
- 実質的な内容のSUMMARY.mdが作成された
- STATE.mdが更新された（位置、決定、問題、セッション）
- ROADMAP.mdが更新された
- コードベースマップが存在する場合：実行の変更でマップが更新された（または重要な変更がない場合はスキップ）
- USER-SETUP.mdが作成された場合：完了出力で目立つように表示された
</success_criteria>
