# Continue-Hereテンプレート

この構造をコピーして `.planning/phases/XX-name/.continue-here.md` に記入してください:

```yaml
---
phase: XX-name
task: 3
total_tasks: 7
status: in_progress
last_updated: 2025-01-15T14:30:00Z
---
```

```markdown
<current_state>
[今どこにいるか？直近のコンテキストは？]
</current_state>

<completed_work>
[このセッションで完了した作業 - 具体的に]

- Task 1: [名前] - 完了
- Task 2: [名前] - 完了
- Task 3: [名前] - 進行中、[何が完了したか]
</completed_work>

<remaining_work>
[このフェーズの残りの作業]

- Task 3: [名前] - [残りの作業]
- Task 4: [名前] - 未着手
- Task 5: [名前] - 未着手
</remaining_work>

<decisions_made>
[主要な判断とその理由 — 次のセッションで再議論しないために]

- [X]を使用することに決定、理由は[理由]
- [代替案]より[アプローチ]を選択、理由は[理由]
</decisions_made>

<blockers>
[行き詰まっているもの、外部要因を待っているもの]

- [ブロッカー1]: [状況/回避策]
</blockers>

<context>
[心理状態、「雰囲気」、スムーズに再開するために役立つ情報]

[何を考えていたか？計画は何だったか？
これは「中断した場所からまさにそのまま再開する」ためのコンテキストです。]
</context>

<next_action>
[再開時に最初にすべきこと]

Start with: [具体的なアクション]
</next_action>
```

<yaml_fields>
必須のYAMLフロントマター:

- `phase`: ディレクトリ名（例: `02-authentication`）
- `task`: 現在のタスク番号
- `total_tasks`: フェーズ内のタスク数
- `status`: `in_progress`、`blocked`、`almost_done`
- `last_updated`: ISOタイムスタンプ
</yaml_fields>

<guidelines>
- 新しいClaudeインスタンスがすぐに理解できるほど具体的に書く
- 判断の理由（WHY）を含める、何を決めたかだけでなく
- `<next_action>`は他に何も読まなくても実行可能であるべき
- このファイルは再開後に削除される — 永続的な保存ではない
</guidelines>
</output>