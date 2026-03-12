<purpose>
GSDの保証（アトミックコミット、STATE.md追跡）を持つ小規模なアドホックタスクを実行する。クイックモードはgsd-planner（クイックモード）+ gsd-executor(s)を生成し、タスクを`.planning/quick/`で追跡し、STATE.mdの"Quick Tasks Completed"テーブルを更新する。

`--discuss`フラグ: 計画前の軽量ディスカッションフェーズ。前提を明らかにし、曖昧な点を明確にし、決定をCONTEXT.mdに記録してプランナーがそれを確定として扱うようにする。

`--full`フラグ: フルマイルストーンの儀式なしに品質保証のための計画チェック（最大2回の反復）と実行後の検証を有効にする。

フラグは組み合わせ可能: `--discuss --full`でディスカッション + 計画チェック + 検証が行われる。
</purpose>

<required_reading>
開始前に、呼び出しプロンプトのexecution_contextで参照されるすべてのファイルを読み取ること。
</required_reading>

<process>
**Step 1: 引数の解析とタスク説明の取得**

`$ARGUMENTS`を以下についてパース:
- `--full`フラグ → `$FULL_MODE`として保存（true/false）
- `--discuss`フラグ → `$DISCUSS_MODE`として保存（true/false）
- 残りのテキスト → 空でなければ`$DESCRIPTION`として使用

パース後に`$DESCRIPTION`が空の場合、ユーザーに対話的に確認:

```
AskUserQuestion(
  header: "クイックタスク",
  question: "何をしたいですか？",
  followUp: null
)
```

応答を`$DESCRIPTION`として保存。

それでも空の場合、再確認: "タスクの説明を入力してください。"

有効なフラグに基づいてバナーを表示:

`$DISCUSS_MODE`かつ`$FULL_MODE`の場合:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► QUICK TASK (DISCUSS + FULL)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ ディスカッション + 計画チェック + 検証が有効
```

`$DISCUSS_MODE`のみの場合:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► QUICK TASK (DISCUSS)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ ディスカッションフェーズが有効 — 計画前に曖昧な点を明確化
```

`$FULL_MODE`のみの場合:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► QUICK TASK (FULL MODE)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ 計画チェック + 検証が有効
```

---

**Step 2: 初期化**

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init quick "$DESCRIPTION")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

JSONをパース: `planner_model`, `executor_model`, `checker_model`, `verifier_model`, `commit_docs`, `next_num`, `slug`, `date`, `timestamp`, `quick_dir`, `task_dir`, `roadmap_exists`, `planning_exists`。

**`roadmap_exists`がfalseの場合:** エラー — クイックモードにはROADMAP.mdを持つアクティブなプロジェクトが必要です。先に`/gsd:new-project`を実行してください。

クイックタスクはフェーズ途中でも実行可能 - バリデーションはROADMAP.mdの存在のみを確認し、フェーズの状態は確認しない。

---

**Step 3: タスクディレクトリの作成**

```bash
mkdir -p "${task_dir}"
```

---

**Step 4: クイックタスクディレクトリの作成**

このクイックタスクのディレクトリを作成:

```bash
QUICK_DIR=".planning/quick/${next_num}-${slug}"
mkdir -p "$QUICK_DIR"
```

ユーザーに報告:
```
クイックタスク ${next_num} を作成中: ${DESCRIPTION}
ディレクトリ: ${QUICK_DIR}
```

オーケストレーションで使用するために`$QUICK_DIR`を保存。

---

**Step 4.5: ディスカッションフェーズ（`$DISCUSS_MODE`の場合のみ）**

`$DISCUSS_MODE`でない場合はこのステップを完全にスキップ。

バナーを表示:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DISCUSSING QUICK TASK
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ 曖昧な点を明確化中: ${DESCRIPTION}
```

**4.5a. 曖昧な点の特定**

`$DESCRIPTION`を分析して、2-4の曖昧な点を特定 — 結果を変える可能性がある実装上の決定でユーザーの意見が必要なもの。

