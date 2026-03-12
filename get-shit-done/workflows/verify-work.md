<purpose>
永続的な状態を持つ対話型テストを通じて、構築された機能を検証する。テストの進捗を追跡し、/clearに耐え、ギャップを/gsd:plan-phase --gapsにフィードするUAT.mdを作成する。

ユーザーがテストし、Claudeが記録する。一度に1つのテスト。プレーンテキストで応答。
</purpose>

<philosophy>
**期待される動作を示し、現実が一致するか確認する。**

Claudeは何が起こるべきかを提示する。ユーザーは確認するか、異なる点を説明する。
- "yes" / "y" / "next" / 空 → 合格
- それ以外 → 問題として記録、重要度を推定

Pass/Failボタンなし。重要度の質問なし。ただ: "こうなるはずです。そうなりましたか？"
</philosophy>

<template>
@~/.claude/get-shit-done/templates/UAT.md
</template>

<process>

<step name="initialize" priority="first">
$ARGUMENTSにフェーズ番号が含まれている場合、コンテキストを読み込む:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init verify-work "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

JSONをパース: `planner_model`, `checker_model`, `commit_docs`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `has_verification`。
</step>

<step name="check_active_session">
**まず: アクティブなUATセッションを確認**

```bash
find .planning/phases -name "*-UAT.md" -type f 2>/dev/null | head -5
```

**アクティブなセッションが存在し、$ARGUMENTSが未指定の場合:**

各ファイルのフロントマター（ステータス、フェーズ）とCurrent Testセクションを読み取る。

インラインで表示:

```
## アクティブなUATセッション

| # | フェーズ | ステータス | 現在のテスト | 進捗 |
|---|-------|--------|--------------|----------|
| 1 | 04-comments | testing | 3. Reply to Comment | 2/6 |
| 2 | 05-auth | testing | 1. Login Form | 0/4 |

番号を入力して再開するか、フェーズ番号を入力して新規開始してください。
```

ユーザーの応答を待つ。

- ユーザーが番号（1, 2）で応答 → そのファイルを読み込み、`resume_from_file`へ
- ユーザーがフェーズ番号で応答 → 新規セッションとして扱い、`create_uat_file`へ

**アクティブなセッションが存在し、$ARGUMENTSが指定されている場合:**

そのフェーズのセッションが存在するか確認。存在する場合、再開か再スタートかを提示。
存在しない場合、`create_uat_file`に続行。

**アクティブなセッションがなく、$ARGUMENTSが未指定の場合:**

```
アクティブなUATセッションはありません。

フェーズ番号を指定してテストを開始してください（例: /gsd:verify-work 4）
```

**アクティブなセッションがなく、$ARGUMENTSが指定されている場合:**

`create_uat_file`に続行。
</step>

<step name="find_summaries">
**テスト対象の検出:**

initからの`phase_dir`を使用（またはまだ実行されていない場合はinitを実行）。

```bash
ls "$phase_dir"/*-SUMMARY.md 2>/dev/null
```

各SUMMARY.mdを読み取り、テスト可能な成果物を抽出する。
</step>

<step name="extract_tests">
**SUMMARY.mdからテスト可能な成果物を抽出:**

以下をパース:
1. **達成事項** - 追加された機能/機能性
2. **ユーザー向けの変更** - UI、ワークフロー、インタラクション

実装の詳細ではなく、ユーザーが観察可能な成果に焦点を当てる。

各成果物に対してテストを作成:
- name: 簡潔なテスト名
- expected: ユーザーが見る/体験するべきこと（具体的、観察可能）

例:
- 達成事項: "無限ネストのコメントスレッドを追加"
  → テスト: "コメントへの返信"
  → Expected: "返信をクリックするとコメントの下にインラインコンポーザーが開く。送信すると返信が親の下にビジュアルインデントで表示される。"

内部的/観察不可能な項目（リファクタリング、型変更など）はスキップ。

**コールドスタートスモークテストの注入:**

SUMMARYからテストを抽出した後、SUMMARYファイルで変更/作成されたファイルパスをスキャンする。いずれかのパスが以下のパターンに一致する場合:

