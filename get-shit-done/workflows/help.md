<purpose>
完全なGSDコマンドリファレンスを表示する。リファレンスの内容のみを出力する。プロジェクト固有の分析、gitステータス、次ステップの提案、リファレンス以外のコメントは追加しないこと。
</purpose>

<reference>
# GSD コマンドリファレンス

**GSD**（Get Shit Done）は、Claude Codeによるソロエージェント開発に最適化された階層的プロジェクトプランを作成する。

## クイックスタート

1. `/gsd:new-project` - プロジェクトを初期化（リサーチ、要件、ロードマップを含む）
2. `/gsd:plan-phase 1` - 最初のフェーズの詳細プランを作成
3. `/gsd:execute-phase 1` - フェーズを実行

## 最新の状態を保つ

GSDは急速に進化している。定期的に更新すること：

```bash
npx get-shit-done-cc@latest
```

## コアワークフロー

```
/gsd:new-project → /gsd:plan-phase → /gsd:execute-phase → repeat
```

### プロジェクト初期化

**`/gsd:new-project`**
統合フローで新しいプロジェクトを初期化する。

1つのコマンドでアイデアから計画準備完了まで導く：
- 何を構築するかを理解するための深い質問
- 任意のドメインリサーチ（4つの並列リサーチャーエージェントを生成）
- v1/v2/スコープ外のスコーピングを含む要件定義
- 成功基準付きのフェーズ分割によるロードマップ作成

すべての`.planning/`アーティファクトを作成：
- `PROJECT.md` — ビジョンと要件
- `config.json` — ワークフローモード（interactive/yolo）
- `research/` — ドメインリサーチ（選択した場合）
- `REQUIREMENTS.md` — REQ-ID付きのスコープ済み要件
- `ROADMAP.md` — 要件にマッピングされたフェーズ
- `STATE.md` — プロジェクトメモリ

使い方: `/gsd:new-project`

**`/gsd:map-codebase`**
ブラウンフィールドプロジェクト用に既存のコードベースをマッピングする。

- 並列Exploreエージェントでコードベースを分析
- `.planning/codebase/`に7つの焦点を絞ったドキュメントを作成
- スタック、アーキテクチャ、構造、規約、テスト、統合、懸念事項をカバー
- 既存コードベースでは`/gsd:new-project`の前に使用

使い方: `/gsd:map-codebase`

### フェーズ計画

**`/gsd:discuss-phase <number>`**
計画前にフェーズのビジョンを明確にする手助けをする。

- このフェーズがどのように動作するか想像しているかを収集
- ビジョン、必須事項、境界を含むCONTEXT.mdを作成
- 見た目/感触についてアイデアがある場合に使用

使い方: `/gsd:discuss-phase 2`

**`/gsd:research-phase <number>`**
ニッチ/複雑なドメインのための包括的なエコシステムリサーチ。

- 標準スタック、アーキテクチャパターン、落とし穴を発見
- 「専門家がこれをどう構築するか」の知識を含むRESEARCH.mdを作成
- 3D、ゲーム、オーディオ、シェーダー、ML、その他の専門ドメインに使用
- 「どのライブラリ」を超えてエコシステムの知識に踏み込む

使い方: `/gsd:research-phase 3`

**`/gsd:list-phase-assumptions <number>`**
Claudeが開始前に何を計画しているかを確認する。

- フェーズに対するClaudeの意図するアプローチを表示
- Claudeがビジョンを誤解している場合に軌道修正が可能
- ファイルは作成されない - 会話出力のみ

使い方: `/gsd:list-phase-assumptions 3`

**`/gsd:plan-phase <number>`**
特定のフェーズの詳細な実行プランを作成する。

- `.planning/phases/XX-phase-name/XX-YY-PLAN.md`を生成
- フェーズを具体的で実行可能なタスクに分解
- 検証基準と成功指標を含む
- フェーズごとに複数プラン対応（XX-01、XX-02など）

使い方: `/gsd:plan-phase 1`
結果: `.planning/phases/01-foundation/01-01-PLAN.md`を作成

**PRDエクスプレスパス：** `--prd path/to/requirements.md`を渡すとdiscuss-phaseを完全にスキップ。PRDがCONTEXT.mdのロックされた決定になる。明確な受け入れ基準が既にある場合に便利。

### 実行

**`/gsd:execute-phase <phase-number>`**
フェーズ内のすべてのプランを実行する。

- プランをウェーブ（フロントマターから）でグループ化し、ウェーブを順次実行
- 各ウェーブ内のプランはTaskツールで並列実行
- すべてのプラン完了後にフェーズ目標を検証
- REQUIREMENTS.md、ROADMAP.md、STATE.mdを更新