ドメイン対応ヒューリスティックを使用して、フェーズ固有の（汎用的でない）曖昧な点を生成:
- ユーザーが**見る**もの → レイアウト、密度、インタラクション、状態
- ユーザーが**呼び出す**もの → レスポンス、エラー、認証、バージョニング
- ユーザーが**実行する**もの → 出力形式、フラグ、モード、エラーハンドリング
- ユーザーが**読む**もの → 構造、トーン、深さ、フロー
- **整理される**もの → 基準、グループ化、命名、例外

各曖昧な点は具体的な決定ポイントであるべきで、漠然としたカテゴリではない。例: "UX"ではなく"読み込み時の動作"。

**4.5b. 曖昧な点の提示**

```
AskUserQuestion(
  header: "曖昧な点",
  question: "計画前にどの領域を明確にする必要がありますか？",
  options: [
    { label: "${area_1}", description: "${why_it_matters_1}" },
    { label: "${area_2}", description: "${why_it_matters_2}" },
    { label: "${area_3}", description: "${why_it_matters_3}" },
    { label: "すべて明確", description: "ディスカッションをスキップ — 必要なものはわかっている" }
  ],
  multiSelect: true
)
```

ユーザーが"すべて明確"を選択した場合 → Step 5にスキップ（CONTEXT.mdは書き込まない）。

**4.5c. 選択された領域のディスカッション**

選択された各領域について、AskUserQuestionで1-2の焦点を絞った質問をする:

```
AskUserQuestion(
  header: "${area_name}",
  question: "${この領域に関する具体的な質問}",
  options: [
    { label: "${concrete_choice_1}", description: "${what_this_means}" },
    { label: "${concrete_choice_2}", description: "${what_this_means}" },
    { label: "${concrete_choice_3}", description: "${what_this_means}" },
    { label: "おまかせ", description: "Claudeの裁量" }
  ],
  multiSelect: false
)
```

ルール:
- オプションは具体的な選択肢でなければならず、抽象的なカテゴリではない
- 明確な意見がある場合は推奨される選択肢をハイライト
- ユーザーが自由記述テキストで"その他"を選択した場合、プレーンテキストのフォローアップに切り替える（questioning.mdの自由記述ルールに従う）
- ユーザーが"おまかせ"を選択した場合、CONTEXT.mdにClaudeの裁量として記録
- 各領域で最大2つの質問 — これは軽量であり、深掘りではない

すべての決定を`$DECISIONS`に収集。

**4.5d. CONTEXT.mdの書き込み**

標準的なコンテキストテンプレート構造を使用して`${QUICK_DIR}/${next_num}-CONTEXT.md`を書き込む:

```markdown
# Quick Task ${next_num}: ${DESCRIPTION} - Context

**Gathered:** ${date}
**Status:** Ready for planning

<domain>
## Task Boundary

${DESCRIPTION}

</domain>

<decisions>
## Implementation Decisions

### ${area_1_name}
- ${decision_from_discussion}

### ${area_2_name}
- ${decision_from_discussion}

### Claude's Discretion
${ユーザーがおまかせと言った領域、またはディスカッションされなかった領域}

</decisions>

<specifics>
## Specific Ideas

${ディスカッションからの具体的な参照や例}

[なし: "特定の要件なし — 標準的なアプローチで可"]

</specifics>
```

注: クイックタスクのCONTEXT.mdは`<code_context>`と`<deferred>`セクションを省略（コードベーススカウティングなし、フェーズスコープへの延期なし）。簡潔に保つ。

報告: `コンテキストを記録: ${QUICK_DIR}/${next_num}-CONTEXT.md`

---

**Step 5: プランナーの生成（クイックモード）**

**`$FULL_MODE`の場合:** より厳格な制約で`quick-full`モードを使用。

**`$FULL_MODE`でない場合:** 標準の`quick`モードを使用。