`server.ts`, `server.js`, `app.ts`, `app.js`, `index.ts`, `index.js`, `main.ts`, `main.js`, `database/*`, `db/*`, `seed/*`, `seeds/*`, `migrations/*`, `startup*`, `docker-compose*`, `Dockerfile*`

テストリストの**先頭に**このテストを追加:

- name: "コールドスタートスモークテスト"
- expected: "実行中のサーバー/サービスをすべて停止する。一時的な状態（一時DB、キャッシュ、ロックファイル）をクリアする。アプリケーションをゼロから起動する。サーバーがエラーなく起動し、シード/マイグレーションが完了し、プライマリクエリ（ヘルスチェック、ホームページ読み込み、または基本的なAPI呼び出し）がライブデータを返す。"

これはフレッシュスタート時にのみ発生するバグをキャッチする — 起動シーケンスの競合状態、サイレントなシード失敗、環境セットアップの欠落 — ウォーム状態では通過するが本番環境で壊れるもの。
</step>

<step name="create_uat_file">
**すべてのテストを含むUATファイルの作成:**

```bash
mkdir -p "$PHASE_DIR"
```

抽出された成果物からテストリストを構築。

ファイルを作成:

```markdown
---
status: testing
phase: XX-name
source: [SUMMARY.mdファイルのリスト]
started: [ISOタイムスタンプ]
updated: [ISOタイムスタンプ]
---

## Current Test
<!-- 各テストで上書き - 現在位置を表示 -->

number: 1
name: [最初のテスト名]
expected: |
  [ユーザーが観察すべきこと]
awaiting: user response

## Tests

### 1. [テスト名]
expected: [観察可能な動作]
result: [pending]

### 2. [テスト名]
expected: [観察可能な動作]
result: [pending]

...

## Summary

total: [N]
passed: 0
issues: 0
pending: [N]
skipped: 0

## Gaps

[まだなし]
```

`.planning/phases/XX-name/{phase_num}-UAT.md`に書き込む

`present_test`に進む。
</step>

<step name="present_test">
**現在のテストをユーザーに提示:**

UATファイルのCurrent Testセクションを読み取る。

チェックポイントボックス形式で表示:

```
╔══════════════════════════════════════════════════════════════╗
║  CHECKPOINT: 検証が必要です                                    ║
╚══════════════════════════════════════════════════════════════╝

**テスト {number}: {name}**

{expected}

──────────────────────────────────────────────────────────────
→ "pass"と入力するか、問題点を記述してください
──────────────────────────────────────────────────────────────
```

ユーザーの応答を待つ（プレーンテキスト、AskUserQuestionは使用しない）。
</step>

<step name="process_response">
**ユーザーの応答を処理しファイルを更新:**

**応答が合格を示す場合:**
- 空の応答、"yes", "y", "ok", "pass", "next", "approved", "✓"

Testsセクションを更新:
```
### {N}. {name}
expected: {expected}
result: pass
```

**応答がスキップを示す場合:**
- "skip", "can't test", "n/a"

Testsセクションを更新:
```
### {N}. {name}
expected: {expected}
result: skipped
reason: [ユーザーの理由（提供された場合）]
```

**応答がそれ以外の場合:**
- 問題の説明として扱う

説明から重要度を推定:
- 含む: crash, error, exception, fails, broken, unusable → blocker
- 含む: doesn't work, wrong, missing, can't → major
- 含む: slow, weird, off, minor, small → minor
- 含む: color, font, spacing, alignment, visual → cosmetic
- 不明な場合のデフォルト: major

Testsセクションを更新:
```
### {N}. {name}
expected: {expected}
result: issue
reported: "{ユーザーの応答をそのまま}"
severity: {推定}
```

Gapsセクションに追記（plan-phase --gaps用の構造化YAML）:
```yaml
- truth: "{テストからの期待される動作}"
  status: failed
  reason: "User reported: {ユーザーの応答をそのまま}"
  severity: {推定}
  test: {N}
  artifacts: []  # 診断で記入
  missing: []    # 診断で記入
```

**いずれの応答後も:**

Summaryのカウントを更新。
フロントマターのupdatedタイムスタンプを更新。

テストが残っている場合 → Current Testを更新、`present_test`へ
テストがない場合 → `complete_session`へ
</step>

<step name="resume_from_file">
**UATファイルからテストを再開:**

