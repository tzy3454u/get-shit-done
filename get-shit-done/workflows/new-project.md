<purpose>
統合フローを通じて新しいプロジェクトを初期化する：質問、リサーチ（任意）、要件定義、ロードマップ。これはプロジェクトにおける最も重要な瞬間であり、ここでの深い質問がより良い計画、より良い実行、より良い成果につながる。1つのワークフローでアイデアから計画準備完了まで導く。
</purpose>

<required_reading>
開始前に、呼び出し元プロンプトのexecution_contextで参照されているすべてのファイルを読み込むこと。
</required_reading>

<auto_mode>
## 自動モード検出

$ARGUMENTSに`--auto`フラグが存在するか確認する。

**自動モードの場合：**
- ブラウンフィールドマッピングの提案をスキップ（グリーンフィールドを想定）
- 深い質問をスキップ（提供されたドキュメントからコンテキストを抽出）
- 設定：YOLOモードは暗黙的（その質問はスキップ）、ただし粒度/git/エージェントを最初に確認（ステップ2a）
- 設定後：ステップ6-9をスマートデフォルトで自動実行：
  - リサーチ：常にはい
  - 要件：提供されたドキュメントからすべてのテーブルステークス＋機能を含む
  - 要件承認：自動承認
  - ロードマップ承認：自動承認

**ドキュメント要件：**
自動モードにはアイデアドキュメントが必要 — 以下のいずれか：
- ファイル参照：`/gsd:new-project --auto @prd.md`
- プロンプトに貼り付けまたは記述されたテキスト

ドキュメントコンテンツが提供されていない場合、エラー：

```
Error: --auto requires an idea document.

Usage:
  /gsd:new-project --auto @your-idea.md
  /gsd:new-project --auto [paste or write your idea here]

The document should describe what you want to build.
```
</auto_mode>

<process>

## 1. セットアップ

**必須の最初のステップ — ユーザーとのやり取りの前にこれらのチェックを実行すること：**

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init new-project)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

JSONをパース：`researcher_model`, `synthesizer_model`, `roadmapper_model`, `commit_docs`, `project_exists`, `has_codebase_map`, `planning_exists`, `has_existing_code`, `has_package_file`, `is_brownfield`, `needs_codebase_map`, `has_git`, `project_path`。

**`project_exists`がtrueの場合：** エラー — プロジェクトは既に初期化済み。`/gsd:progress`を使用すること。

**`has_git`がfalseの場合：** gitを初期化：
```bash
git init
```

## 2. ブラウンフィールド提案

**自動モードの場合：** ステップ4にスキップ（グリーンフィールドを想定し、提供されたドキュメントからPROJECT.mdを合成）。

**`needs_codebase_map`がtrueの場合**（initから — 既存のコードが検出されたがコードベースマップがない）：

AskUserQuestionを使用：
- header: "Codebase"
- question: "このディレクトリに既存のコードを検出しました。先にコードベースをマッピングしますか？"
- options:
  - "Map codebase first" — 既存のアーキテクチャを理解するために/gsd:map-codebaseを実行（推奨）
  - "Skip mapping" — プロジェクト初期化を続行

**"Map codebase first"の場合：**
```
先に`/gsd:map-codebase`を実行し、その後`/gsd:new-project`に戻る
```
コマンドを終了。

**"Skip mapping"の場合 または `needs_codebase_map`がfalseの場合：** ステップ3へ続行。

## 2a. 自動モード設定（自動モードのみ）

**自動モードの場合：** アイデアドキュメントの処理前に設定を事前に収集する。

YOLOモードは暗黙的（auto = YOLO）。残りの設定質問を確認：

**ラウンド1 — コア設定（3つの質問、モード質問なし）：**

```
AskUserQuestion([
  {
    header: "Granularity",
    question: "スコープをどの程度細かくフェーズに分割しますか？",
    multiSelect: false,
    options: [
      { label: "Coarse (Recommended)", description: "少数の広いフェーズ（3-5フェーズ、各1-3プラン）" },
      { label: "Standard", description: "バランスの取れたフェーズサイズ（5-8フェーズ、各3-5プラン）" },
      { label: "Fine", description: "多数の集中フェーズ（8-12フェーズ、各5-10プラン）" }
    ]
  },
  {
    header: "Execution",
    question: "プランを並列実行しますか？",
    multiSelect: false,
    options: [
      { label: "Parallel (Recommended)", description: "独立したプランを同時に実行" },
      { label: "Sequential", description: "一度に1つのプラン" }
    ]
  },
  {
    header: "Git Tracking",
    question: "計画ドキュメントをgitにコミットしますか？",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "計画ドキュメントをバージョン管理で追跡" },
      { label: "No", description: ".planning/をローカル専用に保持（.gitignoreに追加）" }
    ]
  }
])
```

