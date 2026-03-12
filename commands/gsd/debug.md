---
name: gsd:debug
description: コンテキストリセットをまたいで永続的な状態を保持する体系的デバッグ
argument-hint: [issue description]
allowed-tools:
  - Read
  - Bash
  - Task
  - AskUserQuestion
---

<objective>
サブエージェント分離による科学的手法でissueをデバッグする。

**オーケストレーターの役割:** 症状の収集、gsd-debuggerエージェントの起動、チェックポイントの処理、継続の起動。

**サブエージェントを使う理由:** 調査はコンテキストを急速に消費する（ファイルの読み取り、仮説の構築、テスト）。調査ごとにフレッシュな200kコンテキストを確保。メインコンテキストはユーザーとのやり取りのためにスリムに保つ。
</objective>

<context>
ユーザーのissue: $ARGUMENTS

アクティブセッションの確認:
```bash
ls .planning/debug/*.md 2>/dev/null | grep -v resolved | head -5
```
</context>

<process>

## 0. コンテキストの初期化

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから`commit_docs`を抽出する。debuggerモデルを解決:
```bash
debugger_model=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-debugger --raw)
```

## 1. アクティブセッションの確認

アクティブセッションが存在し、$ARGUMENTSがない場合:
- ステータス、仮説、次のアクションとともにセッション一覧を表示
- ユーザーが再開する番号を選択するか、新しいissueを説明

$ARGUMENTSが指定されている場合、またはユーザーが新しいissueを説明した場合:
- 症状収集に進む

## 2. 症状の収集（新しいissueの場合）

各項目にAskUserQuestionを使用:

1. **期待される動作** - 何が起こるべきか？
2. **実際の動作** - 代わりに何が起こるか？
3. **エラーメッセージ** - エラーはあるか？（貼り付けまたは説明）
4. **タイムライン** - いつから始まったか？以前は動作していたか？
5. **再現方法** - どうやってトリガーするか？

すべて収集後、調査を開始する準備ができたことを確認する。

## 3. gsd-debuggerエージェントの起動

プロンプトを埋めて起動:

```markdown
<objective>
issueの調査: {slug}

**概要:** {trigger}
</objective>

<symptoms>
expected: {expected}
actual: {actual}
errors: {errors}
reproduction: {reproduction}
timeline: {timeline}
</symptoms>

<mode>
symptoms_prefilled: true
goal: find_and_fix
</mode>

<debug_file>
Create: .planning/debug/{slug}.md
</debug_file>
```

```
Task(
  prompt=filled_prompt,
  subagent_type="gsd-debugger",
  model="{debugger_model}",
  description="Debug {slug}"
)
```

## 4. エージェントの返却処理

**`## ROOT CAUSE FOUND`の場合:**
- 根本原因と証拠の要約を表示
- オプションを提示:
  - "今すぐ修正" - 修正サブエージェントを起動
  - "修正を計画" - /gsd:plan-phase --gaps を提案
  - "手動で修正" - 完了

**`## CHECKPOINT REACHED`の場合:**
- チェックポイントの詳細をユーザーに提示
- ユーザーの応答を取得
- チェックポイントタイプが`human-verify`の場合:
  - ユーザーが修正を確認した場合: エージェントがファイナライズ/解決/アーカイブできるように続行
  - ユーザーが問題を報告した場合: エージェントが調査/修正に戻れるように続行
- 継続エージェントを起動（ステップ5参照）

**`## INVESTIGATION INCONCLUSIVE`の場合:**
- 調査済みの内容と除外されたものを表示
- オプションを提示:
  - "調査を継続" - 追加コンテキストで新しいエージェントを起動
  - "手動で調査" - 完了
  - "コンテキストを追加" - さらに症状を収集して再起動

## 5. 継続エージェントの起動（チェックポイント後）

ユーザーがチェックポイントに応答したら、フレッシュなエージェントを起動:

```markdown
<objective>
{slug}のデバッグを継続。証拠はデバッグファイルにある。
</objective>

<prior_state>
<files_to_read>
- .planning/debug/{slug}.md (デバッグセッション状態)
</files_to_read>
</prior_state>

<checkpoint_response>
**Type:** {checkpoint_type}
**Response:** {user_response}
</checkpoint_response>

<mode>
goal: find_and_fix
</mode>
```

```
Task(
  prompt=continuation_prompt,
  subagent_type="gsd-debugger",
  model="{debugger_model}",
  description="Continue debug {slug}"
)
```

</process>

<success_criteria>
- [ ] アクティブセッションを確認済み
- [ ] 症状を収集済み（新しいissueの場合）
- [ ] gsd-debuggerをコンテキスト付きで起動済み
- [ ] チェックポイントを正しく処理済み
- [ ] 修正前に根本原因を確認済み
</success_criteria>

