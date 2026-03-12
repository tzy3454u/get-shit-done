<purpose>
ウェーブベースの並列実行を使用して、フェーズ内のすべてのプランを実行する。オーケストレーターは軽量に保つ — プラン実行をサブエージェントに委任する。
</purpose>

<core_principle>
オーケストレーターは調整する、実行しない。各サブエージェントは完全なexecute-planコンテキストを読み込む。オーケストレーター：プランを発見 → 依存関係を分析 → ウェーブにグループ化 → エージェントを生成 → チェックポイントを処理 → 結果を収集。
</core_principle>

<required_reading>
操作前にSTATE.mdを読み、プロジェクトコンテキストを読み込むこと。
</required_reading>

<process>

<step name="initialize" priority="first">
1回の呼び出しですべてのコンテキストを読み込む：

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init execute-phase "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

JSONをパース：`executor_model`, `verifier_model`, `commit_docs`, `parallelization`, `branching_strategy`, `branch_name`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `plans`, `incomplete_plans`, `plan_count`, `incomplete_count`, `state_exists`, `roadmap_exists`, `phase_req_ids`。

**`phase_found`がfalseの場合：** エラー — フェーズディレクトリが見つからない。
**`plan_count`が0の場合：** エラー — フェーズにプランが見つからない。
**`state_exists`がfalseだが`.planning/`が存在する場合：** 再構築するか続行するか提案。

`parallelization`がfalseの場合、ウェーブ内のプランは順次実行される。

**チェーンフラグをインテントと同期** — ユーザーが手動で呼び出した場合（`--auto`なし）、以前の中断された`--auto`チェーンからのエフェメラルチェーンフラグをクリアする。これは`workflow.auto_advance`（ユーザーの永続設定）には触れない。設定の読み取り前に実行する必要がある（チェックポイント処理も自動進行フラグを読む）：
```bash
if [[ ! "$ARGUMENTS" =~ --auto ]]; then
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false 2>/dev/null
fi
```
</step>

<step name="handle_branching">
initから`branching_strategy`を確認：

**"none"：** スキップし、現在のブランチで続行。

**"phase"または"milestone"：** initから事前計算された`branch_name`を使用：
```bash
git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
```

以降のすべてのコミットはこのブランチに行われる。マージはユーザーが処理する。
</step>

<step name="validate_phase">
init JSONから：`phase_dir`, `plan_count`, `incomplete_count`。

報告：「{phase_dir}に{plan_count}個のプランが見つかりました（{incomplete_count}個が未完了）」
</step>

<step name="discover_and_group_plans">
1回の呼び出しでウェーブグループ付きのプランインベントリを読み込む：

```bash
PLAN_INDEX=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase-plan-index "${PHASE_NUMBER}")
```

JSONをパース：`phase`, `plans[]`（各々`id`, `wave`, `autonomous`, `objective`, `files_modified`, `task_count`, `has_summary`付き）, `waves`（ウェーブ番号→プランIDのマップ）, `incomplete`, `has_checkpoints`。

**フィルタリング：** `has_summary: true`のプランをスキップ。`--gaps-only`の場合：非gap_closureプランもスキップ。すべてフィルタされた場合：「一致する未完了プランなし」→ 終了。

報告：
```
## 実行計画

**Phase {X}: {Name}** — {total_plans}プラン、{wave_count}ウェーブ

| Wave | Plans | 構築内容 |
|------|-------|----------------|
| 1 | 01-01, 01-02 | {プランの目的から、3-8語} |
| 2 | 01-03 | ... |
```
</step>

<step name="execute_waves">
各ウェーブを順次実行する。ウェーブ内：`PARALLELIZATION=true`なら並列、`false`なら順次。

**各ウェーブについて：**

1. **構築内容を説明する（生成前に）：**

   各プランの`<objective>`を読む。何を構築し、なぜかを抽出。

   ```
   ---
   ## Wave {N}

   **{Plan ID}: {Plan Name}**
   {2-3文：何を構築するか、技術的アプローチ、なぜ重要か}

   {count}個のエージェントを生成中...
   ---
   ```

   - 悪い例：「地形生成プランを実行中」
   - 良い例：「パーリンノイズを使用した手続き的地形ジェネレーター — 高さマップ、バイオームゾーン、衝突メッシュを作成。車両物理が地面とインタラクションする前に必要。」

