<purpose>

出荷されたバージョン（v1.0、v1.1、v2.0）を完了としてマークする。MILESTONES.mdに履歴レコードを作成し、PROJECT.mdの完全な進化レビューを実施し、マイルストーングループ化によるROADMAP.mdの再編成を行い、gitでリリースにタグを付ける。

</purpose>

<required_reading>

1. templates/milestone.md
2. templates/milestone-archive.md
3. `.planning/ROADMAP.md`
4. `.planning/REQUIREMENTS.md`
5. `.planning/PROJECT.md`

</required_reading>

<archival_behavior>

マイルストーン完了時：

1. マイルストーンの詳細を `.planning/milestones/v[X.Y]-ROADMAP.md` に抽出
2. 要件を `.planning/milestones/v[X.Y]-REQUIREMENTS.md` にアーカイブ
3. ROADMAP.md を更新 — マイルストーンの詳細を1行サマリーに置換
4. REQUIREMENTS.md を削除（次のマイルストーン用に新規作成）
5. PROJECT.md の完全な進化レビューを実施
6. 次のマイルストーンのインライン作成を提案

**コンテキスト効率：** アーカイブによりROADMAP.mdのサイズを一定に保ち、REQUIREMENTS.mdをマイルストーンスコープに限定する。

**ROADMAPアーカイブ** は `templates/milestone-archive.md` を使用 — マイルストーンヘッダー（ステータス、フェーズ、日付）、完全なフェーズ詳細、マイルストーンサマリー（決定事項、問題、技術的負債）を含む。

**要件アーカイブ** は成果付きで完了マークされたすべての要件、最終ステータス付きのトレーサビリティテーブル、変更された要件のメモを含む。

</archival_behavior>

<process>

<step name="verify_readiness">

**包括的な準備状況チェックに `roadmap analyze` を使用する：**

```bash
ROADMAP=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)
```

これはすべてのフェーズの計画/サマリー数とディスクステータスを返す。以下を検証するために使用する：
- このマイルストーンにどのフェーズが属するか？
- すべてのフェーズが完了しているか（すべての計画にサマリーがあるか）？各フェーズの `disk_status === 'complete'` を確認する。
- `progress_percent` が100%であるべき。

**要件完了チェック（表示前に必須）：**

REQUIREMENTS.mdのトレーサビリティテーブルをパースする：
- v1要件の合計数とチェック済み（`[x]`）の要件数を数える
- トレーサビリティテーブルでComplete以外の行を識別する

表示する：

```
マイルストーン: [名前、例："v1.0 MVP"]

含まれるフェーズ：
- Phase 1: Foundation (2/2 計画完了)
- Phase 2: Authentication (2/2 計画完了)
- Phase 3: Core Features (3/3 計画完了)
- Phase 4: Polish (1/1 計画完了)

合計: {phase_count} フェーズ、{total_plans} 計画、すべて完了
要件: {N}/{M} v1要件チェック済み
```

**要件が未完了の場合**（N < M）：

```
⚠ 未チェックの要件：

- [ ] {REQ-ID}: {description} (Phase {X})
- [ ] {REQ-ID}: {description} (Phase {Y})
```

3つのオプションを提示する（必須）：
1. **そのまま続行** — 既知のギャップありでマイルストーンを完了としてマーク
2. **先に監査を実行** — `/gsd:audit-milestone` でギャップの深刻度を評価
3. **中止** — 開発に戻る

ユーザーが "そのまま続行" を選択した場合：未完了の要件をMILESTONES.mdの `### Known Gaps` にREQ-IDと説明付きで記録する。

<config-check>

```bash
cat .planning/config.json 2>/dev/null
```

</config-check>

<if mode="yolo">

```
⚡ 自動承認: マイルストーンスコープの検証
[プロンプトなしで内訳サマリーを表示]
統計情報の収集に進みます...
```

gather_statsに進む。

</if>

<if mode="interactive" OR="custom with gates.confirm_milestone_scope true">

```
このマイルストーンを出荷済みとしてマークする準備はできていますか？
(yes / wait / adjust scope)
```

確認を待つ。
- "adjust scope": どのフェーズを含めるか確認する。
- "wait": 停止する。準備ができたらユーザーが戻る。

</if>

</step>

<step name="gather_stats">

