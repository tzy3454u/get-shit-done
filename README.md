<div align="center">

# GET SHIT DONE

**Claude Code、OpenCode、Gemini CLI、Codex向けの、軽量かつ強力なメタプロンプティング / コンテキストエンジニアリング / 仕様駆動開発システム。**

**Claudeのコンテキストウィンドウが埋まるにつれて起きる品質劣化（context rot）を解決します。**

[![npm version](https://img.shields.io/npm/v/get-shit-done-cc?style=for-the-badge&logo=npm&logoColor=white&color=CB3837)](https://www.npmjs.com/package/get-shit-done-cc)
[![npm downloads](https://img.shields.io/npm/dm/get-shit-done-cc?style=for-the-badge&logo=npm&logoColor=white&color=CB3837)](https://www.npmjs.com/package/get-shit-done-cc)
[![Tests](https://img.shields.io/github/actions/workflow/status/glittercowboy/get-shit-done/test.yml?branch=main&style=for-the-badge&logo=github&label=Tests)](https://github.com/glittercowboy/get-shit-done/actions/workflows/test.yml)
[![Discord](https://img.shields.io/badge/Discord-Join-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/gsd)
[![X (Twitter)](https://img.shields.io/badge/X-@gsd__foundation-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/gsd_foundation)
[![$GSD Token](https://img.shields.io/badge/$GSD-Dexscreener-1C1C1C?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48Y2lyY2xlIGN4PSIxMiIgY3k9IjEyIiByPSIxMCIgZmlsbD0iIzAwRkYwMCIvPjwvc3ZnPg==&logoColor=00FF00)](https://dexscreener.com/solana/dwudwjvan7bzkw9zwlbyv6kspdlvhwzrqy6ebk8xzxkv)
[![GitHub stars](https://img.shields.io/github/stars/glittercowboy/get-shit-done?style=for-the-badge&logo=github&color=181717)](https://github.com/glittercowboy/get-shit-done)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)

<br>

```bash
npx get-shit-done-cc@latest
```

**Mac / Windows / Linux で動作します。**

<br>

![GSD Install](assets/terminal.svg)

<br>

*"何を作りたいかが明確なら、これはそれを作ってくれる。余計なことなし。"*

*"SpecKit、OpenSpec、Taskmasterも使ったけど、個人的に一番結果が良かった。"*

*"Claude Codeへの追加で圧倒的に強力。過剰設計なし。文字通り、ただ仕事が進む。"*

<br>

**Amazon、Google、Shopify、Webflowのエンジニアにも使われています。**

[Why I Built This](#why-i-built-this) · [How It Works](#how-it-works) · [Commands](#commands) · [Why It Works](#why-it-works) · [User Guide](docs/USER-GUIDE.md)

</div>

---

## Why I Built This

私はソロ開発者です。コードを書くのは私ではなくClaude Codeです。

仕様駆動開発ツールは他にもあります。BMAD、Speckitなど。ただ、多くは必要以上に複雑です（スプリント儀式、ストーリーポイント、ステークホルダー同期、レトロ、Jira運用など）。あるいは、作るもの全体への理解が浅い。私は50人規模のソフトウェア会社ではありません。エンタープライズごっこはしたくない。ただ、ちゃんと動く良いものを作りたいだけです。

だからGSDを作りました。複雑さはあなたの運用ではなく、システム側に閉じ込めています。裏側では、コンテキストエンジニアリング、XMLプロンプト整形、サブエージェントオーケストレーション、状態管理をやっています。あなたが見るのは「ちゃんと動く数個のコマンド」だけです。

システムは、Claudeが作業するために必要な情報と検証手段の両方を渡します。私はこのワークフローを信頼しています。実際にうまく機能します。

これがGSDです。エンタープライズのロールプレイは不要。Claude Codeで継続的に良いものを作るための、非常に実用的なシステムです。

— **TÂCHES**

---

Vibecodingは評判が悪くなりがちです。欲しいものを説明するとAIがコードを生成するが、結果は不安定で、規模が大きくなると破綻する。

GSDはそこを直します。Claude Codeを信頼できる形で動かすためのコンテキストエンジニアリング層です。アイデアを説明すれば、システムが必要情報を抽出し、Claude Codeが実装を進めます。

---

## Who This Is For

「欲しいものを説明したら、正しく形になる」ことを求める人向けです。50人規模の開発組織を運営しているフリをする必要はありません。

---

## Getting Started

```bash
npx get-shit-done-cc@latest
```

インストーラーで次を選びます:
1. **Runtime** — Claude Code、OpenCode、Gemini、Codex、またはすべて
2. **Location** — Global（全プロジェクト）または Local（現在のプロジェクトのみ）

確認コマンド:
- Claude Code / Gemini: `/gsd:help`
- OpenCode: `/gsd-help`
- Codex: `$gsd-help`

> [!NOTE]
> Codex へのインストールは、カスタムプロンプトではなく skills（`skills/gsd-*/SKILL.md`）方式です。

### Staying Updated

GSDは高速に進化しています。定期的に更新してください:

```bash
npx get-shit-done-cc@latest
```

<details>
<summary><strong>Non-interactive Install (Docker, CI, Scripts)</strong></summary>

```bash
# Claude Code
npx get-shit-done-cc --claude --global   # Install to ~/.claude/
npx get-shit-done-cc --claude --local    # Install to ./.claude/

# OpenCode (open source, free models)
npx get-shit-done-cc --opencode --global # Install to ~/.config/opencode/

# Gemini CLI
npx get-shit-done-cc --gemini --global   # Install to ~/.gemini/

# Codex (skills-first)
npx get-shit-done-cc --codex --global    # Install to ~/.codex/
npx get-shit-done-cc --codex --local     # Install to ./.codex/

# All runtimes
npx get-shit-done-cc --all --global      # Install to all directories
```

ロケーション選択をスキップするには `--global`（`-g`）または `--local`（`-l`）を使います。
ランタイム選択をスキップするには `--claude`、`--opencode`、`--gemini`、`--codex`、`--all` を使います。

</details>

<details>
<summary><strong>Development Installation</strong></summary>

リポジトリをクローンして、ローカルでインストーラーを実行します:

```bash
git clone https://github.com/glittercowboy/get-shit-done.git
cd get-shit-done
node bin/install.js --claude --local
```

`./.claude/` にインストールされるので、コントリビュート前に変更を検証できます。

</details>

### Recommended: Skip Permissions Mode

GSDは摩擦の少ない自動化を前提に設計されています。Claude Codeは次で実行してください:

```bash
claude --dangerously-skip-permissions
```

> [!TIP]
> これがGSDの想定運用です。`date` や `git commit` の承認を毎回求められる状態だと、目的を達成できません。

<details>
<summary><strong>Alternative: Granular Permissions</strong></summary>

このフラグを使いたくない場合は、プロジェクトの `.claude/settings.json` に次を追加してください:

```json
{
  "permissions": {
    "allow": [
      "Bash(date:*)",
      "Bash(echo:*)",
      "Bash(cat:*)",
      "Bash(ls:*)",
      "Bash(mkdir:*)",
      "Bash(wc:*)",
      "Bash(head:*)",
      "Bash(tail:*)",
      "Bash(sort:*)",
      "Bash(grep:*)",
      "Bash(tr:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(git tag:*)"
    ]
  }
}
```

</details>

---

## How It Works

> **既存コードがある場合** は、まず `/gsd:map-codebase` を実行してください。スタック、アーキテクチャ、規約、懸念点を並列エージェントで分析します。その後の `/gsd:new-project` は既存コードを理解した状態で進むため、質問は「何を追加するか」に集中し、計画時には既存パターンが自動読み込みされます。

### 1. Initialize Project

```
/gsd:new-project
```

1コマンドで1フロー。システムは次を行います:

1. **Questions** — 目標、制約、技術選好、エッジケースまで、必要な情報が揃うまで質問
2. **Research** — ドメイン調査を並列エージェントで実施（任意だが推奨）
3. **Requirements** — v1 / v2 / スコープ外を抽出
4. **Roadmap** — 要件にマップされたフェーズを作成

ロードマップを承認したら実装開始です。

**Creates:** `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`, `.planning/research/`

---

### 2. Discuss Phase

```
/gsd:discuss-phase 1
```

**実装の方向性を固める最重要ステップです。**

ロードマップ上の各フェーズ説明は1〜2文程度です。それだけでは「あなたが望む形」で作るには情報不足です。このステップでは、調査や計画の前に好みと判断を明確化します。

システムは、フェーズ内容から曖昧になりやすい領域を特定します:

- **Visual features** → レイアウト、密度、インタラクション、空状態
- **APIs/CLIs** → レスポンス形式、フラグ、エラーハンドリング、詳細度
- **Content systems** → 構造、トーン、深さ、流れ
- **Organization tasks** → グルーピング基準、命名、重複処理、例外

選択した領域について、納得するまで対話します。出力される `CONTEXT.md` は次の2ステップに直接使われます:

1. **Researcherが読む** — 何を調べるべきかを理解（例: カードレイアウト希望 -> カードUIパターンを調査）
2. **Plannerが読む** — 何が固定決定かを理解（例: 無限スクロール決定済み -> それを前提に計画）

ここで具体化するほど、実装結果はあなたのイメージに近づきます。スキップすれば妥当なデフォルトで進みます。使えば「あなたの設計意図」が反映されます。

**Creates:** `{phase_num}-CONTEXT.md`

---

### 3. Plan Phase

```
/gsd:plan-phase 1
```

システムは次を実行します:

1. **Researches** — CONTEXT.md の決定を反映して実装方法を調査
2. **Plans** — 2〜3タスク単位のアトミックなXMLプランを作成
3. **Verifies** — 要件照合で自己検証し、通るまで反復

各プランは新しいコンテキストウィンドウで実行できる粒度です。劣化せず、途中から簡略化されることもありません。

**Creates:** `{phase_num}-RESEARCH.md`, `{phase_num}-{N}-PLAN.md`

---

### 4. Execute Phase

```
/gsd:execute-phase 1
```

システムは次を実行します:

1. **Runs plans in waves** — 依存関係に応じて並列 / 直列を切り替え
2. **Fresh context per plan** — 各プランを独立した新規コンテキストで実行
3. **Commits per task** — 各タスクをアトミックコミット
4. **Verifies against goals** — フェーズの約束を実際のコードで検証

離席して戻れば、作業は完了し、履歴もきれいな状態です。

**How Wave Execution Works:**

プランは依存関係に基づいて「ウェーブ」に分類されます。同一wave内は並列、wave間は直列です。

```
┌─────────────────────────────────────────────────────────────────────┐
│  PHASE EXECUTION                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  WAVE 1 (parallel)          WAVE 2 (parallel)          WAVE 3       │
│  ┌─────────┐ ┌─────────┐    ┌─────────┐ ┌─────────┐    ┌─────────┐ │
│  │ Plan 01 │ │ Plan 02 │ →  │ Plan 03 │ │ Plan 04 │ →  │ Plan 05 │ │
│  │         │ │         │    │         │ │         │    │         │ │
│  │ User    │ │ Product │    │ Orders  │ │ Cart    │    │ Checkout│ │
│  │ Model   │ │ Model   │    │ API     │ │ API     │    │ UI      │ │
│  └─────────┘ └─────────┘    └─────────┘ └─────────┘    └─────────┘ │
│       │           │              ↑           ↑              ↑       │
│       └───────────┴──────────────┴───────────┘              │       │
│              Dependencies: Plan 03 needs Plan 01            │       │
│                          Plan 04 needs Plan 02              │       │
│                          Plan 05 needs Plans 03 + 04        │       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Why waves matter:**
- Independent plans -> Same wave -> Run in parallel
- Dependent plans -> Later wave -> Wait for dependencies
- File conflicts -> Sequential plans or same plan

そのため、「縦割りスライス（例: Plan 01 で User機能をエンドツーエンド実装）」は「横割りレイヤー（例: Plan 01 で全モデル、Plan 02 で全API）」より並列化しやすくなります。

**Creates:** `{phase_num}-{N}-SUMMARY.md`, `{phase_num}-VERIFICATION.md`

---

### 5. Verify Work

```
/gsd:verify-work 1
```

**実際に期待通り動くかを確定する工程です。**

自動検証は「コードがあるか」「テストが通るか」は確認できますが、「望んだ体験か」はあなたの確認が必要です。

システムは次を実行します:

1. **Extracts testable deliverables** — 今できるはずのことを抽出
2. **Walks you through one at a time** — 1項目ずつ Yes/No で確認
3. **Diagnoses failures automatically** — 失敗時はデバッグエージェントで原因特定
4. **Creates verified fix plans** — すぐ再実行できる修正プランを生成

すべて通れば次へ。問題があっても手動デバッグは不要で、生成された修正プランを `/gsd:execute-phase` で再実行できます。

**Creates:** `{phase_num}-UAT.md`, 問題があれば修正プラン

---

### 6. Repeat -> Complete -> Next Milestone

```
/gsd:discuss-phase 2
/gsd:plan-phase 2
/gsd:execute-phase 2
/gsd:verify-work 2
...
/gsd:complete-milestone
/gsd:new-milestone
```

**discuss -> plan -> execute -> verify** をマイルストーン完了まで繰り返します。

各フェーズで、意思決定（discuss）、調査（plan）、実装（execute）、人間確認（verify）を揃えることで、コンテキストを新鮮に保ちながら品質を維持します。

全フェーズ完了後、`/gsd:complete-milestone` でアーカイブとリリースタグ付けを実施。
次に `/gsd:new-milestone` で次バージョンを開始します。`new-project` と同じ流れを既存コード向けに適用します。各マイルストーンは「定義 -> 構築 -> 出荷」の独立したサイクルになります。

---

### Quick Mode

```
/gsd:quick
```

**フル計画が不要なスポット作業向け。**

Quick mode は高速ですが、GSDの保証（アトミックコミット、状態追跡）は維持します:

- **Same agents** — Planner + Executor で品質を維持
- **Skips optional steps** — 調査 / plan checker / verifier を省略
- **Separate tracking** — `.planning/quick/` で管理（フェーズ管理とは分離）

用途: バグ修正、小機能追加、設定変更、単発タスク

```
/gsd:quick
> What do you want to do? "Add dark mode toggle to settings"
```

**Creates:** `.planning/quick/001-add-dark-mode-toggle/PLAN.md`, `SUMMARY.md`

---

## Why It Works

### Context Engineering

Claude Code は、必要なコンテキストを正しく渡せば非常に強力です。多くの失敗はここで起きます。

GSDはこれを自動で処理します:

| File | What it does |
|------|--------------|
| `PROJECT.md` | プロジェクトのビジョン（常時読み込み） |
| `research/` | エコシステム知識（スタック、機能、設計、落とし穴） |
| `REQUIREMENTS.md` | v1/v2のスコープ要件とフェーズ追跡 |
| `ROADMAP.md` | 現在地と到達点の可視化 |
| `STATE.md` | 決定事項、ブロッカー、進行位置のセッション記憶 |
| `PLAN.md` | XML構造のアトミックタスク + 検証手順 |
| `SUMMARY.md` | 実施内容と変更履歴を記録 |
| `todos/` | 後で扱うアイデア / タスクを保存 |

サイズ制約は、Claudeの品質低下が始まる地点を基準に設計されています。閾値内に保つことで、一貫した品質が得られます。

### XML Prompt Formatting

各プランはClaude向けに最適化した構造化XMLです:

```xml
<task type="auto">
  <name>Create login endpoint</name>
  <files>src/app/api/auth/login/route.ts</files>
  <action>
    Use jose for JWT (not jsonwebtoken - CommonJS issues).
    Validate credentials against users table.
    Return httpOnly cookie on success.
  </action>
  <verify>curl -X POST localhost:3000/api/auth/login returns 200 + Set-Cookie</verify>
  <done>Valid credentials return cookie, invalid return 401</done>
</task>
```

曖昧さが少なく、推測を減らし、検証まで内包しています。

### Multi-Agent Orchestration

各段階は同じ構造です。薄いオーケストレーターが専門エージェントを起動し、結果を統合して次へ渡します。

| Stage | Orchestrator does | Agents do |
|-------|------------------|-----------|
| Research | 進行管理と結果提示 | 4つの並列研究エージェントがスタック、機能、設計、落とし穴を調査 |
| Planning | 検証と反復管理 | Plannerが計画作成、Checkerが検証し、合格まで反復 |
| Execution | Wave実行と進捗管理 | Executorが並列実装（各自が新規コンテキスト） |
| Verification | 結果提示と分岐制御 | Verifierが目標達成を検証し、Debuggerが失敗原因を特定 |

オーケストレーターは重い処理を行わず、起動・待機・統合に専念します。

**結果:** 深い調査、複数プラン作成・検証、並列実装、大量コード生成、目標照合まで実行しても、メインセッションのコンテキストは軽いまま保てます。

### Atomic Git Commits

各タスク完了直後に個別コミットします:

```bash
abc123f docs(08-02): complete user registration plan
def456g feat(08-02): add email confirmation flow
hij789k feat(08-02): implement password hashing
lmn012o feat(08-02): create registration endpoint
```

> [!NOTE]
> **Benefits:** `git bisect` で失敗地点を即特定。タスク単位でリバート可能。将来セッションでClaudeが履歴を読みやすい。AI自動化ワークフローの可観測性が向上。

履歴は常に外科的で追跡可能、意味のある単位になります。

### Modular by Design

- 現在のマイルストーンにフェーズ追加
- フェーズ間へ緊急作業を挿入
- マイルストーン完了後に新規開始
- 全体を壊さず計画だけ調整

固定化されず、状況に合わせて進化できます。

---

## Commands

### Core Workflow

| Command | What it does |
|---------|--------------|
| `/gsd:new-project [--auto]` | 初期化フルフロー: 質問 -> 調査 -> 要件 -> ロードマップ |
| `/gsd:discuss-phase [N] [--auto]` | 計画前に実装意思決定を確定 |
| `/gsd:plan-phase [N] [--auto]` | フェーズ単位で調査 + 計画 + 検証 |
| `/gsd:execute-phase <N>` | ウェーブ並列で実行、完了時に検証 |
| `/gsd:verify-work [N]` | 手動受け入れテスト（UAT）¹ |
| `/gsd:audit-milestone` | マイルストーンのDefinition of Doneを監査 |
| `/gsd:complete-milestone` | マイルストーンをアーカイブし、リリースタグ付け |
| `/gsd:new-milestone [name]` | 次バージョン開始: 質問 -> 調査 -> 要件 -> ロードマップ |

### Navigation

| Command | What it does |
|---------|--------------|
| `/gsd:progress` | 現在地と次アクションを表示 |
| `/gsd:help` | 全コマンドと使い方を表示 |
| `/gsd:update` | 変更履歴プレビュー付きで更新 |
| `/gsd:join-discord` | GSD Discord コミュニティへ参加 |

### Brownfield

| Command | What it does |
|---------|--------------|
| `/gsd:map-codebase` | 既存コードベースを分析してからnew-projectへ |

### Phase Management

| Command | What it does |
|---------|--------------|
| `/gsd:add-phase` | ロードマップ末尾にフェーズ追加 |
| `/gsd:insert-phase [N]` | フェーズ間に緊急作業を挿入 |
| `/gsd:remove-phase [N]` | 未来フェーズを削除して再採番 |
| `/gsd:list-phase-assumptions [N]` | 計画前の想定アプローチを確認 |
| `/gsd:plan-milestone-gaps` | 監査で見つかった不足を埋めるフェーズを生成 |

### Session

| Command | What it does |
|---------|--------------|
| `/gsd:pause-work` | フェーズ途中で中断用ハンドオフを作成 |
| `/gsd:resume-work` | 前回セッション状態から再開 |

### Utilities

| Command | What it does |
|---------|--------------|
| `/gsd:settings` | モデルプロファイルとワークフローエージェントを設定 |
| `/gsd:set-profile <profile>` | モデルプロファイル切替（quality/balanced/budget） |
| `/gsd:add-todo [desc]` | 後でやるタスクを記録 |
| `/gsd:check-todos` | 保留Todoを一覧表示 |
| `/gsd:debug [desc]` | 永続状態付きの体系的デバッグ |
| `/gsd:quick [--full] [--discuss]` | スポットタスクをGSD保証で実行（`--full`で計画検証 + 検証追加、`--discuss`で先に文脈収集） |
| `/gsd:health [--repair]` | `.planning/` 整合性を検証し、`--repair`で自動修復 |

<sup>¹ Contributed by reddit user OracleGreyBeard</sup>

---

## Configuration

GSDのプロジェクト設定は `.planning/config.json` に保存されます。`/gsd:new-project` 時に設定するか、後で `/gsd:settings` で更新できます。完全なスキーマ、ワークフロートグル、gitブランチ設定、エージェント別モデル内訳は [User Guide](docs/USER-GUIDE.md#configuration-reference) を参照してください。

### Core Settings

| Setting | Options | Default | What it controls |
|---------|---------|---------|------------------|
| `mode` | `yolo`, `interactive` | `interactive` | 自動承認か、各ステップ確認か |
| `granularity` | `coarse`, `standard`, `fine` | `standard` | フェーズ粒度（フェーズ x プランの分割細かさ） |

### Model Profiles

各エージェントが使うモデルを制御します。品質とトークンコストのバランスを調整します。

| Profile | Planning | Execution | Verification |
|---------|----------|-----------|--------------|
| `quality` | Opus | Opus | Sonnet |
| `balanced` (default) | Opus | Sonnet | Sonnet |
| `budget` | Sonnet | Sonnet | Haiku |

切り替え:
```
/gsd:set-profile budget
```

または `/gsd:settings` から設定してください。

### Workflow Agents

これらは計画 / 実行中に追加エージェントを起動します。品質は上がりますが、トークンと時間は増えます。

| Setting | Default | What it does |
|---------|---------|--------------|
| `workflow.research` | `true` | 各フェーズの計画前にドメイン調査 |
| `workflow.plan_check` | `true` | 実行前に計画がフェーズ目標を満たすか検証 |
| `workflow.verifier` | `true` | 実行後にmust-haves達成を確認 |
| `workflow.auto_advance` | `false` | discuss -> plan -> execute を自動連鎖 |

`/gsd:settings` で切り替えるか、実行時に上書き可能:
- `/gsd:plan-phase --skip-research`
- `/gsd:plan-phase --skip-verify`

### Execution

| Setting | Default | What it controls |
|---------|---------|------------------|
| `parallelization.enabled` | `true` | 独立プランを同時実行 |
| `planning.commit_docs` | `true` | `.planning/` を git で追跡 |

### Git Branching

実行中のブランチ運用を制御します。

| Setting | Options | Default | What it does |
|---------|---------|---------|--------------|
| `git.branching_strategy` | `none`, `phase`, `milestone` | `none` | ブランチ作成戦略 |
| `git.phase_branch_template` | string | `gsd/phase-{phase}-{slug}` | フェーズブランチ名テンプレート |
| `git.milestone_branch_template` | string | `gsd/{milestone}-{slug}` | マイルストーンブランチ名テンプレート |

**Strategies:**
- **`none`** — 現在のブランチにコミット（GSDデフォルト）
- **`phase`** — フェーズごとにブランチを作成し、フェーズ完了時にマージ
- **`milestone`** — マイルストーン全体で1ブランチを作成し、完了時にマージ

マイルストーン完了時には squash merge（推奨）または履歴保持マージを選べます。

---

## Security

### Protecting Sensitive Files

GSDのコードベース分析コマンドは、プロジェクト理解のためにファイルを読み取ります。**シークレットを含むファイルは保護してください**。Claude Codeの deny list に追加します:

1. Claude Code設定（`.claude/settings.json` またはグローバル）を開く
2. deny list に機密ファイルパターンを追加

```json
{
  "permissions": {
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Read(**/secrets/*)",
      "Read(**/*credential*)",
      "Read(**/*.pem)",
      "Read(**/*.key)"
    ]
  }
}
```

これで、どのコマンドを実行しても対象ファイルの読み取りを防げます。

> [!IMPORTANT]
> GSDにはシークレット誤コミット防止が組み込まれていますが、多層防御がベストプラクティスです。まずは機密ファイルの読み取り拒否を設定してください。

---

## Troubleshooting

**インストール後にコマンドが見つからない場合**
- ランタイムを再起動してコマンド/skillsを再読み込み
- Claude向けなら `~/.claude/commands/gsd/`（global）または `./.claude/commands/gsd/`（local）を確認
- Codex向けなら `~/.codex/skills/gsd-*/SKILL.md`（global）または `./.codex/skills/gsd-*/SKILL.md`（local）を確認

**コマンドが期待通り動かない場合**
- `/gsd:help` で導入状態を確認
- `npx get-shit-done-cc` を再実行して再インストール

**最新版に更新する場合**
```bash
npx get-shit-done-cc@latest
```

**Docker / コンテナ環境で使う場合**

チルダパス（`~/.claude/...`）で読み取り失敗する場合は、インストール前に `CLAUDE_CONFIG_DIR` を設定してください:
```bash
CLAUDE_CONFIG_DIR=/home/youruser/.claude npx get-shit-done-cc --global
```
これで `~` ではなく絶対パスが使われ、コンテナ環境でも安定します。

### Uninstalling

GSDを完全に削除する場合:

```bash
# Global installs
npx get-shit-done-cc --claude --global --uninstall
npx get-shit-done-cc --opencode --global --uninstall
npx get-shit-done-cc --codex --global --uninstall

# Local installs (current project)
npx get-shit-done-cc --claude --local --uninstall
npx get-shit-done-cc --opencode --local --uninstall
npx get-shit-done-cc --codex --local --uninstall
```

これにより、GSDのコマンド、エージェント、フック、設定が削除され、他の設定は保持されます。

---

## Community Ports

OpenCode、Gemini CLI、Codex は現在 `npx get-shit-done-cc` でネイティブ対応しています。

マルチランタイム対応を先行して切り開いたコミュニティポート:

| Project | Platform | Description |
|---------|----------|-------------|
| [gsd-opencode](https://github.com/rokicool/gsd-opencode) | OpenCode | Original OpenCode adaptation |
| gsd-gemini (archived) | Gemini CLI | Original Gemini adaptation by uberfuzzy |

---

## Star History

<a href="https://star-history.com/#glittercowboy/get-shit-done&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=glittercowboy/get-shit-done&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=glittercowboy/get-shit-done&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=glittercowboy/get-shit-done&type=Date" />
 </picture>
</a>

---

## License

MIT License. 詳細は [LICENSE](LICENSE) を参照してください。

---

<div align="center">

**Claude Code は強力。GSD はそれを信頼できる開発体験にします。**

</div>