2. **エグゼキューターエージェントを生成：**

   パスのみを渡す — エグゼキューターは新しい200kコンテキストで自身でファイルを読む。
   これによりオーケストレーターのコンテキストが軽量に保たれる（約10-15%）。

   ```
   Task(
     subagent_type="gsd-executor",
     model="{executor_model}",
     prompt="
       <objective>
       フェーズ{phase_number}-{phase_name}のプラン{plan_number}を実行する。
       各タスクをアトミックにコミット。SUMMARY.mdを作成。STATE.mdとROADMAP.mdを更新。
       </objective>

       <execution_context>
       @~/.claude/get-shit-done/workflows/execute-plan.md
       @~/.claude/get-shit-done/templates/summary.md
       @~/.claude/get-shit-done/references/checkpoints.md
       @~/.claude/get-shit-done/references/tdd.md
       </execution_context>

       <files_to_read>
       実行開始時にReadツールでこれらのファイルを読むこと：
       - {phase_dir}/{plan_file} (プラン)
       - .planning/STATE.md (状態)
       - .planning/config.json (設定、存在する場合)
       - ./CLAUDE.md (プロジェクト指示、存在する場合 — プロジェクト固有のガイドラインとコーディング規約に従う)
       - .claude/skills/ or .agents/skills/ (プロジェクトスキル、いずれかが存在する場合 — スキルをリストし、各SKILL.mdを読み、実装中に関連ルールに従う)
       </files_to_read>

       <success_criteria>
       - [ ] すべてのタスクが実行された
       - [ ] 各タスクが個別にコミットされた
       - [ ] SUMMARY.mdがプランディレクトリに作成された
       - [ ] STATE.mdが位置と決定で更新された
       - [ ] ROADMAP.mdがプラン進捗で更新された（`roadmap update-plan-progress`経由）
       </success_criteria>
     "
   )
   ```

3. **ウェーブ内のすべてのエージェントの完了を待つ。**

4. **完了を報告 — まず主張をスポットチェック：**

   各SUMMARY.mdについて：
   - `key-files.created`の最初の2ファイルがディスク上に存在するか検証
   - `git log --oneline --all --grep="{phase}-{plan}"`が1つ以上のコミットを返すか確認
   - `## Self-Check: FAILED`マーカーを確認

   スポットチェックが1つでも失敗した場合：どのプランが失敗したか報告し、失敗ハンドラーにルーティング — 「プランをリトライしますか？」または「残りのウェーブを続行しますか？」

   合格した場合：
   ```
   ---
   ## Wave {N} Complete

   **{Plan ID}: {Plan Name}**
   {何が構築されたか — SUMMARY.mdから}
   {注目すべき逸脱があれば}

   {さらにウェーブがある場合：これが次のウェーブで何を可能にするか}
   ---
   ```

   - 悪い例：「Wave 2完了。Wave 3に進みます。」
   - 良い例：「地形システム完了 — 3つのバイオームタイプ、高さベースのテクスチャリング、物理衝突メッシュ。車両物理（Wave 3）が地面サーフェスを参照可能に。」

5. **失敗を処理：**

   **既知のClaude Codeバグ（classifyHandoffIfNeeded）：** エージェントが「failed」と報告し、エラーに`classifyHandoffIfNeeded is not defined`が含まれる場合、これはClaude Codeのランタイムバグであり、GSDやエージェントの問題ではない。エラーは完了ハンドラーですべてのツール呼び出しが終了した後に発生する。この場合：ステップ4と同じスポットチェックを実行（SUMMARY.mdが存在、gitコミットが存在、Self-Check: FAILEDなし）。スポットチェックが合格 → **成功**として扱う。スポットチェックが不合格 → 以下の実際の失敗として扱う。

   実際の失敗の場合：どのプランが失敗したか報告 → 「続行しますか？」または「停止しますか？」→ 続行の場合、依存プランも失敗する可能性がある。停止の場合、部分的な完了報告。