```
Task(
  prompt="
<planning_context>

**Mode:** ${FULL_MODE ? 'quick-full' : 'quick'}
**Directory:** ${QUICK_DIR}
**Description:** ${DESCRIPTION}

<files_to_read>
- .planning/STATE.md (プロジェクト状態)
- ./CLAUDE.md (存在する場合 — プロジェクト固有のガイドラインに従う)
${DISCUSS_MODE ? '- ' + QUICK_DIR + '/' + next_num + '-CONTEXT.md (ユーザーの決定 — 確定済み、再検討しない)' : ''}
</files_to_read>

**Project skills:** .claude/skills/または.agents/skills/ディレクトリを確認（存在する場合） — SKILL.mdファイルを読み、計画はプロジェクトスキルのルールを考慮すべき

</planning_context>

<constraints>
- 1-3の焦点を絞ったタスクで単一の計画を作成
- クイックタスクはアトミックで自己完結的であるべき
- 調査フェーズなし
${FULL_MODE ? '- コンテキスト使用量の約40%を目標（検証用に構造化）' : '- コンテキスト使用量の約30%を目標（シンプル、焦点を絞る）'}
${FULL_MODE ? '- 計画のフロントマターに`must_haves`を生成すること必須（truths, artifacts, key_links）' : ''}
${FULL_MODE ? '- 各タスクに`files`, `action`, `verify`, `done`フィールドが必須' : ''}
</constraints>

<output>
計画の書き込み先: ${QUICK_DIR}/${next_num}-PLAN.md
返却: ## PLANNING COMPLETEと計画パス
</output>
",
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Quick plan: ${DESCRIPTION}"
)
```

プランナーが返った後:
1. `${QUICK_DIR}/${next_num}-PLAN.md`に計画が存在することを確認
2. 計画数を抽出（クイックタスクでは通常1）
3. 報告: "計画を作成: ${QUICK_DIR}/${next_num}-PLAN.md"

計画が見つからない場合、エラー: "プランナーが${next_num}-PLAN.mdの作成に失敗"

---

**Step 5.5: 計画チェッカーループ（`$FULL_MODE`の場合のみ）**

`$FULL_MODE`でない場合はこのステップを完全にスキップ。

バナーを表示:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CHECKING PLAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ プランチェッカーを生成中...
```

チェッカープロンプト:

```markdown
<verification_context>
**Mode:** quick-full
**Task Description:** ${DESCRIPTION}

<files_to_read>
- ${QUICK_DIR}/${next_num}-PLAN.md (検証する計画)
</files_to_read>

**Scope:** これはクイックタスクであり、フルフェーズではない。ROADMAPフェーズゴールが必要なチェックはスキップ。
</verification_context>

<check_dimensions>
- 要件カバレッジ: 計画はタスクの説明に対応しているか？
- タスクの完全性: タスクにfiles, action, verify, doneフィールドがあるか？
- キーリンク: 参照されているファイルは実在するか？
- スコープの妥当性: クイックタスクとして適切なサイズか（1-3タスク）？
- must_havesの導出: must_havesはタスク説明に遡及可能か？

スキップ: クロスプラン依存（単一計画）、ROADMAPとの整合性
${DISCUSS_MODE ? '- コンテキスト準拠: 計画はCONTEXT.mdの確定された決定を遵守しているか？' : '- スキップ: コンテキスト準拠（CONTEXT.mdなし）'}
</check_dimensions>

<expected_output>
- ## VERIFICATION PASSED — すべてのチェックが合格
- ## ISSUES FOUND — 構造化された問題リスト
</expected_output>
```

```
Task(
  prompt=checker_prompt,
  subagent_type="gsd-plan-checker",
  model="{checker_model}",
  description="Check quick plan: ${DESCRIPTION}"
)
```

**チェッカーの戻り値の処理:**

- **`## VERIFICATION PASSED`:** 確認を表示、Step 6に進む。
- **`## ISSUES FOUND`:** 問題を表示、反復回数を確認、修正ループに入る。

**修正ループ（最大2回の反復）:**

`iteration_count`を追跡（初期計画 + チェック後に1から開始）。

**iteration_count < 2の場合:**

表示: `修正のためプランナーに差し戻し中... (反復 ${N}/2)`

修正プロンプト:

```markdown
<revision_context>
**Mode:** quick-full (revision)

<files_to_read>
- ${QUICK_DIR}/${next_num}-PLAN.md (既存の計画)
</files_to_read>

**チェッカーの問題:** ${structured_issues_from_checker}

</revision_context>

<instructions>
チェッカーの問題に対処するためのターゲットを絞った更新を行う。
問題が根本的でない限り、ゼロから再計画しないこと。
変更点を返すこと。
</instructions>
```