**ラウンド2 — ワークフローエージェント（ステップ5と同じ）：**

```
AskUserQuestion([
  {
    header: "Research",
    question: "各フェーズの計画前にリサーチしますか？（トークン/時間が追加されます）",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "ドメインを調査し、パターンを発見し、落とし穴を浮き彫りにする" },
      { label: "No", description: "要件から直接計画する" }
    ]
  },
  {
    header: "Plan Check",
    question: "プランが目標を達成するか検証しますか？（トークン/時間が追加されます）",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "実行開始前にギャップを検出" },
      { label: "No", description: "検証なしでプランを実行" }
    ]
  },
  {
    header: "Verifier",
    question: "各フェーズ後に作業が要件を満たしているか検証しますか？（トークン/時間が追加されます）",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "成果物がフェーズ目標に合致していることを確認" },
      { label: "No", description: "実行を信頼し、検証をスキップ" }
    ]
  },
  {
    header: "AI Models",
    question: "計画エージェントにどのAIモデルを使用しますか？",
    multiSelect: false,
    options: [
      { label: "Balanced (Recommended)", description: "ほとんどのエージェントにSonnet — 品質/コストのバランスが良い" },
      { label: "Quality", description: "リサーチ/ロードマップにOpus — 高コスト、より深い分析" },
      { label: "Budget", description: "可能な場合Haiku — 最速、最低コスト" }
    ]
  }
])
```

`.planning/config.json`を作成し、modeを"yolo"に設定：

```json
{
  "mode": "yolo",
  "granularity": "[selected]",
  "parallelization": true|false,
  "commit_docs": true|false,
  "model_profile": "quality|balanced|budget",
  "workflow": {
    "research": true|false,
    "plan_check": true|false,
    "verifier": true|false,
    "nyquist_validation": depth !== "quick",
    "auto_advance": true
  }
}
```

**commit_docs = Noの場合：** `.planning/`を`.gitignore`に追加。

**config.jsonをコミット：**

```bash
mkdir -p .planning
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "chore: add project config" --files .planning/config.json
```

**auto-advanceチェーンフラグを設定に永続化（コンテキスト圧縮を生き残る）：**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active true
```

ステップ4に進む（ステップ3と5をスキップ）。

## 3. 深い質問

**自動モードの場合：** スキップ（ステップ2aで既に処理済み）。代わりに提供されたドキュメントからプロジェクトコンテキストを抽出し、ステップ4に進む。

**ステージバナーを表示：**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► QUESTIONING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**会話を開始する：**

インラインで質問（自由形式、AskUserQuestionではない）：

「何を作りたいですか？」

応答を待つ。これにより、知的なフォローアップ質問をするために必要なコンテキストが得られる。

**スレッドを追う：**

相手が言ったことに基づいて、応答を深掘りするフォローアップ質問をする。AskUserQuestionを使用して、言及された内容を調査するオプションを提示する — 解釈、明確化、具体例。

スレッドを追い続ける。各回答が探求すべき新しいスレッドを開く。以下について質問する：
- 何に興奮しているか
- どんな問題がきっかけか
- 曖昧な用語の意味
- 実際にどのように見えるか
- 既に決まっていること

テクニックについては`questioning.md`を参照：
- 曖昧さに挑戦する
- 抽象的なものを具体的にする
- 前提を浮き彫りにする
- 境界を見つける
- 動機を明らかにする

**コンテキストの確認（バックグラウンドで、声に出さない）：**

進行中に、`questioning.md`のコンテキストチェックリストを頭の中で確認する。ギャップが残っている場合、自然に質問を織り込む。突然チェックリストモードに切り替えないこと。

**判断ゲート：**

明確なPROJECT.mdを書けると判断したら、AskUserQuestionを使用：

- header: "Ready?"
- question: "あなたが目指しているものを理解できたと思います。PROJECT.mdを作成しますか？"
- options:
  - "Create PROJECT.md" — 先に進みましょう
  - "Keep exploring" — もっと共有したい / もっと質問して

"Keep exploring"の場合 — 何を追加したいか聞くか、ギャップを特定して自然に深掘りする。

"Create PROJECT.md"が選択されるまでループ。

## 4. PROJECT.mdの作成

**自動モードの場合：** 提供されたドキュメントから合成する。"Ready?"ゲートは表示されなかった — 直接コミットに進む。

すべてのコンテキストを`templates/project.md`のテンプレートを使用して`.planning/PROJECT.md`に合成する。

**グリーンフィールドプロジェクトの場合：**

要件を仮説として初期化：

```markdown
## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] [Requirement 1]
- [ ] [Requirement 2]
- [ ] [Requirement 3]