使い方: `/gsd:execute-phase 5`

### クイックモード

**`/gsd:quick`**
GSDの保証付きで小規模なアドホックタスクを実行し、オプションのエージェントをスキップする。

クイックモードは同じシステムでより短いパスを使用：
- プランナー＋エグゼキューターを生成（リサーチャー、チェッカー、ベリファイアーをスキップ）
- クイックタスクは計画済みフェーズとは別の`.planning/quick/`に格納
- STATE.md追跡を更新（ROADMAP.mdではない）

何をすべきか正確にわかっていて、リサーチや検証が不要な小さなタスクの場合に使用。

使い方: `/gsd:quick`
結果: `.planning/quick/NNN-slug/PLAN.md`、`.planning/quick/NNN-slug/SUMMARY.md`を作成

### ロードマップ管理

**`/gsd:add-phase <description>`**
現在のマイルストーンの末尾に新しいフェーズを追加する。

- ROADMAP.mdに追記
- 次の連番を使用
- フェーズディレクトリ構造を更新

使い方: `/gsd:add-phase "Add admin dashboard"`

**`/gsd:insert-phase <after> <description>`**
既存フェーズの間に小数フェーズとして緊急作業を挿入する。

- 中間フェーズを作成（例：7と8の間に7.1）
- マイルストーン途中で発見された作業に便利
- フェーズの順序を維持

使い方: `/gsd:insert-phase 7 "Fix critical auth bug"`
結果: Phase 7.1を作成

**`/gsd:remove-phase <number>`**
将来のフェーズを削除し、後続フェーズの番号を振り直す。

- フェーズディレクトリとすべての参照を削除
- すべての後続フェーズの番号を振り直してギャップを埋める
- 将来の（未開始の）フェーズにのみ有効
- Gitコミットが履歴記録を保持

使い方: `/gsd:remove-phase 17`
結果: Phase 17が削除され、フェーズ18-20が17-19になる

### マイルストーン管理

**`/gsd:new-milestone <name>`**
統合フローで新しいマイルストーンを開始する。

- 次に構築するものを理解するための深い質問
- 任意のドメインリサーチ（4つの並列リサーチャーエージェントを生成）
- スコーピング付きの要件定義
- フェーズ分割によるロードマップ作成

ブラウンフィールドプロジェクト（既存のPROJECT.md）向けに`/gsd:new-project`フローを反映。

使い方: `/gsd:new-milestone "v2.0 Features"`

**`/gsd:complete-milestone <version>`**
完了したマイルストーンをアーカイブし、次のバージョンの準備をする。

- 統計付きのMILESTONES.mdエントリを作成
- milestones/ディレクトリに完全な詳細をアーカイブ
- リリース用のgitタグを作成
- 次のバージョン用にワークスペースを準備

使い方: `/gsd:complete-milestone 1.0.0`

### 進捗追跡

**`/gsd:progress`**
プロジェクトステータスを確認し、次のアクションにインテリジェントにルーティングする。

- ビジュアルなプログレスバーと完了率を表示
- SUMMARYファイルからの最近の作業を要約
- 現在の位置と次の内容を表示
- 主要な決定とオープンな問題をリスト
- 次のプランの実行を提案、または不足している場合は作成を提案
- 100%マイルストーン完了を検出

使い方: `/gsd:progress`

### セッション管理

**`/gsd:resume-work`**
完全なコンテキスト復元で前回のセッションから作業を再開する。

- STATE.mdを読みプロジェクトコンテキストを取得
- 現在の位置と最近の進捗を表示
- プロジェクト状態に基づいて次のアクションを提案

使い方: `/gsd:resume-work`

**`/gsd:pause-work`**
フェーズ途中で作業を一時停止する際にコンテキストハンドオフを作成する。

- 現在の状態を含む.continue-hereファイルを作成
- STATE.mdのセッション継続セクションを更新
- 進行中の作業コンテキストを記録

使い方: `/gsd:pause-work`

### デバッグ

**`/gsd:debug [issue description]`**
コンテキストリセット間で持続的な状態を持つ体系的なデバッグ。

- 適応的な質問を通じて症状を収集
- 調査を追跡する`.planning/debug/[slug].md`を作成
- 科学的手法（証拠 → 仮説 → テスト）を使用して調査
- `/clear`後も存続 — 引数なしで`/gsd:debug`を実行して再開
- 解決済みの問題を`.planning/debug/resolved/`にアーカイブ