6. **ウェーブ間にチェックポイントプランを実行** — `<checkpoint_handling>`を参照。

7. **次のウェーブに進む。**
</step>

<step name="checkpoint_handling">
`autonomous: false`のプランはユーザーインタラクションが必要。

**自動モードのチェックポイント処理：**

自動進行設定を読む（チェーンフラグ＋ユーザー設定）：
```bash
AUTO_CHAIN=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
AUTO_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
```

エグゼキューターがチェックポイントを返し、かつ（`AUTO_CHAIN`が`"true"` または `AUTO_CFG`が`"true"`）の場合：
- **human-verify** → `{user_response}` = `"approved"`で継続エージェントを自動生成。ログ `⚡ Auto-approved checkpoint`。
- **decision** → `{user_response}` = チェックポイント詳細の最初のオプションで継続エージェントを自動生成。ログ `⚡ Auto-selected: [option]`。
- **human-action** → ユーザーに提示（以下の既存の動作）。認証ゲートは自動化できない。

**標準フロー（自動モードでない場合、またはhuman-actionタイプ）：**

1. チェックポイントプラン用にエージェントを生成
2. エージェントがチェックポイントタスクまたは認証ゲートまで実行 → 構造化された状態を返す
3. エージェントの戻り値に含まれるもの：完了タスクテーブル、現在のタスク＋ブロッカー、チェックポイントタイプ/詳細、待機中の内容
4. **ユーザーに提示：**
   ```
   ## Checkpoint: [Type]

   **Plan:** 03-03 Dashboard Layout
   **Progress:** 2/3タスク完了

   [エージェントの戻り値からのチェックポイント詳細]
   [エージェントの戻り値からの待機セクション]
   ```
5. ユーザーが応答：「approved」/「done」| 問題の説明 | 決定の選択
6. **継続エージェントを生成（再開ではない）** continuation-prompt.mdテンプレートを使用：
   - `{completed_tasks_table}`：チェックポイントの戻り値から
   - `{resume_task_number}` + `{resume_task_name}`：現在のタスク
   - `{user_response}`：ユーザーが提供したもの
   - `{resume_instructions}`：チェックポイントタイプに基づく
7. 継続エージェントが以前のコミットを検証し、再開ポイントから続行
8. プランが完了するかユーザーが停止するまで繰り返し

**なぜ再開ではなく新しいエージェントか：** 再開は並列ツール呼び出しで壊れる内部シリアライゼーションに依存する。明示的な状態を持つ新しいエージェントの方が信頼性が高い。

**並列ウェーブでのチェックポイント：** エージェントが一時停止して返す間、他の並列エージェントが完了する可能性がある。チェックポイントを提示し、継続を生成し、次のウェーブの前にすべてを待つ。
</step>

<step name="aggregate_results">
すべてのウェーブの後：

```markdown
## Phase {X}: {Name} 実行完了

**ウェーブ:** {N} | **プラン:** {M}/{total} 完了

| Wave | Plans | Status |
|------|-------|--------|
| 1 | plan-01, plan-02 | ✓ Complete |
| CP | plan-03 | ✓ Verified |
| 2 | plan-04 | ✓ Complete |

### プラン詳細
1. **03-01**: [SUMMARY.mdからの一行要約]
2. **03-02**: [SUMMARY.mdからの一行要約]

### 発生した問題
[SUMMARYからの集約、または "None"]
```
</step>

<step name="close_parent_artifacts">
**小数点/ポリッシュフェーズのみ（X.Yパターン）：** 親のUATとデバッグアーティファクトを解決してフィードバックループを閉じる。

**スキップ条件：** フェーズ番号に小数がない場合（例：`3`, `04`）— `4.1`, `03.1`のようなギャップクロージャーフェーズにのみ適用。

**1. 小数フェーズを検出し親を導出：**
```bash
# phase_numberに小数が含まれるか確認
if [[ "$PHASE_NUMBER" == *.* ]]; then
  PARENT_PHASE="${PHASE_NUMBER%%.*}"
fi
```

