<purpose>
GSDワークフローエージェント（research、plan_check、verifier）のインタラクティブな設定と、複数質問プロンプトによるモデルプロファイルの選択。ユーザーの設定で.planning/config.jsonを更新する。オプションで設定をグローバルデフォルト（~/.gsd/defaults.json）として保存し、将来のプロジェクトに適用できる。
</purpose>

<required_reading>
開始前に、呼び出しプロンプトのexecution_contextで参照されているすべてのファイルを読み込むこと。
</required_reading>

<process>

<step name="ensure_and_load_config">
設定が存在することを確認し、現在の状態を読み込む：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-ensure-section
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

`.planning/config.json`が存在しない場合はデフォルトで作成し、現在の設定値を読み込む。
</step>

<step name="read_current">
```bash
cat .planning/config.json
```

現在の値を解析する（存在しない場合はデフォルトで`true`）：
- `workflow.research` — plan-phase中にリサーチャーを生成
- `workflow.plan_check` — plan-phase中にプランチェッカーを生成
- `workflow.verifier` — execute-phase中にベリファイアを生成
- `workflow.nyquist_validation` — plan-phase中のバリデーションアーキテクチャリサーチ（デフォルト: 存在しない場合はtrue）
- `model_profile` — 各エージェントが使用するモデル（デフォルト: `balanced`）
- `git.branching_strategy` — ブランチ戦略（デフォルト: `"none"`）
</step>

<step name="present_settings">
現在の値が事前選択されたAskUserQuestionを使用する：

```
AskUserQuestion([
  {
    question: "エージェントにどのモデルプロファイルを使用しますか？",
    header: "Model",
    multiSelect: false,
    options: [
      { label: "Quality", description: "検証以外すべてOpus（最高コスト）" },
      { label: "Balanced（推奨）", description: "計画にOpus、実行/検証にSonnet" },
      { label: "Budget", description: "執筆にSonnet、リサーチ/検証にHaiku（最低コスト）" }
    ]
  },
  {
    question: "プランリサーチャーを生成しますか？（計画前にドメインを調査）",
    header: "Research",
    multiSelect: false,
    options: [
      { label: "はい", description: "計画前にフェーズの目標を調査" },
      { label: "いいえ", description: "調査をスキップし、直接計画" }
    ]
  },
  {
    question: "プランチェッカーを生成しますか？（実行前にプランを検証）",
    header: "Plan Check",
    multiSelect: false,
    options: [
      { label: "はい", description: "プランがフェーズの目標を満たすか検証" },
      { label: "いいえ", description: "プラン検証をスキップ" }
    ]
  },
  {
    question: "実行ベリファイアを生成しますか？（フェーズ完了を検証）",
    header: "Verifier",
    multiSelect: false,
    options: [
      { label: "はい", description: "実行後に必須要件を検証" },
      { label: "いいえ", description: "実行後の検証をスキップ" }
    ]
  },
  {
    question: "パイプラインを自動進行しますか？（discuss → plan → execute を自動実行）",
    header: "Auto",
    multiSelect: false,
    options: [
      { label: "いいえ（推奨）", description: "ステージ間で手動 /clear + ペースト" },
      { label: "はい", description: "Task()サブエージェント経由でステージを連鎖（同じ分離）" }
    ]
  },
  {
    question: "Nyquistバリデーションを有効にしますか？（計画中にテストカバレッジを調査）",
    header: "Nyquist",
    multiSelect: false,
    options: [
      { label: "はい（推奨）", description: "plan-phase中に自動テストカバレッジを調査。プランにバリデーション要件を追加。タスクに自動検証がない場合、承認をブロック。" },
      { label: "いいえ", description: "バリデーション調査をスキップ。ラピッドプロトタイピングやテストなしフェーズに適しています。" }
    ]
  },
  // Note: Nyquist validation depends on research output. If research is disabled,
  // plan-phase automatically skips Nyquist steps (no RESEARCH.md to extract from).
  {
    question: "Gitブランチ戦略は？",
    header: "Branching",
    multiSelect: false,
    options: [
      { label: "なし（推奨）", description: "現在のブランチに直接コミット" },
      { label: "フェーズごと", description: "各フェーズにブランチを作成（gsd/phase-{N}-{name}）" },
      { label: "マイルストーンごと", description: "マイルストーン全体にブランチを作成（gsd/{version}-{name}）" }
    ]
  }
])
```
</step>

<step name="update_config">
新しい設定を既存のconfig.jsonにマージする：

```json
{
  ...existing_config,
  "model_profile": "quality" | "balanced" | "budget",
  "workflow": {
    "research": true/false,
    "plan_check": true/false,
    "verifier": true/false,
    "auto_advance": true/false,
    "nyquist_validation": true/false
  },
  "git": {
    "branching_strategy": "none" | "phase" | "milestone"
  }
}
```

更新された設定を`.planning/config.json`に書き込む。
</step>

<step name="save_as_defaults">
これらの設定を将来のプロジェクトのグローバルデフォルトとして保存するか確認する：

```
AskUserQuestion([
  {
    question: "これらをすべての新規プロジェクトのデフォルト設定として保存しますか？",
    header: "Defaults",
    multiSelect: false,
    options: [
      { label: "はい", description: "新規プロジェクトがこれらの設定で開始されます（~/.gsd/defaults.jsonに保存）" },
      { label: "いいえ", description: "このプロジェクトにのみ適用" }
    ]
  }
])
```

「はい」の場合：同じ設定オブジェクト（`brave_search`などのプロジェクト固有フィールドを除く）を`~/.gsd/defaults.json`に書き込む：

```bash
mkdir -p ~/.gsd
```

Write `~/.gsd/defaults.json` with:
```json
{
  "mode": <current>,
  "granularity": <current>,
  "model_profile": <current>,
  "commit_docs": <current>,
  "parallelization": <current>,
  "branching_strategy": <current>,
  "workflow": {
    "research": <current>,
    "plan_check": <current>,
    "verifier": <current>,
    "auto_advance": <current>,
    "nyquist_validation": <current>
  }
}
```
</step>

<step name="confirm">
表示：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► 設定更新完了
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| 設定                  | 値 |
|----------------------|-------|
| モデルプロファイル       | {quality/balanced/budget} |
| プランリサーチャー       | {On/Off} |
| プランチェッカー         | {On/Off} |
| 実行ベリファイア         | {On/Off} |
| 自動進行               | {On/Off} |
| Nyquistバリデーション    | {On/Off} |
| Gitブランチ            | {なし/フェーズごと/マイルストーンごと} |
| デフォルトとして保存     | {はい/いいえ} |

これらの設定は今後の /gsd:plan-phase と /gsd:execute-phase の実行に適用される。

クイックコマンド:
- /gsd:set-profile <profile> — モデルプロファイルを切り替え
- /gsd:plan-phase --research — リサーチを強制
- /gsd:plan-phase --skip-research — リサーチをスキップ
- /gsd:plan-phase --skip-verify — プランチェックをスキップ
```
</step>

</process>

<success_criteria>
- [ ] 現在の設定が読み込まれた
- [ ] ユーザーに7つの設定が提示された（プロファイル + 5つのワークフロートグル + gitブランチ）
- [ ] 設定がmodel_profile、workflow、gitセクションで更新された
- [ ] ユーザーにグローバルデフォルトとしての保存が提案された（~/.gsd/defaults.json）
- [ ] 変更がユーザーに確認された
</success_criteria>