### Out of Scope

- [Exclusion 1] — [why]
- [Exclusion 2] — [why]
```

すべてのActive要件は、出荷して検証されるまで仮説である。

**ブラウンフィールドプロジェクトの場合（コードベースマップが存在する）：**

既存のコードからValidated要件を推論：

1. `.planning/codebase/ARCHITECTURE.md`と`STACK.md`を読む
2. コードベースが既に行っていることを特定する
3. これらが初期のValidatedセットになる

```markdown
## Requirements

### Validated

- ✓ [Existing capability 1] — existing
- ✓ [Existing capability 2] — existing
- ✓ [Existing capability 3] — existing

### Active

- [ ] [New requirement 1]
- [ ] [New requirement 2]

### Out of Scope

- [Exclusion 1] — [why]
```

**Key Decisions：**

質問中に行われた決定で初期化：

```markdown
## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| [Choice from questioning] | [Why] | — Pending |
```

**最終更新フッター：**

```markdown
---
*Last updated: [date] after initialization*
```

圧縮しないこと。収集されたすべてを記録する。

**PROJECT.mdをコミット：**

```bash
mkdir -p .planning
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: initialize project" --files .planning/PROJECT.md
```

## 5. ワークフロー設定

**自動モードの場合：** スキップ — 設定はステップ2aで収集済み。ステップ5.5に進む。

`~/.gsd/defaults.json`で**グローバルデフォルトを確認**する。ファイルが存在する場合、保存されたデフォルトの使用を提案：

```
AskUserQuestion([
  {
    question: "保存されたデフォルト設定を使用しますか？（~/.gsd/defaults.jsonから）",
    header: "Defaults",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "保存されたデフォルトを使用し、設定質問をスキップ" },
      { label: "No", description: "手動で設定を構成" }
    ]
  }
])
```

"Yes"の場合：`~/.gsd/defaults.json`を読み、その値をconfig.jsonに使用し、以下の**config.jsonをコミット**に直接スキップ。

"No"の場合 または `~/.gsd/defaults.json`が存在しない場合：以下の質問に進む。

**ラウンド1 — コアワークフロー設定（4つの質問）：**

```
questions: [
  {
    header: "Mode",
    question: "どのように作業しますか？",
    multiSelect: false,
    options: [
      { label: "YOLO (Recommended)", description: "自動承認、実行するだけ" },
      { label: "Interactive", description: "各ステップで確認" }
    ]
  },
  {
    header: "Granularity",
    question: "スコープをどの程度細かくフェーズに分割しますか？",
    multiSelect: false,
    options: [
      { label: "Coarse", description: "少数の広いフェーズ（3-5フェーズ、各1-3プラン）" },
      { label: "Standard", description: "バランスの取れたフェーズサイズ（5-8フェーズ、各3-5プラン）" },
      { label: "Fine", description: "多数の集中フェーズ（8-12フェーズ、各5-10プラン）" }
    ]
  },
  {
    header: "Execution",
    question: "プランを並列実行しますか？",
    multiSelect: false,
    options: [
      { label: "Parallel (Recommended)", description: "独立したプランを同時に実行" },
      { label: "Sequential", description: "一度に1つのプラン" }
    ]
  },
  {
    header: "Git Tracking",
    question: "計画ドキュメントをgitにコミットしますか？",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "計画ドキュメントをバージョン管理で追跡" },
      { label: "No", description: ".planning/をローカル専用に保持（.gitignoreに追加）" }
    ]
  }
]
```

**ラウンド2 — ワークフローエージェント：**

これらは計画/実行中に追加のエージェントを生成する。トークンと時間が追加されるが、品質が向上する。

| エージェント | 実行タイミング | 機能 |
|-------|--------------|--------------|
| **Researcher** | 各フェーズの計画前 | ドメインを調査し、パターンを発見し、落とし穴を浮き彫りにする |
| **Plan Checker** | プラン作成後 | プランが実際にフェーズ目標を達成するか検証する |
| **Verifier** | フェーズ実行後 | 必須項目が納品されたことを確認する |

重要なプロジェクトにはすべて推奨。簡単な実験にはスキップ可能。

```
questions: [
  {
    header: "Research",
    question: "各フェーズの計画前にリサーチしますか？（トークン/時間が追加されます）",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "ドメインを調査し、パターンを発見し、落とし穴を浮き彫りにする" },
      { label: "No", description: "要件から直接計画する" }
    ]
  },
  {
    header: "Plan Check",
    question: "プランが目標を達成するか検証しますか？（トークン/時間が追加されます）",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "実行開始前にギャップを検出" },
      { label: "No", description: "検証なしでプランを実行" }
    ]
  },
  {
    header: "Verifier",
    question: "各フェーズ後に作業が要件を満たしているか検証しますか？（トークン/時間が追加されます）",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "成果物がフェーズ目標に合致していることを確認" },
      { label: "No", description: "実行を信頼し、検証をスキップ" }
    ]
  },
  {
    header: "AI Models",
    question: "計画エージェントにどのAIモデルを使用しますか？",
    multiSelect: false,
    options: [
      { label: "Balanced (Recommended)", description: "ほとんどのエージェントにSonnet — 品質/コストのバランスが良い" },
      { label: "Quality", description: "リサーチ/ロードマップにOpus — 高コスト、より深い分析" },
      { label: "Budget", description: "可能な場合Haiku — 最速、最低コスト" }
    ]
  }
]
```

すべての設定で`.planning/config.json`を作成：

```json
{
  "mode": "yolo|interactive",
  "granularity": "coarse|standard|fine",
  "parallelization": true|false,
  "commit_docs": true|false,
  "model_profile": "quality|balanced|budget",
  "workflow": {
    "research": true|false,
    "plan_check": true|false,
    "verifier": true|false,
    "nyquist_validation": depth !== "quick"
  }
}
```

**commit_docs = Noの場合：**
- config.jsonに`commit_docs: false`を設定
- `.planning/`を`.gitignore`に追加（必要に応じて作成）

**commit_docs = Yesの場合：**
- 追加のgitignoreエントリは不要

**config.jsonをコミット：**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "chore: add project config" --files .planning/config.json
```