マイルストーンの統計情報を計算する：

```bash
git log --oneline --grep="feat(" | head -20
git diff --stat FIRST_COMMIT..LAST_COMMIT | tail -1
find . -name "*.swift" -o -name "*.ts" -o -name "*.py" | xargs wc -l 2>/dev/null
git log --format="%ai" FIRST_COMMIT | tail -1
git log --format="%ai" LAST_COMMIT | head -1
```

表示する：

```
マイルストーン統計:
- フェーズ: [X-Y]
- 計画: [Z] 合計
- タスク: [N] 合計（フェーズサマリーから）
- 変更ファイル: [M]
- コード行数: [LOC] [言語]
- タイムライン: [日数] 日 ([開始] → [終了])
- Gitレンジ: feat(XX-XX) → feat(YY-YY)
```

</step>

<step name="extract_accomplishments">

summary-extractを使用してSUMMARY.mdファイルからワンライナーを抽出する：

```bash
# マイルストーン内の各フェーズについてワンライナーを抽出
for summary in .planning/phases/*-*/*-SUMMARY.md; do
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" summary-extract "$summary" --fields one_liner | jq -r '.one_liner'
done
```

4-6個の主要な成果を抽出する。表示する：

```
このマイルストーンの主要な成果：
1. [フェーズ1からの成果]
2. [フェーズ2からの成果]
3. [フェーズ3からの成果]
4. [フェーズ4からの成果]
5. [フェーズ5からの成果]
```

</step>

<step name="create_milestone_entry">

**注意：** MILESTONES.mdのエントリは、archive_milestoneステップの `gsd-tools milestone complete` によって自動的に作成される。エントリにはバージョン、日付、フェーズ/計画/タスク数、SUMMARY.mdファイルから抽出された成果が含まれる。

追加の詳細が必要な場合（例：ユーザー提供の "Delivered" サマリー、gitレンジ、LOC統計）、CLIがベースエントリを作成した後に手動で追加する。

</step>

<step name="evolve_project_full_review">

マイルストーン完了時のPROJECT.md完全進化レビュー。

すべてのフェーズサマリーを読み込む：

```bash
cat .planning/phases/*-*/*-SUMMARY.md
```

**完全レビューチェックリスト：**

1. **"What This Is" の正確性：**
   - 現在の説明と構築されたものを比較
   - プロダクトが意味のある変化をした場合は更新

2. **Core Valueの確認：**
   - まだ正しい優先度か？出荷により異なるcore valueが明らかになったか？
   - そのONE THINGが変わった場合は更新

3. **要件の監査：**

   **Validatedセクション：**
   - このマイルストーンで出荷されたすべてのActive要件 → Validatedに移動
   - フォーマット: `- ✓ [要件] — v[X.Y]`

   **Activeセクション：**
   - Validatedに移動した要件を削除
   - 次のマイルストーンの新しい要件を追加
   - 未対応の要件を保持

   **Out of Scopeの監査：**
   - 各項目を確認 — 理由がまだ有効か？
   - 関連性のない項目を削除
   - マイルストーン中に無効になった要件を追加

4. **コンテキストの更新：**
   - 現在のコードベース状態（LOC、技術スタック）
   - ユーザーフィードバックのテーマ（ある場合）
   - 既知の問題や技術的負債

5. **Key Decisionsの監査：**
   - マイルストーンのフェーズサマリーからすべての決定事項を抽出
   - 成果付きでKey Decisionsテーブルに追加
   - ✓ Good、⚠️ Revisit、または — Pending をマーク

6. **制約の確認：**
   - 開発中に変更された制約があるか？必要に応じて更新

PROJECT.mdをインラインで更新する。"Last updated" フッターを更新する：

```markdown
---
*Last updated: [date] after v[X.Y] milestone*
```

**完全進化の例（v1.0 → v1.1準備）：**

Before:

```markdown
## What This Is

リモートチーム向けのリアルタイム共同ホワイトボード。

## Core Value

即座に感じるリアルタイム同期。

## Requirements

### Validated

(まだなし — 出荷して検証)

### Active

- [ ] キャンバス描画ツール
- [ ] リアルタイム同期 < 500ms
- [ ] ユーザー認証
- [ ] PNGエクスポート

### Out of Scope

- モバイルアプリ — Webファーストアプローチ
- ビデオチャット — 外部ツールを使用
```

