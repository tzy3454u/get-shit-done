<purpose>
GSDセッション中に浮上したアイデア、タスク、または問題を、後で作業するための構造化されたtodoとしてキャプチャする。コンテキストを失わずに「思いつき → 記録 → 続行」のフローを可能にする。
</purpose>

<required_reading>
開始前に、呼び出しプロンプトのexecution_contextで参照されるすべてのファイルを読み取ること。
</required_reading>

<process>

<step name="init_context">
todoコンテキストを読み込む:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init todos)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから抽出: `commit_docs`, `date`, `timestamp`, `todo_count`, `todos`, `pending_dir`, `todos_dir_exists`。

ディレクトリの存在を確認:
```bash
mkdir -p .planning/todos/pending .planning/todos/done
```

infer_areaステップでの一貫性のために、todos配列から既存のエリアをメモ。
</step>

<step name="extract_content">
**引数ありの場合:** タイトル/焦点として使用。
- `/gsd:add-todo Add auth token refresh` → title = "Add auth token refresh"

**引数なしの場合:** 最近の会話を分析して以下を抽出:
- 議論された具体的な問題、アイデア、またはタスク
- 言及された関連ファイルパス
- 技術的な詳細（エラーメッセージ、行番号、制約）

策定:
- `title`: 3-10語の説明的なタイトル（動詞で始めることを推奨）
- `problem`: 何が問題か、なぜこれが必要か
- `solution`: アプローチのヒント、またはアイデア段階なら"TBD"
- `files`: 会話からの行番号付き関連パス
</step>

<step name="infer_area">
ファイルパスからエリアを推定:

| パスパターン | エリア |
|--------------|------|
| `src/api/*`, `api/*` | `api` |
| `src/components/*`, `src/ui/*` | `ui` |
| `src/auth/*`, `auth/*` | `auth` |
| `src/db/*`, `database/*` | `database` |
| `tests/*`, `__tests__/*` | `testing` |
| `docs/*` | `docs` |
| `.planning/*` | `planning` |
| `scripts/*`, `bin/*` | `tooling` |
| ファイルなしまたは不明 | `general` |

類似する一致がある場合はステップ2の既存エリアを使用。
</step>

<step name="check_duplicates">
```bash
# タイトルのキーワードを既存のtodoで検索
grep -l -i "[タイトルからのキーワード]" .planning/todos/pending/*.md 2>/dev/null
```

重複の可能性がある場合:
1. 既存のtodoを読み取る
2. スコープを比較

重複している場合、AskUserQuestionを使用:
- header: "重複？"
- question: "類似するtodoが存在します: [title]。どうしますか？"
- options:
  - "スキップ" — 既存のtodoを維持
  - "置き換え" — 新しいコンテキストで既存を更新
  - "そのまま追加" — 別のtodoとして作成
</step>

<step name="create_file">
initコンテキストの値を使用: `timestamp`と`date`は既に利用可能。

タイトルのスラグを生成:
```bash
slug=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" generate-slug "$title" --raw)
```

`.planning/todos/pending/${date}-${slug}.md`に書き込む:

```markdown
---
created: [timestamp]
title: [title]
area: [area]
files:
  - [file:lines]
---

## Problem

[問題の説明 - 数週間後の将来のClaudeが理解できるだけの十分なコンテキスト]

## Solution

[アプローチのヒントまたは"TBD"]
```
</step>

<step name="update_state">
`.planning/STATE.md`が存在する場合:

1. initコンテキストの`todo_count`を使用（またはカウントが変わった場合は`init todos`を再実行）
2. "## Accumulated Context"の下の"### Pending Todos"を更新
</step>

<step name="git_commit">
todoと更新された状態をコミット:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: capture todo - [title]" --files .planning/todos/pending/[filename] .planning/STATE.md
```

ツールは`commit_docs`設定とgitignoreを自動的に尊重する。

確認: "Committed: docs: capture todo - [title]"
</step>

<step name="confirm">
```
Todo保存済み: .planning/todos/pending/[filename]

  [title]
  エリア: [area]
  ファイル: [count]件の参照

---

次に何をしますか:

1. 現在の作業を続行
2. 別のtodoを追加
3. すべてのtodoを表示 (/gsd:check-todos)
```
</step>

</process>

<success_criteria>
- [ ] ディレクトリ構造が存在
- [ ] 有効なフロントマターを持つtodoファイルが作成された
- [ ] Problemセクションに将来のClaudeのための十分なコンテキストがある
- [ ] 重複なし（確認済みかつ解決済み）
- [ ] エリアが既存のtodoと一貫している
- [ ] STATE.mdが存在する場合は更新
- [ ] todoと状態がgitにコミットされた
</success_criteria>
</output>