使い方: `/gsd:debug "login button doesn't work"`
使い方: `/gsd:debug` (アクティブなセッションを再開)

### Todo管理

**`/gsd:add-todo [description]`**
現在の会話からアイデアやタスクをtodoとして記録する。

- 会話からコンテキストを抽出（または提供された説明を使用）
- `.planning/todos/pending/`に構造化されたtodoファイルを作成
- グループ化のためにファイルパスからエリアを推論
- 作成前に重複をチェック
- STATE.mdのtodoカウントを更新

使い方: `/gsd:add-todo` (会話から推論)
使い方: `/gsd:add-todo Add auth token refresh`

**`/gsd:check-todos [area]`**
保留中のtodoをリストし、作業するものを選択する。

- すべての保留中のtodoをタイトル、エリア、経過時間付きでリスト
- オプションのエリアフィルター（例：`/gsd:check-todos api`）
- 選択したtodoの完全なコンテキストを読み込み
- 適切なアクション（今すぐ作業、フェーズに追加、ブレインストーム）にルーティング
- 作業開始時にtodoをdone/に移動

使い方: `/gsd:check-todos`
使い方: `/gsd:check-todos api`

### ユーザー受け入れテスト

**`/gsd:verify-work [phase]`**
会話型UATを通じて構築された機能を検証する。

- SUMMARYファイルからテスト可能な成果物を抽出
- テストを1つずつ提示（yes/no応答）
- 失敗を自動診断し修正プランを作成
- 問題が見つかった場合は再実行の準備が完了

使い方: `/gsd:verify-work 3`

### マイルストーン監査

**`/gsd:audit-milestone [version]`**
元の意図に対するマイルストーンの完了を監査する。

- すべてのフェーズのVERIFICATION.mdファイルを読む
- 要件カバレッジを確認
- クロスフェーズの接続のための統合チェッカーを生成
- ギャップと技術的負債を含むMILESTONE-AUDIT.mdを作成

使い方: `/gsd:audit-milestone`

**`/gsd:plan-milestone-gaps`**
監査で特定されたギャップを埋めるフェーズを作成する。

- MILESTONE-AUDIT.mdを読み、ギャップをフェーズにグループ化
- 要件の優先度（must/should/nice）で優先順位付け
- ROADMAP.mdにギャップクロージャーフェーズを追加
- 新しいフェーズに対する`/gsd:plan-phase`の準備が完了

使い方: `/gsd:plan-milestone-gaps`

### 設定

**`/gsd:settings`**
ワークフロートグルとモデルプロファイルをインタラクティブに設定する。

- リサーチャー、プランチェッカー、ベリファイアーエージェントのトグル
- モデルプロファイルの選択（quality/balanced/budget）
- `.planning/config.json`を更新

使い方: `/gsd:settings`

**`/gsd:set-profile <profile>`**
GSDエージェントのモデルプロファイルをクイック切り替え。

- `quality` — 検証以外すべてにOpus
- `balanced` — 計画にOpus、実行にSonnet（デフォルト）
- `budget` — 執筆にSonnet、リサーチ/検証にHaiku

使い方: `/gsd:set-profile budget`

### ユーティリティコマンド

**`/gsd:cleanup`**
完了したマイルストーンから蓄積されたフェーズディレクトリをアーカイブする。

- `.planning/phases/`にまだ残っている完了マイルストーンのフェーズを特定
- 移動前にドライランサマリーを表示
- フェーズディレクトリを`.planning/milestones/v{X.Y}-phases/`に移動
- 複数のマイルストーン後に`.planning/phases/`の散らかりを減らすために使用

使い方: `/gsd:cleanup`

**`/gsd:help`**
このコマンドリファレンスを表示する。

**`/gsd:update`**
変更ログプレビュー付きでGSDを最新バージョンに更新する。

- インストール済みと最新バージョンの比較を表示
- 見逃したバージョンの変更ログエントリを表示
- 破壊的変更をハイライト
- インストール前に確認
- 生の`npx get-shit-done-cc`より優れている

使い方: `/gsd:update`

**`/gsd:join-discord`**
GSD Discordコミュニティに参加する。

- ヘルプを得る、構築しているものを共有、最新情報を入手
- 他のGSDユーザーとつながる

使い方: `/gsd:join-discord`

## ファイルと構造

