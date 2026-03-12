<purpose>
保留中のTODOを一覧表示し、選択を可能にし、選択されたTODOの完全なコンテキストを読み込み、適切なアクションにルーティングする。
</purpose>

<required_reading>
開始前に、呼び出し元プロンプトのexecution_contextで参照されているすべてのファイルを読み込むこと。
</required_reading>

<process>

<step name="init_context">
TODOコンテキストを読み込む：

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init todos)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから `todo_count`、`todos`、`pending_dir` を抽出する。

`todo_count` が0の場合：
```
保留中のTODOはありません。

TODOは /gsd:add-todo で作業セッション中にキャプチャされる。

---

次のアクション：

1. 現在のフェーズを続行 (/gsd:progress)
2. 今すぐTODOを追加 (/gsd:add-todo)
```

終了。
</step>

<step name="parse_filter">
引数のエリアフィルタを確認する：
- `/gsd:check-todos` → すべて表示
- `/gsd:check-todos api` → area:api のみにフィルタ
</step>

<step name="list_todos">
initコンテキストの `todos` 配列を使用する（指定された場合はエリアでフィルタ済み）。

パースして番号付きリストとして表示する：

```
保留中のTODO：

1. 認証トークンのリフレッシュを追加 (api, 2日前)
2. モーダルのz-index問題を修正 (ui, 1日前)
3. データベース接続プールのリファクタリング (database, 5時間前)

---

詳細を見るには番号で返信、または：
- `/gsd:check-todos [area]` でエリアでフィルタ
- `q` で終了
```

作成タイムスタンプからの相対時間として経過時間をフォーマットする。
</step>

<step name="handle_selection">
ユーザーが番号で返信するのを待つ。

有効な場合：選択されたTODOを読み込み、続行する。
無効な場合："無効な選択。番号 (1-[N]) を入力するか、`q` で終了すること。"
</step>

<step name="load_context">
TODOファイルを完全に読み込む。表示する：

```
## [title]

**Area:** [area]
**Created:** [date] ([relative time]前)
**Files:** [リスト または "なし"]

### 問題
[problem セクションの内容]

### 解決策
[solution セクションの内容]
```

`files` フィールドにエントリがある場合、それぞれを読み込み簡潔に要約する。
</step>

<step name="check_roadmap">
ロードマップを確認する（init progressを使用するか、ファイルの存在を直接確認可能）：

`.planning/ROADMAP.md` が存在する場合：
1. TODOのエリアが次のフェーズと一致するか確認する
2. TODOのファイルがフェーズのスコープと重複するか確認する
3. アクションオプションのためにマッチを記録する
</step>

<step name="offer_actions">
**TODOがロードマップのフェーズにマッピングされる場合：**

AskUserQuestionを使用する：
- header: "Action"
- question: "このTODOはフェーズ [N]: [name] に関連しています。どうしますか？"
- options:
  - "今すぐ作業する" — doneに移動して作業を開始
  - "フェーズ計画に追加" — フェーズ [N] の計画時に含める
  - "アプローチをブレインストーミング" — 決定する前に考える
  - "戻す" — リストに戻る

**ロードマップのマッチがない場合：**

AskUserQuestionを使用する：
- header: "Action"
- question: "このTODOをどうしますか？"
- options:
  - "今すぐ作業する" — doneに移動して作業を開始
  - "フェーズを作成" — このスコープで /gsd:add-phase
  - "アプローチをブレインストーミング" — 決定する前に考える
  - "戻す" — リストに戻る
</step>

<step name="execute_action">
**今すぐ作業する：**
```bash
mv ".planning/todos/pending/[filename]" ".planning/todos/done/"
```
STATE.mdのTODOカウントを更新する。問題/解決策のコンテキストを表示する。作業を開始するか、どう進めるか確認する。

**フェーズ計画に追加：**
フェーズの計画メモにTODO参照を記録する。pendingに保持する。リストに戻るか終了する。

**フェーズを作成：**
表示する：`/gsd:add-phase [TODOからの説明]`
pendingに保持する。ユーザーが新しいコンテキストでコマンドを実行する。

**アプローチをブレインストーミング：**
pendingに保持する。問題とアプローチについての議論を開始する。

**戻す：**
list_todosステップに戻る。
</step>

<step name="update_state">
TODOカウントを変更するアクションの後：

`init todos` を再実行して更新されたカウントを取得し、STATE.mdの "### Pending Todos" セクションが存在する場合は更新する。
</step>

<step name="git_commit">
TODOがdone/に移動された場合、変更をコミットする：

```bash
git rm --cached .planning/todos/pending/[filename] 2>/dev/null || true
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: start work on todo - [title]" --files .planning/todos/done/[filename] .planning/STATE.md
```

ツールは `commit_docs` 設定とgitignoreを自動的に尊重する。

確認表示："Committed: docs: start work on todo - [title]"
</step>

</process>

<success_criteria>
- [ ] すべての保留中TODOがタイトル、エリア、経過時間とともに一覧表示されている
- [ ] 指定された場合にエリアフィルタが適用されている
- [ ] 選択されたTODOの完全なコンテキストが読み込まれている
- [ ] フェーズマッチのためにロードマップコンテキストが確認されている
- [ ] 適切なアクションが提示されている
- [ ] 選択されたアクションが実行されている
- [ ] TODOカウントが変更された場合にSTATE.mdが更新されている
- [ ] 変更がgitにコミットされている（TODOがdone/に移動された場合）
</success_criteria>