```
Task(
  prompt=revision_prompt,
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Revise quick plan: ${DESCRIPTION}"
)
```

プランナーが返った後 → チェッカーを再度生成、iteration_countをインクリメント。

**iteration_count >= 2の場合:**

表示: `最大反復回数に達しました。${N}件の問題が残っています:` + 問題リスト

提示: 1) 強制続行, 2) 中止

---

**Step 6: エクゼキューターの生成**

計画参照付きでgsd-executorを生成:

```
Task(
  prompt="
クイックタスク ${next_num} を実行。

<files_to_read>
- ${QUICK_DIR}/${next_num}-PLAN.md (計画)
- .planning/STATE.md (プロジェクト状態)
- ./CLAUDE.md (プロジェクト指示、存在する場合)
- .claude/skills/または.agents/skills/ (プロジェクトスキル、いずれかが存在する場合 — スキルを一覧し、各SKILL.mdを読み、実装中に関連するルールに従う)
</files_to_read>

<constraints>
- 計画のすべてのタスクを実行
- 各タスクをアトミックにコミット
- サマリーを作成: ${QUICK_DIR}/${next_num}-SUMMARY.md
- ROADMAP.mdを更新しないこと（クイックタスクは計画されたフェーズとは別）
</constraints>
",
  subagent_type="gsd-executor",
  model="{executor_model}",
  description="Execute: ${DESCRIPTION}"
)
```

エクゼキューターが返った後:
1. `${QUICK_DIR}/${next_num}-SUMMARY.md`にサマリーが存在することを確認
2. エクゼキューターの出力からコミットハッシュを抽出
3. 完了ステータスを報告

**既知のClaude Codeバグ (classifyHandoffIfNeeded):** エクゼキューターが`classifyHandoffIfNeeded is not defined`エラーで"failed"と報告した場合、これはClaude Codeのランタイムバグであり、実際の失敗ではない。サマリーファイルが存在し、git logにコミットが表示されるか確認。そうであれば成功として扱う。

サマリーが見つからない場合、エラー: "エクゼキューターが${next_num}-SUMMARY.mdの作成に失敗"

注: 複数の計画を生成するクイックタスク（まれ）の場合、execute-phaseパターンに従ってエクゼキューターを並列ウェーブで生成。

---

**Step 6.5: 検証（`$FULL_MODE`の場合のみ）**

`$FULL_MODE`でない場合はこのステップを完全にスキップ。

バナーを表示:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFYING RESULTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ ベリファイアを生成中...
```

```
Task(
  prompt="クイックタスクのゴール達成を検証。
タスクディレクトリ: ${QUICK_DIR}
タスクゴール: ${DESCRIPTION}

<files_to_read>
- ${QUICK_DIR}/${next_num}-PLAN.md (計画)
</files_to_read>

must_havesを実際のコードベースに対してチェック。${QUICK_DIR}/${next_num}-VERIFICATION.mdにVERIFICATION.mdを作成。",
  subagent_type="gsd-verifier",
  model="{verifier_model}",
  description="Verify: ${DESCRIPTION}"
)
```

検証ステータスを読み取る:
```bash
grep "^status:" "${QUICK_DIR}/${next_num}-VERIFICATION.md" | cut -d: -f2 | tr -d ' '
```

`$VERIFICATION_STATUS`として保存。

| ステータス | アクション |
|--------|--------|
| `passed` | `$VERIFICATION_STATUS = "Verified"`を保存、Step 7に続行 |
| `human_needed` | 手動チェックが必要な項目を表示、`$VERIFICATION_STATUS = "Needs Review"`を保存、続行 |
| `gaps_found` | ギャップサマリーを表示、提示: 1) ギャップ修正のためにエクゼキューターを再実行, 2) そのまま受け入れ。`$VERIFICATION_STATUS = "Gaps"`を保存 |

---

**Step 7: STATE.mdの更新**

クイックタスクの完了記録でSTATE.mdを更新。

**7a. "Quick Tasks Completed"セクションの存在確認:**

STATE.mdを読み取り、`### Quick Tasks Completed`セクションを確認。

**7b. セクションが存在しない場合、作成:**

`### Blockers/Concerns`セクションの後に挿入:

**`$FULL_MODE`の場合:**
```markdown
### Quick Tasks Completed

| # | Description | Date | Commit | Status | Directory |
|---|-------------|------|--------|--------|-----------|
```

**`$FULL_MODE`でない場合:**
```markdown
### Quick Tasks Completed

| # | Description | Date | Commit | Directory |
|---|-------------|------|--------|-----------|
```

**注:** テーブルが既に存在する場合、既存のカラム形式に合わせる。Statusカラムのないクイックタスクが既にあるプロジェクトに`--full`を追加する場合、ヘッダー行とセパレーター行にStatusカラムを追加し、新しい行の前の既存行のStatusは空のままにする。

**7c. テーブルに新しい行を追記:**

initからの`date`を使用:

**`$FULL_MODE`の場合（またはテーブルにStatusカラムがある場合）:**
```markdown
| ${next_num} | ${DESCRIPTION} | ${date} | ${commit_hash} | ${VERIFICATION_STATUS} | [${next_num}-${slug}](./quick/${next_num}-${slug}/) |
```

**`$FULL_MODE`でない場合（かつテーブルにStatusカラムがない場合）:**
```markdown
| ${next_num} | ${DESCRIPTION} | ${date} | ${commit_hash} | [${next_num}-${slug}](./quick/${next_num}-${slug}/) |
```

**7d. "Last activity"行の更新:**

initからの`date`を使用:
```
Last activity: ${date} - Completed quick task ${next_num}: ${DESCRIPTION}
```

Editツールを使用してこれらの変更をアトミックに行う

---

**Step 8: 最終コミットと完了**

クイックタスクの成果物をステージングしてコミット:

ファイルリストを構築:
- `${QUICK_DIR}/${next_num}-PLAN.md`
- `${QUICK_DIR}/${next_num}-SUMMARY.md`
- `.planning/STATE.md`
- `$DISCUSS_MODE`でコンテキストファイルが存在する場合: `${QUICK_DIR}/${next_num}-CONTEXT.md`
- `$FULL_MODE`で検証ファイルが存在する場合: `${QUICK_DIR}/${next_num}-VERIFICATION.md`

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(quick-${next_num}): ${DESCRIPTION}" --files ${file_list}
```

最終コミットハッシュを取得:
```bash
commit_hash=$(git rev-parse --short HEAD)
```

完了出力を表示:

**`$FULL_MODE`の場合:**
```
---

GSD > QUICK TASK COMPLETE (FULL MODE)

クイックタスク ${next_num}: ${DESCRIPTION}

サマリー: ${QUICK_DIR}/${next_num}-SUMMARY.md
検証: ${QUICK_DIR}/${next_num}-VERIFICATION.md (${VERIFICATION_STATUS})
コミット: ${commit_hash}

---

次のタスクの準備完了: /gsd:quick
```

**`$FULL_MODE`でない場合:**
```
---

GSD > QUICK TASK COMPLETE

クイックタスク ${next_num}: ${DESCRIPTION}

サマリー: ${QUICK_DIR}/${next_num}-SUMMARY.md
コミット: ${commit_hash}

---

次のタスクの準備完了: /gsd:quick
```

</process>

<success_criteria>
- [ ] ROADMAP.mdのバリデーションが合格
- [ ] ユーザーがタスクの説明を提供
- [ ] 引数に存在する場合、`--full`と`--discuss`フラグをパース
- [ ] スラグを生成（小文字、ハイフン、最大40文字）
- [ ] 次の番号を計算（001, 002, 003...）
- [ ] `.planning/quick/NNN-slug/`にディレクトリを作成
- [ ] (--discuss) 曖昧な点を特定して提示、決定を`${next_num}-CONTEXT.md`に記録
- [ ] プランナーが`${next_num}-PLAN.md`を作成（--discussの場合はCONTEXT.mdの決定を遵守）
- [ ] (--full) プランチェッカーが計画を検証、修正ループは最大2回
- [ ] エクゼキューターが`${next_num}-SUMMARY.md`を作成
- [ ] (--full) ベリファイアが`${next_num}-VERIFICATION.md`を作成
- [ ] STATE.mdにクイックタスク行を更新（--fullの場合はStatusカラム）
- [ ] 成果物をコミット
</success_criteria>
</output>