**2. 親のUATファイルを見つける：**
```bash
PARENT_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" find-phase "${PARENT_PHASE}" --raw)
# PARENT_INFO JSONからディレクトリを抽出し、そのディレクトリでUATファイルを検索
```

**親のUATが見つからない場合：** このステップをスキップ（ギャップクロージャーはVERIFICATION.mdによってトリガーされた可能性がある）。

**3. UATギャップステータスを更新：**

親のUATファイルの`## Gaps`セクションを読む。`status: failed`の各ギャップエントリについて：
- `status: resolved`に更新

**4. UATフロントマターを更新：**

すべてのギャップが`status: resolved`になった場合：
- フロントマター`status: diagnosed` → `status: resolved`に更新
- フロントマター`updated:`タイムスタンプを更新

**5. 参照されたデバッグセッションを解決：**

`debug_session:`フィールドを持つ各ギャップについて：
- デバッグセッションファイルを読む
- フロントマター`status:` → `resolved`に更新
- フロントマター`updated:`タイムスタンプを更新
- resolvedディレクトリに移動：
```bash
mkdir -p .planning/debug/resolved
mv .planning/debug/{slug}.md .planning/debug/resolved/
```

**6. 更新されたアーティファクトをコミット：**
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-${PARENT_PHASE}): resolve UAT gaps and debug sessions after ${PHASE_NUMBER} gap closure" --files .planning/phases/*${PARENT_PHASE}*/*-UAT.md .planning/debug/resolved/*.md
```
</step>

<step name="verify_phase_goal">
フェーズがタスクを完了しただけでなく、目標を達成したかを検証する。

```
Task(
  prompt="フェーズ{phase_number}の目標達成を検証する。
フェーズディレクトリ: {phase_dir}
フェーズ目標: {ROADMAPからの目標}
フェーズ要件ID: {phase_req_ids}
must_havesを実際のコードベースに対してチェック。
PLANフロントマターの要件IDをREQUIREMENTS.mdと照合 — すべてのIDが説明されなければならない。
VERIFICATION.mdを作成。",
  subagent_type="gsd-verifier",
  model="{verifier_model}"
)
```

ステータスを読む：
```bash
grep "^status:" "$PHASE_DIR"/*-VERIFICATION.md | cut -d: -f2 | tr -d ' '
```

| ステータス | アクション |
|--------|--------|
| `passed` | → update_roadmap |
| `human_needed` | ヒューマンテスト項目を提示し、承認またはフィードバックを取得 |
| `gaps_found` | ギャップサマリーを提示し、`/gsd:plan-phase {phase} --gaps`を提案 |

**human_neededの場合：**
```
## ✓ Phase {X}: {Name} — ヒューマン検証が必要

自動チェックはすべて合格。{N}項目がヒューマンテストを必要としています：

{VERIFICATION.mdのhuman_verificationセクションから}

「approved」→ 続行 | 問題を報告 → ギャップクロージャー
```

**gaps_foundの場合：**
```
## ⚠ Phase {X}: {Name} — ギャップ検出

**スコア:** {N}/{M} must-havesが検証済み
**レポート:** {phase_dir}/{phase_num}-VERIFICATION.md

### 不足しているもの
{VERIFICATION.mdからのギャップサマリー}

---
## ▶ Next Up

`/gsd:plan-phase {X} --gaps`

<sub>`/clear` first → 新しいコンテキストウィンドウ</sub>

Also: `cat {phase_dir}/{phase_num}-VERIFICATION.md` — 完全レポート
Also: `/gsd:verify-work {X}` — まず手動テスト
```

ギャップクロージャーサイクル：`/gsd:plan-phase {X} --gaps`がVERIFICATION.mdを読む → `gap_closure: true`のギャッププランを作成 → ユーザーが`/gsd:execute-phase {X} --gaps-only`を実行 → ベリファイアーが再実行。
</step>

<step name="update_roadmap">
**フェーズを完了としてマークし、すべての追跡ファイルを更新：**

```bash
COMPLETION=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase complete "${PHASE_NUMBER}")
```

CLIが処理する内容：
- フェーズのチェックボックスを`[x]`に完了日付付きでマーク
- Progressテーブルを更新（Status → Complete、日付）
- プランカウントを最終値に更新
- STATE.mdを次のフェーズに進める
- REQUIREMENTS.mdのトレーサビリティを更新

結果から抽出：`next_phase`, `next_phase_name`, `is_last_phase`。

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-{X}): complete phase execution" --files .planning/ROADMAP.md .planning/STATE.md .planning/REQUIREMENTS.md {phase_dir}/*-VERIFICATION.md
```
</step>

<step name="offer_next">

**例外：** `gaps_found`の場合、`verify_phase_goal`ステップが既にギャップクロージャーパス（`/gsd:plan-phase {X} --gaps`）を提示済み。追加のルーティングは不要 — 自動進行をスキップ。

**トランジションなしチェック（自動進行チェーンから生成された場合）：**

$ARGUMENTSから`--no-transition`フラグをパース。

**`--no-transition`フラグが存在する場合：**

execute-phaseはplan-phaseの自動進行によって生成された。transition.mdを実行しない。
検証が合格しロードマップが更新された後、親に完了ステータスを返す：

```
## PHASE COMPLETE

Phase: ${PHASE_NUMBER} - ${PHASE_NAME}
Plans: ${completed_count}/${total_count}
Verification: {Passed | Gaps Found}

[aggregate_resultsの出力を含める]
```

停止。自動進行またはトランジションに進まない。

**`--no-transition`フラグが存在しない場合：**

**自動進行検出：**

1. $ARGUMENTSから`--auto`フラグをパース
2. チェーンフラグとユーザー設定の両方を読む（チェーンフラグはinitステップで既に同期済み）：
   ```bash
   AUTO_CHAIN=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
   AUTO_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**`--auto`フラグが存在する場合 または `AUTO_CHAIN`がtrue または `AUTO_CFG`がtrue（かつ検証がギャップなしで合格）の場合：**

```
╔══════════════════════════════════════════╗
║  AUTO-ADVANCING → TRANSITION             ║
║  Phase {X} verified, continuing chain    ║
╚══════════════════════════════════════════╝
```

トランジションワークフローをインラインで実行（Taskを使用しない — オーケストレーターのコンテキストは約10-15%で、トランジションには既にコンテキスト内のフェーズ完了データが必要）：

`~/.claude/get-shit-done/workflows/transition.md`を読み従う。`--auto`フラグを渡して次のフェーズ呼び出しに伝播させる。

**`--auto`も`AUTO_CFG`もtrueでない場合：**

ワークフローは終了する。ユーザーが`/gsd:progress`を実行するか、トランジションワークフローを手動で呼び出す。
</step>

</process>

<context_efficiency>
オーケストレーター：約10-15%コンテキスト。サブエージェント：各200k新規。ポーリングなし（Taskがブロック）。コンテキスト漏洩なし。
</context_efficiency>

<failure_handling>
- **classifyHandoffIfNeededの偽陽性失敗：** エージェントが「failed」と報告するがエラーが`classifyHandoffIfNeeded is not defined` → Claude Codeバグ、GSDではない。スポットチェック（SUMMARY存在、コミット存在）→ 合格なら成功として扱う
- **エージェントがプラン途中で失敗：** SUMMARY.mdが見つからない → 報告、ユーザーに対応方法を確認
- **依存チェーンが切断：** Wave 1が失敗 → Wave 2の依存先も失敗の可能性 → ユーザーが試行またはスキップを選択
- **ウェーブ内の全エージェントが失敗：** システム的な問題 → 停止、調査のために報告
- **チェックポイントが解決不能：** 「このプランをスキップしますか？」または「フェーズ実行を中止しますか？」→ STATE.mdに部分的な進捗を記録
</failure_handling>

<resumption>
`/gsd:execute-phase {phase}`を再実行 → discover_plansが完了したSUMMARYを見つける → スキップ → 最初の未完了プランから再開 → ウェーブ実行を続行。

STATE.mdが追跡：最後に完了したプラン、現在のウェーブ、保留中のチェックポイント。
</resumption>
</output>