v1.0後:

```markdown
## What This Is

即座の同期と描画ツールを備えたリモートチーム向けリアルタイム共同ホワイトボード。

## Core Value

即座に感じるリアルタイム同期。

## Requirements

### Validated

- ✓ キャンバス描画ツール — v1.0
- ✓ リアルタイム同期 < 500ms — v1.0 (平均200ms達成)
- ✓ ユーザー認証 — v1.0

### Active

- [ ] PNGエクスポート
- [ ] 元に戻す/やり直し履歴
- [ ] 図形ツール（長方形、円）

### Out of Scope

- モバイルアプリ — Webファーストアプローチ、PWAが良好に機能
- ビデオチャット — 外部ツールを使用
- オフラインモード — リアルタイムがcore value

## Context

v1.0を2,400行のTypeScriptで出荷。
技術スタック: Next.js, Supabase, Canvas API。
初期ユーザーテストで図形ツールの需要が判明。
```

**ステップ完了条件：**

- [ ] "What This Is" がレビューされ、必要に応じて更新されている
- [ ] Core Valueがまだ正しいことが確認されている
- [ ] 出荷されたすべての要件がValidatedに移動されている
- [ ] 次のマイルストーンの新しい要件がActiveに追加されている
- [ ] Out of Scopeの理由が監査されている
- [ ] コンテキストが現在の状態で更新されている
- [ ] マイルストーンのすべての決定事項がKey Decisionsに追加されている
- [ ] "Last updated" フッターがマイルストーン完了を反映している

</step>

<step name="reorganize_roadmap">

`.planning/ROADMAP.md` を更新する — 完了したマイルストーンのフェーズをグループ化する：

```markdown
# Roadmap: [プロジェクト名]

## Milestones

- ✅ **v1.0 MVP** — Phases 1-4 (出荷 YYYY-MM-DD)
- 🚧 **v1.1 Security** — Phases 5-6 (進行中)
- 📋 **v2.0 Redesign** — Phases 7-10 (計画済み)

## Phases

<details>
<summary>✅ v1.0 MVP (Phases 1-4) — SHIPPED YYYY-MM-DD</summary>

- [x] Phase 1: Foundation (2/2 計画) — 完了 YYYY-MM-DD
- [x] Phase 2: Authentication (2/2 計画) — 完了 YYYY-MM-DD
- [x] Phase 3: Core Features (3/3 計画) — 完了 YYYY-MM-DD
- [x] Phase 4: Polish (1/1 計画) — 完了 YYYY-MM-DD

</details>

### 🚧 v[Next] [名前] (進行中 / 計画済み)

- [ ] Phase 5: [名前] ([N] 計画)
- [ ] Phase 6: [名前] ([N] 計画)

## Progress

| Phase             | Milestone | Plans Complete | Status      | Completed  |
| ----------------- | --------- | -------------- | ----------- | ---------- |
| 1. Foundation     | v1.0      | 2/2            | Complete    | YYYY-MM-DD |
| 2. Authentication | v1.0      | 2/2            | Complete    | YYYY-MM-DD |
| 3. Core Features  | v1.0      | 3/3            | Complete    | YYYY-MM-DD |
| 4. Polish         | v1.0      | 1/1            | Complete    | YYYY-MM-DD |
| 5. Security Audit | v1.1      | 0/1            | Not started | -          |
| 6. Hardening      | v1.1      | 0/2            | Not started | -          |
```

</step>

<step name="archive_milestone">

**アーカイブをgsd-toolsに委譲する：**

```bash
ARCHIVE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" milestone complete "v[X.Y]" --name "[マイルストーン名]")
```

CLIが処理する内容：
- `.planning/milestones/` ディレクトリの作成
- ROADMAP.mdを `milestones/v[X.Y]-ROADMAP.md` にアーカイブ
- REQUIREMENTS.mdを `milestones/v[X.Y]-REQUIREMENTS.md` にアーカイブヘッダー付きでアーカイブ
- 監査ファイルが存在する場合はmilestonesに移動
- SUMMARY.mdファイルからの成果付きでMILESTONES.mdエントリを作成/追記
- STATE.mdの更新（ステータス、最終アクティビティ）

結果から抽出する：`version`、`date`、`phases`、`plans`、`tasks`、`accomplishments`、`archived`。