```
.planning/
├── PROJECT.md            # プロジェクトビジョン
├── ROADMAP.md            # 現在のフェーズ分割
├── STATE.md              # プロジェクトメモリとコンテキスト
├── RETROSPECTIVE.md      # リビングレトロスペクティブ（マイルストーンごとに更新）
├── config.json           # ワークフローモードとゲート
├── todos/                # 記録されたアイデアとタスク
│   ├── pending/          # 作業待ちのTodo
│   └── done/             # 完了したTodo
├── debug/                # アクティブなデバッグセッション
│   └── resolved/         # アーカイブされた解決済み問題
├── milestones/
│   ├── v1.0-ROADMAP.md       # アーカイブされたロードマップスナップショット
│   ├── v1.0-REQUIREMENTS.md  # アーカイブされた要件
│   └── v1.0-phases/          # アーカイブされたフェーズディレクトリ（/gsd:cleanupまたは--archive-phases経由）
│       ├── 01-foundation/
│       └── 02-core-features/
├── codebase/             # コードベースマップ（ブラウンフィールドプロジェクト）
│   ├── STACK.md          # 言語、フレームワーク、依存関係
│   ├── ARCHITECTURE.md   # パターン、レイヤー、データフロー
│   ├── STRUCTURE.md      # ディレクトリレイアウト、キーファイル
│   ├── CONVENTIONS.md    # コーディング規約、命名
│   ├── TESTING.md        # テストセットアップ、パターン
│   ├── INTEGRATIONS.md   # 外部サービス、API
│   └── CONCERNS.md       # 技術的負債、既知の問題
└── phases/
    ├── 01-foundation/
    │   ├── 01-01-PLAN.md
    │   └── 01-01-SUMMARY.md
    └── 02-core-features/
        ├── 02-01-PLAN.md
        └── 02-01-SUMMARY.md
```

## ワークフローモード

`/gsd:new-project`で設定：

**Interactive Mode**

- 各主要な決定を確認
- 承認のためにチェックポイントで一時停止
- 全体を通じてより多くのガイダンス

**YOLO Mode**

- ほとんどの決定を自動承認
- 確認なしでプランを実行
- クリティカルなチェックポイントでのみ停止

`.planning/config.json`を編集していつでも変更可能

## 計画設定

`.planning/config.json`で計画アーティファクトの管理方法を設定：

**`planning.commit_docs`** (デフォルト: `true`)
- `true`: 計画アーティファクトをgitにコミット（標準ワークフロー）
- `false`: 計画アーティファクトをローカル専用に保持、コミットしない

`commit_docs: false`の場合：
- `.planning/`を`.gitignore`に追加
- OSSコントリビューション、クライアントプロジェクト、計画をプライベートに保ちたい場合に便利
- すべての計画ファイルは通常通り動作するが、gitで追跡されない

**`planning.search_gitignored`** (デフォルト: `false`)
- `true`: 広範なripgrep検索に`--no-ignore`を追加
- `.planning/`がgitignoreされていて、プロジェクト全体の検索に含めたい場合にのみ必要

設定例：
```json
{
  "planning": {
    "commit_docs": false,
    "search_gitignored": true
  }
}
```

## 一般的なワークフロー

**新しいプロジェクトを開始する：**

```
/gsd:new-project        # 統合フロー：質問 → リサーチ → 要件 → ロードマップ
/clear
/gsd:plan-phase 1       # 最初のフェーズのプランを作成
/clear
/gsd:execute-phase 1    # フェーズ内のすべてのプランを実行
```

**休憩後に作業を再開する：**

```
/gsd:progress  # 中断した場所を確認して続行
```

**マイルストーン途中で緊急作業を追加する：**

```
/gsd:insert-phase 5 "Critical security fix"
/gsd:plan-phase 5.1
/gsd:execute-phase 5.1
```

**マイルストーンを完了する：**

```
/gsd:complete-milestone 1.0.0
/clear
/gsd:new-milestone  # 次のマイルストーンを開始（質問 → リサーチ → 要件 → ロードマップ）
```

**作業中にアイデアを記録する：**

```
/gsd:add-todo                    # 会話コンテキストから記録
/gsd:add-todo Fix modal z-index  # 明示的な説明で記録
/gsd:check-todos                 # Todoを確認して作業
/gsd:check-todos api             # エリアでフィルター
```

**問題をデバッグする：**

```
/gsd:debug "form submission fails silently"  # デバッグセッションを開始
# ... 調査が行われ、コンテキストがいっぱいになる ...
/clear
/gsd:debug                                    # 中断した場所から再開
```

## ヘルプを得る

- `.planning/PROJECT.md`を読んでプロジェクトビジョンを確認
- `.planning/STATE.md`を読んで現在のコンテキストを確認
- `.planning/ROADMAP.md`を確認してフェーズステータスを確認
- `/gsd:progress`を実行して現在の進捗を確認
</reference>
</output>