UATファイル全体を読み取る。

`result: [pending]`の最初のテストを見つける。

通知:
```
再開中: フェーズ{phase} UAT
進捗: {passed + issues + skipped}/{total}
これまでに見つかった問題: {issues count}

テスト{N}から続行...
```

Current Testセクションを保留中のテストで更新。
`present_test`に進む。
</step>

<step name="complete_session">
**テスト完了とコミット:**

フロントマターを更新:
- status: complete
- updated: [現在]

Current Testセクションをクリア:
```
## Current Test

[テスト完了]
```

UATファイルをコミット:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "test({phase_num}): complete UAT - {passed} passed, {issues} issues" --files ".planning/phases/XX-name/{phase_num}-UAT.md"
```

サマリーを提示:
```
## UAT完了: フェーズ{phase}

| 結果 | 件数 |
|--------|-------|
| 合格 | {N}   |
| 問題 | {N}   |
| スキップ| {N}   |

[issues > 0の場合:]
### 検出された問題

[Issuesセクションからのリスト]
```

**issues > 0の場合:** `diagnose_issues`に進む

**issues == 0の場合:**
```
すべてのテストが合格しました。続行する準備ができています。

- `/gsd:plan-phase {next}` — 次のフェーズを計画
- `/gsd:execute-phase {next}` — 次のフェーズを実行
```
</step>

<step name="diagnose_issues">
**修正計画の前に根本原因を診断:**

```
---

{N}件の問題が見つかりました。根本原因を診断中...

各問題を調査するために並列デバッグエージェントを生成します。
```

- diagnose-issuesワークフローを読み込む
- @~/.claude/get-shit-done/workflows/diagnose-issues.mdに従う
- 各問題に対して並列デバッグエージェントを生成
- 根本原因を収集
- UAT.mdを根本原因で更新
- `plan_gap_closure`に進む

診断は自動的に実行 - ユーザーへの確認なし。並列エージェントが同時に調査するため、オーバーヘッドは最小限で修正の精度が向上。
</step>

<step name="plan_gap_closure">
**診断されたギャップからの修正を自動計画:**

表示:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PLANNING FIXES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ ギャップクロージャーのためにプランナーを生成中...
```

gsd-plannerを--gapsモードで生成:

```
Task(
  prompt="""
<planning_context>

**Phase:** {phase_number}
**Mode:** gap_closure

<files_to_read>
- {phase_dir}/{phase_num}-UAT.md (診断付きUAT)
- .planning/STATE.md (プロジェクト状態)
- .planning/ROADMAP.md (ロードマップ)
</files_to_read>

</planning_context>

<downstream_consumer>
出力は/gsd:execute-phaseで使用されます。
計画は実行可能なプロンプトでなければなりません。
</downstream_consumer>
""",
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Plan gap fixes for Phase {phase}"
)
```

戻り値:
- **PLANNING COMPLETE:** `verify_gap_plans`に進む
- **PLANNING INCONCLUSIVE:** 報告し、手動介入を提示
</step>

<step name="verify_gap_plans">
**チェッカーで修正計画を検証:**