確認する：`✅ マイルストーンが .planning/milestones/ にアーカイブされました`

**フェーズアーカイブ（オプション）：** アーカイブ完了後、ユーザーに確認する：

AskUserQuestion(header="Archive Phases", question="フェーズディレクトリをmilestones/にアーカイブしますか？", options: "はい — milestones/v[X.Y]-phases/ に移動" | "スキップ — フェーズをそのまま保持")

"はい" の場合：フェーズディレクトリをマイルストーンアーカイブに移動する：
```bash
mkdir -p .planning/milestones/v[X.Y]-phases
# .planning/phases/ 内の各フェーズディレクトリについて：
mv .planning/phases/{phase-dir} .planning/milestones/v[X.Y]-phases/
```
確認する：`✅ フェーズディレクトリが .planning/milestones/v[X.Y]-phases/ にアーカイブされました`

"スキップ" の場合：フェーズディレクトリは `.planning/phases/` に生の実行履歴として残る。後で `/gsd:cleanup` を使って遡及的にアーカイブ可能。

アーカイブ後、AIが引き続き処理する内容：
- マイルストーングループ化によるROADMAP.mdの再編成（判断が必要）
- PROJECT.mdの完全進化レビュー（理解が必要）
- 元のROADMAP.mdとREQUIREMENTS.mdの削除
- これらはコンテンツのAI解釈が必要なため完全には委譲されない

</step>

<step name="reorganize_roadmap_and_delete_originals">

`milestone complete` がアーカイブした後、マイルストーングループ化でROADMAP.mdを再編成し、元ファイルを削除する：

**ROADMAP.mdを再編成** — 完了したマイルストーンのフェーズをグループ化する：

```markdown
# Roadmap: [プロジェクト名]

## Milestones

- ✅ **v1.0 MVP** — Phases 1-4 (出荷 YYYY-MM-DD)
- 🚧 **v1.1 Security** — Phases 5-6 (進行中)

## Phases

<details>
<summary>✅ v1.0 MVP (Phases 1-4) — SHIPPED YYYY-MM-DD</summary>

- [x] Phase 1: Foundation (2/2 計画) — 完了 YYYY-MM-DD
- [x] Phase 2: Authentication (2/2 計画) — 完了 YYYY-MM-DD

</details>
```

**その後、元ファイルを削除する：**

```bash
rm .planning/ROADMAP.md
rm .planning/REQUIREMENTS.md
```

</step>

<step name="write_retrospective">

**リビングレトロスペクティブに追記する：**

既存のレトロスペクティブを確認する：
```bash
ls .planning/RETROSPECTIVE.md 2>/dev/null
```

**存在する場合：** ファイルを読み込み、"## Cross-Milestone Trends" セクションの前に新しいマイルストーンセクションを追記する。

**存在しない場合：** `~/.claude/get-shit-done/templates/retrospective.md` のテンプレートから作成する。

**レトロスペクティブデータを収集する：**

1. SUMMARY.mdファイルから：主要な成果物、ワンライナー、技術的決定を抽出
2. VERIFICATION.mdファイルから：検証スコア、発見されたギャップを抽出
3. UAT.mdファイルから：テスト結果、発見された問題を抽出
4. gitログから：コミット数を数え、タイムラインを計算
5. マイルストーンの作業から：うまくいったことと改善が必要なことを振り返る

**マイルストーンセクションを書く：**

```markdown
## Milestone: v{version} — {name}

**出荷日:** {date}
**フェーズ:** {phase_count} | **計画:** {plan_count}

### 構築したもの
{SUMMARY.mdのワンライナーから抽出}

### うまくいったこと
{スムーズな実行につながったパターン}

### 非効率だったこと
{逃したチャンス、やり直し、ボトルネック}

### 確立されたパターン
{このマイルストーンで発見された新しい規約}

### 主な教訓
{具体的で実行可能なテイクアウェイ}

### コスト観察
- モデル配分: {X}% opus, {Y}% sonnet, {Z}% haiku
- セッション数: {count}
- 注目: {効率に関する観察}
```

**マイルストーン横断のトレンドを更新する：**

"## Cross-Milestone Trends" セクションが存在する場合、このマイルストーンの新しいデータでテーブルを更新する。