**注意：** いつでも`/gsd:settings`を実行してこれらの設定を更新可能。

## 5.5. モデルプロファイルの解決

initからのモデルを使用：`researcher_model`, `synthesizer_model`, `roadmapper_model`。

## 6. リサーチの判断

**自動モードの場合：** 質問せずに"Research first"をデフォルトにする。

AskUserQuestionを使用：
- header: "Research"
- question: "要件定義の前にドメインエコシステムをリサーチしますか？"
- options:
  - "Research first (Recommended)" — 標準スタック、期待される機能、アーキテクチャパターンを発見する
  - "Skip research" — このドメインに詳しいので、直接要件定義に進む

**"Research first"の場合：**

ステージバナーを表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► RESEARCHING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[domain]エコシステムをリサーチ中...
```

リサーチディレクトリを作成：
```bash
mkdir -p .planning/research
```

**マイルストーンコンテキストの決定：**

グリーンフィールドか後続マイルストーンかを確認：
- PROJECT.mdに"Validated"要件がない場合 → グリーンフィールド（ゼロから構築）
- "Validated"要件が存在する場合 → 後続マイルストーン（既存アプリに追加）

生成インジケーターを表示：
```
◆ 4つのリサーチャーを並列で生成中...
  → スタックリサーチ
  → 機能リサーチ
  → アーキテクチャリサーチ
  → 落とし穴リサーチ