表示:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFYING FIX PLANS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ プランチェッカーを生成中...
```

初期化: `iteration_count = 1`

gsd-plan-checkerを生成:

```
Task(
  prompt="""
<verification_context>

**Phase:** {phase_number}
**Phase Goal:** UATから診断されたギャップのクロージャー

<files_to_read>
- {phase_dir}/*-PLAN.md (検証する計画)
</files_to_read>

</verification_context>

<expected_output>
以下のいずれかを返す:
- ## VERIFICATION PASSED — すべてのチェックが合格
- ## ISSUES FOUND — 構造化された問題リスト
</expected_output>
""",
  subagent_type="gsd-plan-checker",
  model="{checker_model}",
  description="Verify Phase {phase} fix plans"
)
```

戻り値:
- **VERIFICATION PASSED:** `present_ready`に進む
- **ISSUES FOUND:** `revision_loop`に進む
</step>

<step name="revision_loop">
**計画が合格するまでプランナー ↔ チェッカーを反復（最大3回）:**

**iteration_count < 3の場合:**

表示: `修正のためプランナーに差し戻し中... (反復 {N}/3)`

修正コンテキスト付きでgsd-plannerを生成:

```
Task(
  prompt="""
<revision_context>

**Phase:** {phase_number}
**Mode:** revision

<files_to_read>
- {phase_dir}/*-PLAN.md (既存の計画)
</files_to_read>

**チェッカーの問題:**
{structured_issues_from_checker}

</revision_context>

<instructions>
既存のPLAN.mdファイルを読み取る。チェッカーの問題に対処するためのターゲットを絞った更新を行う。
問題が根本的でない限り、ゼロから再計画しないこと。
</instructions>
""",
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Revise Phase {phase} plans"
)
```

プランナーが返した後 → チェッカーを再度生成（verify_gap_plansのロジック）
iteration_countをインクリメント

**iteration_count >= 3の場合:**

表示: `最大反復回数に達しました。{N}件の問題が残っています。`

オプションを提示:
1. 強制続行（問題があっても実行）
2. ガイダンスを提供（ユーザーが方向性を示し、リトライ）
3. 中止（終了、ユーザーが手動で/gsd:plan-phaseを実行）

ユーザーの応答を待つ。
</step>

<step name="present_ready">
**完了と次のステップを提示:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► FIXES READY ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**フェーズ {X}: {Name}** — {N}件のギャップが診断され、{M}件の修正計画が作成されました

| ギャップ | 根本原因 | 修正計画 |
|-----|------------|----------|
| {truth 1} | {root_cause} | {phase}-04 |
| {truth 2} | {root_cause} | {phase}-04 |

計画は検証済みで実行の準備ができています。

───────────────────────────────────────────────────────────────

## ▶ Next Up

**修正を実行** — 修正計画を実行する

`/clear` then `/gsd:execute-phase {phase} --gaps-only`

───────────────────────────────────────────────────────────────
```
</step>

</process>

<update_rules>
**効率化のためのバッチ書き込み:**

結果をメモリに保持。ファイルへの書き込みは以下の場合のみ:
1. **問題が見つかった場合** — 問題を即座に保存
2. **セッション完了時** — コミット前の最終書き込み
3. **チェックポイント** — 5つのテストが合格するごとに（安全ネット）

| セクション | ルール | 書き込みタイミング |
|---------|------|--------------|
| Frontmatter.status | 上書き | 開始時、完了時 |
| Frontmatter.updated | 上書き | ファイル書き込み時 |
| Current Test | 上書き | ファイル書き込み時 |
| Tests.{N}.result | 上書き | ファイル書き込み時 |
| Summary | 上書き | ファイル書き込み時 |
| Gaps | 追記 | 問題が見つかった時 |

コンテキストリセット時: ファイルは最後のチェックポイントを表示。そこから再開。
</update_rules>

<severity_inference>
**ユーザーの自然言語から重要度を推定:**

| ユーザーの発言 | 推定 |
|-----------|-------|
| "crashes", "error", "exception", "fails completely" | blocker |
| "doesn't work", "nothing happens", "wrong behavior" | major |
| "works but...", "slow", "weird", "minor issue" | minor |
| "color", "spacing", "alignment", "looks off" | cosmetic |

不明な場合は**major**をデフォルトとする。必要に応じてユーザーが修正できる。

**「これはどの程度深刻ですか？」とは決して聞かない** — 推定して先に進む。
</severity_inference>

<success_criteria>
- [ ] SUMMARY.mdのすべてのテストを含むUATファイルを作成
- [ ] 期待される動作とともにテストを1つずつ提示
- [ ] ユーザーの応答を合格/問題/スキップとして処理
- [ ] 説明から重要度を推定（決して聞かない）
- [ ] バッチ書き込み: 問題時、5回の合格ごと、または完了時
- [ ] 完了時にコミット
- [ ] 問題がある場合: 並列デバッグエージェントが根本原因を診断
- [ ] 問題がある場合: gsd-plannerが修正計画を作成（gap_closureモード）
- [ ] 問題がある場合: gsd-plan-checkerが修正計画を検証
- [ ] 問題がある場合: 計画が合格するまで修正ループ（最大3回の反復）
- [ ] 完了時に`/gsd:execute-phase --gaps-only`の準備完了
</success_criteria>
</output>