**コミット：**
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: update retrospective for v${VERSION}" --files .planning/RETROSPECTIVE.md
```

</step>

<step name="update_state">

STATE.mdの更新の大部分は `milestone complete` で処理されているが、残りのフィールドを確認して更新する：

**Project Reference：**

```markdown
## Project Reference

See: .planning/PROJECT.md (updated [today])

**Core value:** [PROJECT.mdからの現在のcore value]
**Current focus:** [次のマイルストーン または "次のマイルストーンを計画中"]
```

**Accumulated Context：**
- 決定事項のサマリーをクリア（完全なログはPROJECT.mdに）
- 解決済みブロッカーをクリア
- 次のマイルストーンの未解決ブロッカーを保持

</step>

<step name="handle_branches">

ブランチ戦略を確認し、マージオプションを提示する。

コンテキストには `init milestone-op` を使用するか、設定を直接読み込む：

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init execute-phase "1")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから `branching_strategy`、`phase_branch_template`、`milestone_branch_template`、`commit_docs` を抽出する。

**"none" の場合：** git_tagにスキップする。

**"phase" 戦略の場合：**

```bash
BRANCH_PREFIX=$(echo "$PHASE_BRANCH_TEMPLATE" | sed 's/{.*//')
PHASE_BRANCHES=$(git branch --list "${BRANCH_PREFIX}*" 2>/dev/null | sed 's/^\*//' | tr -d ' ')
```

**"milestone" 戦略の場合：**

```bash
BRANCH_PREFIX=$(echo "$MILESTONE_BRANCH_TEMPLATE" | sed 's/{.*//')
MILESTONE_BRANCH=$(git branch --list "${BRANCH_PREFIX}*" 2>/dev/null | sed 's/^\*//' | tr -d ' ' | head -1)
```

**ブランチが見つからない場合：** git_tagにスキップする。

**ブランチが存在する場合：**

```
## 検出されたGitブランチ

ブランチ戦略: {phase/milestone}
ブランチ: {一覧}

オプション：
1. **mainにマージ** — ブランチをmainにマージ
2. **マージせずに削除** — 既にマージ済みまたは不要
3. **ブランチを保持** — 手動処理のために残す
```

AskUserQuestion オプション: Squash merge（推奨）, Merge with history, マージせずに削除, ブランチを保持。

**Squash merge：**

```bash
CURRENT_BRANCH=$(git branch --show-current)
git checkout main

if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  for branch in $PHASE_BRANCHES; do
    git merge --squash "$branch"
    # commit_docsがfalseの場合、ステージングから.planning/を除外
    if [ "$COMMIT_DOCS" = "false" ]; then
      git reset HEAD .planning/ 2>/dev/null || true
    fi
    git commit -m "feat: $branch for v[X.Y]"
  done
fi

if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  git merge --squash "$MILESTONE_BRANCH"
  # commit_docsがfalseの場合、ステージングから.planning/を除外
  if [ "$COMMIT_DOCS" = "false" ]; then
    git reset HEAD .planning/ 2>/dev/null || true
  fi
  git commit -m "feat: $MILESTONE_BRANCH for v[X.Y]"
fi

git checkout "$CURRENT_BRANCH"
```

**Merge with history：**

```bash
CURRENT_BRANCH=$(git branch --show-current)
git checkout main

if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  for branch in $PHASE_BRANCHES; do
    git merge --no-ff --no-commit "$branch"
    # commit_docsがfalseの場合、ステージングから.planning/を除外
    if [ "$COMMIT_DOCS" = "false" ]; then
      git reset HEAD .planning/ 2>/dev/null || true
    fi
    git commit -m "Merge branch '$branch' for v[X.Y]"
  done
fi

if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  git merge --no-ff --no-commit "$MILESTONE_BRANCH"
  # commit_docsがfalseの場合、ステージングから.planning/を除外
  if [ "$COMMIT_DOCS" = "false" ]; then
    git reset HEAD .planning/ 2>/dev/null || true
  fi
  git commit -m "Merge branch '$MILESTONE_BRANCH' for v[X.Y]"
fi

git checkout "$CURRENT_BRANCH"
```

**マージせずに削除：**

```bash
if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  for branch in $PHASE_BRANCHES; do
    git branch -d "$branch" 2>/dev/null || git branch -D "$branch"
  done
