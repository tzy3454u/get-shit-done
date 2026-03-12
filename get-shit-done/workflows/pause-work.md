<purpose>
`.continue-here.md`ハンドオフファイルを作成し、セッション間で完全な作業状態を保存します。完全なコンテキスト復元によるシームレスな再開を可能にします。
</purpose>

<required_reading>
開始前に、呼び出しプロンプトのexecution_contextで参照されているすべてのファイルを読み込んでください。
</required_reading>

<process>

<step name="detect">
最近変更されたファイルから現在のフェーズディレクトリを見つけます：

```bash
# Find most recent phase directory with work
ls -lt .planning/phases/*/PLAN.md 2>/dev/null | head -1 | grep -oP 'phases/\K[^/]+'
```

アクティブなフェーズが検出されない場合、ユーザーにどのフェーズの作業を一時停止するか確認してください。
</step>

<step name="gather">
**ハンドオフのために完全な状態を収集します：**

1. **現在の位置**: どのフェーズ、どのプラン、どのタスク
2. **完了した作業**: このセッションで完了したこと
3. **残りの作業**: 現在のプラン/フェーズで残っていること
4. **行った決定**: 重要な決定とその理由
5. **ブロッカー/問題**: 行き詰まっていること
6. **メンタルコンテキスト**: アプローチ、次のステップ、「雰囲気」
7. **変更されたファイル**: コミットされていない変更

必要に応じて、対話的な質問でユーザーに確認してください。
</step>

<step name="write">
**`.planning/phases/XX-name/.continue-here.md`にハンドオフを書き込みます：**

```markdown
---
phase: XX-name
task: 3
total_tasks: 7
status: in_progress
last_updated: [timestamp from current-timestamp]
---

<current_state>
[現在の正確な位置は？直近のコンテキスト]
</current_state>

<completed_work>

- タスク 1: [名前] - 完了
- タスク 2: [名前] - 完了
- タスク 3: [名前] - 進行中、[完了した部分]
</completed_work>

<remaining_work>

- タスク 3: [残りの作業]
- タスク 4: 未着手
- タスク 5: 未着手
</remaining_work>

<decisions_made>

- [理由]のため[X]を使用することに決定
- [理由]のため[代替案]より[アプローチ]を選択
</decisions_made>

<blockers>
- [ブロッカー 1]: [状態/回避策]
</blockers>

<context>
[メンタルステート、考えていたこと、計画]
</context>

<next_action>
開始時: [再開時の最初の具体的なアクション]
</next_action>
```

新しいClaudeがすぐに理解できるよう、十分に具体的に記述してください。

last_updatedフィールドには`current-timestamp`を使用してください。init todos（タイムスタンプを提供）を使用するか、直接呼び出すことができます：
```bash
timestamp=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" current-timestamp full --raw)
```
</step>

<step name="commit">
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "wip: [phase-name] paused at task [X]/[Y]" --files .planning/phases/*/.continue-here.md
```
</step>

<step name="confirm">
```
✓ ハンドオフ作成完了: .planning/phases/[XX-name]/.continue-here.md

現在の状態:

- フェーズ: [XX-name]
- タスク: [X] / [Y]
- ステータス: [in_progress/blocked]
- WIPとしてコミット済み

再開するには: /gsd:resume-work

```
</step>

</process>

<success_criteria>
- [ ] 正しいフェーズディレクトリに.continue-here.mdが作成された
- [ ] すべてのセクションが具体的な内容で記入された
- [ ] WIPとしてコミットされた
- [ ] ユーザーが場所と再開方法を把握した
</success_criteria>