```

パス参照付きで4つの並列gsd-project-researcherエージェントを生成：

```
Task(prompt="<research_type>
プロジェクトリサーチ — [domain]のスタック次元。
</research_type>

<milestone_context>
[greenfield OR subsequent]

グリーンフィールド：[domain]をゼロから構築するための標準スタックをリサーチする。
後続：既存の[domain]アプリに[target features]を追加するために必要なものをリサーチする。既存システムの再リサーチは不要。
</milestone_context>

<question>
[domain]の2025年標準スタックは何ですか？
</question>

<files_to_read>
- {project_path} (プロジェクトのコンテキストと目標)
</files_to_read>

<downstream_consumer>
あなたのSTACK.mdはロードマップ作成に供給されます。具体的に記述してください：
- バージョン付きの特定のライブラリ
- 各選択の明確な根拠
- 使用すべきでないものとその理由
</downstream_consumer>

<quality_gate>
- [ ] バージョンが最新であること（トレーニングデータではなくContext7/公式ドキュメントで検証）
- [ ] 根拠がWHATではなくWHYを説明していること
- [ ] 各推奨に信頼度レベルが割り当てられていること
</quality_gate>

<output>
Write to: .planning/research/STACK.md
Use template: ~/.claude/get-shit-done/templates/research-project/STACK.md
</output>
", subagent_type="gsd-project-researcher", model="{researcher_model}", description="Stack research")

Task(prompt="<research_type>
プロジェクトリサーチ — [domain]の機能次元。
</research_type>

<milestone_context>
[greenfield OR subsequent]

グリーンフィールド：[domain]製品にはどのような機能がありますか？テーブルステークスと差別化要因は何ですか？
後続：[target features]は通常どのように動作しますか？期待される動作は何ですか？
</milestone_context>

<question>
[domain]製品にはどのような機能がありますか？テーブルステークスと差別化要因は何ですか？
</question>

<files_to_read>
- {project_path} (プロジェクトコンテキスト)
</files_to_read>

<downstream_consumer>
あなたのFEATURES.mdは要件定義に供給されます。明確にカテゴリ分けしてください：
- テーブルステークス（なければユーザーが離れる必須機能）
- 差別化要因（競争優位性）
- アンチフィーチャー（意図的に作らないもの）
</downstream_consumer>

<quality_gate>
- [ ] カテゴリが明確であること（テーブルステークス vs 差別化要因 vs アンチフィーチャー）
- [ ] 各機能の複雑さが記載されていること
- [ ] 機能間の依存関係が特定されていること
</quality_gate>

<output>
Write to: .planning/research/FEATURES.md
Use template: ~/.claude/get-shit-done/templates/research-project/FEATURES.md
</output>
", subagent_type="gsd-project-researcher", model="{researcher_model}", description="Features research")

Task(prompt="<research_type>
プロジェクトリサーチ — [domain]のアーキテクチャ次元。
</research_type>

<milestone_context>
[greenfield OR subsequent]

グリーンフィールド：[domain]システムは通常どのように構成されますか？主要コンポーネントは何ですか？
後続：[target features]は既存の[domain]アーキテクチャにどのように統合されますか？
</milestone_context>

<question>
[domain]システムは通常どのように構成されますか？主要コンポーネントは何ですか？
</question>

<files_to_read>
- {project_path} (プロジェクトコンテキスト)
</files_to_read>

<downstream_consumer>
あなたのARCHITECTURE.mdはロードマップのフェーズ構造に影響します。以下を含めてください：
- コンポーネント境界（何が何と通信するか）
- データフロー（情報がどのように移動するか）
- 推奨構築順序（コンポーネント間の依存関係）
</downstream_consumer>

<quality_gate>
- [ ] コンポーネントが境界付きで明確に定義されていること
- [ ] データフローの方向が明示的であること
- [ ] 構築順序の影響が記載されていること
</quality_gate>

<output>
Write to: .planning/research/ARCHITECTURE.md
Use template: ~/.claude/get-shit-done/templates/research-project/ARCHITECTURE.md
</output>
", subagent_type="gsd-project-researcher", model="{researcher_model}", description="Architecture research")

Task(prompt="<research_type>
プロジェクトリサーチ — [domain]の落とし穴次元。
</research_type>

<milestone_context>
[greenfield OR subsequent]

グリーンフィールド：[domain]プロジェクトがよく間違えることは何ですか？致命的なミスは？
後続：[domain]に[target features]を追加する際の一般的なミスは何ですか？
</milestone_context>

<question>
[domain]プロジェクトがよく間違えることは何ですか？致命的なミスは？
</question>

<files_to_read>
- {project_path} (プロジェクトコンテキスト)
</files_to_read>

<downstream_consumer>
あなたのPITFALLS.mdはロードマップ/計画でのミスを防ぎます。各落とし穴について：
- 警告サイン（早期に検出する方法）
- 予防戦略（回避する方法）
- どのフェーズが対処すべきか
</downstream_consumer>

<quality_gate>
- [ ] 落とし穴がこのドメインに固有であること（一般的なアドバイスではない）
- [ ] 予防戦略が実行可能であること
- [ ] 関連する場合にフェーズマッピングが含まれていること
</quality_gate>

<output>
Write to: .planning/research/PITFALLS.md
Use template: ~/.claude/get-shit-done/templates/research-project/PITFALLS.md
</output>
", subagent_type="gsd-project-researcher", model="{researcher_model}", description="Pitfalls research")
```

4つのエージェントがすべて完了した後、シンセサイザーを生成してSUMMARY.mdを作成：

```
Task(prompt="
<task>
リサーチ出力をSUMMARY.mdに合成する。
</task>

<files_to_read>
- .planning/research/STACK.md
- .planning/research/FEATURES.md
- .planning/research/ARCHITECTURE.md
- .planning/research/PITFALLS.md
</files_to_read>

<output>
Write to: .planning/research/SUMMARY.md
Use template: ~/.claude/get-shit-done/templates/research-project/SUMMARY.md
書き込み後にコミットすること。
</output>
", subagent_type="gsd-research-synthesizer", model="{synthesizer_model}", description="Synthesize research")
```

リサーチ完了バナーと主要な発見を表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► RESEARCH COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 主要な発見

**スタック:** [SUMMARY.mdより]
**テーブルステークス:** [SUMMARY.mdより]
**注意事項:** [SUMMARY.mdより]

ファイル: `.planning/research/`
```

**"Skip research"の場合：** ステップ7に続行。

## 7. 要件定義

ステージバナーを表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DEFINING REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**コンテキストの読み込み：**

PROJECT.mdを読み、以下を抽出：
- コアバリュー（動作しなければならないたった1つのこと）
- 明示された制約（予算、タイムライン、技術的制限）
- 明示的なスコープ境界

**リサーチが存在する場合：** research/FEATURES.mdを読み、機能カテゴリを抽出。

**自動モードの場合：**
- すべてのテーブルステークス機能を自動的に含める（ユーザーが期待するもの）
- 提供されたドキュメントで明示的に言及された機能を含める
- ドキュメントで言及されていない差別化要因を自動的に延期
- カテゴリごとのAskUserQuestionループをスキップ
- 「追加はありますか？」の質問をスキップ
- 要件承認ゲートをスキップ
- REQUIREMENTS.mdを生成して直接コミット

**カテゴリ別に機能を提示（インタラクティブモードのみ）：**

```
[domain]の機能は以下の通りです：

## 認証
**テーブルステークス：**
- メール/パスワードでのサインアップ
- メール認証
- パスワードリセット
- セッション管理

**差別化要因：**
- マジックリンクログイン
- OAuth（Google、GitHub）
- 2FA

**リサーチノート:** [関連するメモ]

---

## [次のカテゴリ]
...
```

**リサーチがない場合：** 代わりに会話を通じて要件を収集する。

質問：「ユーザーができる必要がある主な操作は何ですか？」

言及された各機能について：
- 具体的にするための明確化質問
- 関連する機能の調査
- カテゴリへのグループ化

**各カテゴリのスコーピング：**

各カテゴリについてAskUserQuestionを使用：

- header: "[Category]" (最大12文字)
- question: "v1にどの[category]機能を含めますか？"
- multiSelect: true
- options:
  - "[Feature 1]" — [簡単な説明]
  - "[Feature 2]" — [簡単な説明]
  - "[Feature 3]" — [簡単な説明]
  - "None for v1" — カテゴリ全体を延期

応答を追跡：
- 選択された機能 → v1要件
- 選択されなかったテーブルステークス → v2（ユーザーが期待するもの）
- 選択されなかった差別化要因 → スコープ外

**ギャップの特定：**

AskUserQuestionを使用：
- header: "Additions"
- question: "リサーチが見逃した要件はありますか？（あなたのビジョンに固有の機能）"
- options:
  - "No, research covered it" — 続行
  - "Yes, let me add some" — 追加を記録

**コアバリューの検証：**

PROJECT.mdのコアバリューに対して要件をクロスチェック。ギャップが検出された場合、浮き彫りにする。

**REQUIREMENTS.mdの生成：**

`.planning/REQUIREMENTS.md`を以下で作成：
- カテゴリ別にグループ化されたv1要件（チェックボックス、REQ-ID）
- v2要件（延期）
- スコープ外（理由付きの明示的な除外）
- トレーサビリティセクション（空、ロードマップで記入）

**REQ-IDフォーマット:** `[CATEGORY]-[NUMBER]` (AUTH-01, CONTENT-02)

**要件品質基準：**

良い要件とは：
- **具体的でテスト可能：** 「ユーザーはメールリンクでパスワードをリセットできる」（「パスワードリセットを処理する」ではない）
- **ユーザー中心：** 「ユーザーはXできる」（「システムがYする」ではない）
- **原子的：** 1要件につき1つの機能（「ユーザーはログインしてプロファイルを管理できる」ではない）
- **独立的：** 他の要件への依存が最小

曖昧な要件は拒否する。具体性を求める：
- 「認証を処理する」→「ユーザーはメール/パスワードでログインし、セッション間でログイン状態を維持できる」
- 「共有をサポートする」→「ユーザーは受信者のブラウザで開くリンクで投稿を共有できる」

**完全な要件リストを提示（インタラクティブモードのみ）：**

ユーザー確認のためにすべての要件を表示（カウントではない）：

```
## v1 Requirements

### Authentication
- [ ] **AUTH-01**: ユーザーはメール/パスワードでアカウントを作成できる
- [ ] **AUTH-02**: ユーザーはログインし、セッション間でログイン状態を維持できる
- [ ] **AUTH-03**: ユーザーはどのページからでもログアウトできる

### Content
- [ ] **CONT-01**: ユーザーはテキストで投稿を作成できる
- [ ] **CONT-02**: ユーザーは自分の投稿を編集できる

[... 完全なリスト ...]

---

これはあなたが構築しているものを捉えていますか？ (yes / adjust)
```

"adjust"の場合：スコーピングに戻る。

**要件をコミット：**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: define v1 requirements" --files .planning/REQUIREMENTS.md
```

## 8. ロードマップの作成

ステージバナーを表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CREATING ROADMAP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ ロードマッパーを生成中...
```

パス参照付きでgsd-roadmapperエージェントを生成：

```
Task(prompt="
<planning_context>

<files_to_read>
- .planning/PROJECT.md (プロジェクトコンテキスト)
- .planning/REQUIREMENTS.md (v1要件)
- .planning/research/SUMMARY.md (リサーチ結果 - 存在する場合)
- .planning/config.json (粒度とモード設定)
</files_to_read>

</planning_context>

<instructions>
ロードマップを作成：
1. 要件からフェーズを導出する（構造を押し付けない）
2. すべてのv1要件を正確に1つのフェーズにマッピング
3. フェーズごとに2-5の成功基準を導出（観察可能なユーザー行動）
4. 100%カバレッジを検証
5. ファイルを直ちに書き込む（ROADMAP.md、STATE.md、REQUIREMENTS.mdのトレーサビリティを更新）
6. ROADMAP CREATEDとサマリーを返す

先にファイルを書き込み、その後返す。これにより、コンテキストが失われてもアーティファクトが永続化される。
</instructions>
", subagent_type="gsd-roadmapper", model="{roadmapper_model}", description="Create roadmap")
```

**ロードマッパーの戻り値を処理：**

**`## ROADMAP BLOCKED`の場合：**
- ブロッカー情報を提示
- ユーザーと解決に取り組む
- 解決後に再生成

**`## ROADMAP CREATED`の場合：**

作成されたROADMAP.mdを読み、インラインできれいに提示：

```
---

## 提案されたロードマップ

**[N]フェーズ** | **[X]要件マッピング済み** | すべてのv1要件カバー ✓

| # | フェーズ | 目標 | 要件 | 成功基準 |
|---|-------|------|--------------|------------------|
| 1 | [Name] | [Goal] | [REQ-IDs] | [count] |
| 2 | [Name] | [Goal] | [REQ-IDs] | [count] |
| 3 | [Name] | [Goal] | [REQ-IDs] | [count] |
...

### フェーズ詳細

**Phase 1: [Name]**
目標: [goal]
要件: [REQ-IDs]
成功基準:
1. [criterion]
2. [criterion]
3. [criterion]

**Phase 2: [Name]**
目標: [goal]
要件: [REQ-IDs]
成功基準:
1. [criterion]
2. [criterion]

[... すべてのフェーズについて続行 ...]

---
```

**自動モードの場合：** 承認ゲートをスキップ — 自動承認して直接コミット。

**重要：コミット前に承認を求める（インタラクティブモードのみ）：**

AskUserQuestionを使用：
- header: "Roadmap"
- question: "このロードマップ構造でよいですか？"
- options:
  - "Approve" — コミットして続行
  - "Adjust phases" — 変更点を教えてください
  - "Review full file" — 生のROADMAP.mdを表示

**"Approve"の場合：** コミットに続行。

**"Adjust phases"の場合：**
- ユーザーの調整メモを取得
- 修正コンテキスト付きでロードマッパーを再生成：
  ```
  Task(prompt="
  <revision>
  ロードマップに対するユーザーのフィードバック：
  [user's notes]

  <files_to_read>
  - .planning/ROADMAP.md (修正する現在のロードマップ)
  </files_to_read>

  フィードバックに基づいてロードマップを更新。ファイルをその場で編集。
  ROADMAP REVISEDと行った変更を返す。
  </revision>
  ", subagent_type="gsd-roadmapper", model="{roadmapper_model}", description="Revise roadmap")
  ```
- 修正されたロードマップを提示
- ユーザーが承認するまでループ

**"Review full file"の場合：** 生の`cat .planning/ROADMAP.md`を表示し、再度質問。

**ロードマップをコミット（承認後または自動モード）：**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: create roadmap ([N] phases)" --files .planning/ROADMAP.md .planning/STATE.md .planning/REQUIREMENTS.md
```

## 9. 完了

完了サマリーを提示：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PROJECT INITIALIZED ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**[Project Name]**

| アーティファクト   | 場所                        |
|----------------|-----------------------------|
| Project        | `.planning/PROJECT.md`      |
| Config         | `.planning/config.json`     |
| Research       | `.planning/research/`       |
| Requirements   | `.planning/REQUIREMENTS.md` |
| Roadmap        | `.planning/ROADMAP.md`      |

**[N]フェーズ** | **[X]要件** | 構築準備完了 ✓
```

**自動モードの場合：**

```
╔══════════════════════════════════════════╗
║  AUTO-ADVANCING → DISCUSS PHASE 1        ║
╚══════════════════════════════════════════╝
```

スキルを終了し、SlashCommand("/gsd:discuss-phase 1 --auto")を呼び出す

**インタラクティブモードの場合：**

```
───────────────────────────────────────────────────────────────

## ▶ Next Up

**Phase 1: [Phase Name]** — [ROADMAP.mdからの目標]

/gsd:discuss-phase 1 — コンテキストを収集しアプローチを明確にする

<sub>/clear first → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション：**
- /gsd:plan-phase 1 — ディスカッションをスキップして直接計画

───────────────────────────────────────────────────────────────
```

</process>

<output>

- `.planning/PROJECT.md`
- `.planning/config.json`
- `.planning/research/` (リサーチを選択した場合)
  - `STACK.md`
  - `FEATURES.md`
  - `ARCHITECTURE.md`
  - `PITFALLS.md`
  - `SUMMARY.md`
- `.planning/REQUIREMENTS.md`
- `.planning/ROADMAP.md`
- `.planning/STATE.md`

</output>

<success_criteria>

- [ ] .planning/ディレクトリが作成された
- [ ] Gitリポジトリが初期化された
- [ ] ブラウンフィールド検出が完了した
- [ ] 深い質問が完了した（スレッドが追跡され、急がれていない）
- [ ] PROJECT.mdが完全なコンテキストを記録 → **コミット済み**
- [ ] config.jsonにワークフローモード、粒度、並列化が含まれる → **コミット済み**
- [ ] リサーチが完了した（選択された場合）— 4つの並列エージェントが生成された → **コミット済み**
- [ ] 要件が収集された（リサーチまたは会話から）
- [ ] ユーザーが各カテゴリのスコープを設定した（v1/v2/スコープ外）
- [ ] REQ-ID付きのREQUIREMENTS.mdが作成された → **コミット済み**
- [ ] コンテキスト付きでgsd-roadmapperが生成された
- [ ] ロードマップファイルが直ちに書き込まれた（ドラフトではない）
- [ ] ユーザーのフィードバックが反映された（あれば）
- [ ] フェーズ、要件マッピング、成功基準付きのROADMAP.mdが作成された
- [ ] STATE.mdが初期化された
- [ ] REQUIREMENTS.mdのトレーサビリティが更新された
- [ ] ユーザーが次のステップは`/gsd:discuss-phase 1`であることを認識している

**アトミックコミット：** 各フェーズはアーティファクトを直ちにコミットする。コンテキストが失われても、アーティファクトは永続化される。

</success_criteria>
</output>