fi

if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  git branch -d "$MILESTONE_BRANCH" 2>/dev/null || git branch -D "$MILESTONE_BRANCH"
fi
```

**ブランチを保持：** "ブランチは手動処理のために保持されています" と報告する

</step>

<step name="git_tag">

gitタグを作成する：

```bash
git tag -a v[X.Y] -m "v[X.Y] [名前]

Delivered: [1文]

主要な成果：
- [項目1]
- [項目2]
- [項目3]

詳細は .planning/MILESTONES.md を参照。"
```

確認する："タグ作成完了: v[X.Y]"

確認する："タグをリモートにプッシュしますか？ (y/n)"

はいの場合：
```bash
git push origin v[X.Y]
```

</step>

<step name="git_commit_milestone">

マイルストーン完了をコミットする。

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "chore: complete v[X.Y] milestone" --files .planning/milestones/v[X.Y]-ROADMAP.md .planning/milestones/v[X.Y]-REQUIREMENTS.md .planning/milestones/v[X.Y]-MILESTONE-AUDIT.md .planning/MILESTONES.md .planning/PROJECT.md .planning/STATE.md
```
```

確認する："Committed: chore: complete v[X.Y] milestone"

</step>

<step name="offer_next">

```
✅ マイルストーン v[X.Y] [名前] 完了

出荷内容：
- [N] フェーズ ([M] 計画、[P] タスク)
- [出荷内容の1文]

アーカイブ先：
- milestones/v[X.Y]-ROADMAP.md
- milestones/v[X.Y]-REQUIREMENTS.md

サマリー: .planning/MILESTONES.md
タグ: v[X.Y]

---

## ▶ Next Up

**次のマイルストーンを開始** — 質問 → 調査 → 要件 → ロードマップ

`/gsd:new-milestone`

<sub>`/clear` first → fresh context window</sub>

---
```

</step>

</process>

<milestone_naming>

**バージョン規約：**
- **v1.0** — 初期MVP
- **v1.1, v1.2** — マイナーアップデート、新機能、修正
- **v2.0, v3.0** — 大規模リライト、破壊的変更、新方向性

**名前：** 短い1-2単語（v1.0 MVP, v1.1 Security, v1.2 Performance, v2.0 Redesign）。

</milestone_naming>

<what_qualifies>

**マイルストーンを作成するケース：** 初回リリース、公開リリース、主要な機能セットの出荷、計画のアーカイブ前。

**マイルストーンを作成しないケース：** 各フェーズの完了（粒度が細かすぎる）、作業中、内部開発イテレーション（本当に出荷されていない限り）。

ヒューリスティック："これはデプロイ/使用可能/出荷されているか？" はいなら → マイルストーン。いいえなら → 作業を続ける。

</what_qualifies>

<success_criteria>

マイルストーン完了が成功する条件：

- [ ] MILESTONES.mdエントリが統計と成果付きで作成されている
- [ ] PROJECT.mdの完全進化レビューが完了している
- [ ] 出荷されたすべての要件がPROJECT.mdのValidatedに移動されている
- [ ] Key Decisionsが成果付きで更新されている
- [ ] ROADMAP.mdがマイルストーングループ化で再編成されている
- [ ] ロードマップアーカイブが作成されている (milestones/v[X.Y]-ROADMAP.md)
- [ ] 要件アーカイブが作成されている (milestones/v[X.Y]-REQUIREMENTS.md)
- [ ] REQUIREMENTS.mdが削除されている（次のマイルストーン用に新規作成）
- [ ] STATE.mdが新しいプロジェクト参照で更新されている
- [ ] Gitタグが作成されている (v[X.Y])
- [ ] マイルストーンコミットが作成されている（アーカイブファイルと削除を含む）
- [ ] REQUIREMENTS.mdのトレーサビリティテーブルに対して要件の完了が確認されている
- [ ] 未完了の要件が続行/監査/中止オプション付きで表示されている
- [ ] ユーザーが未完了の要件で続行した場合、既知のギャップがMILESTONES.mdに記録されている
- [ ] RETROSPECTIVE.mdがマイルストーンセクションで更新されている
- [ ] マイルストーン横断のトレンドが更新されている
- [ ] ユーザーが次のステップを知っている (/gsd:new-milestone)

</success_criteria>
</output>
