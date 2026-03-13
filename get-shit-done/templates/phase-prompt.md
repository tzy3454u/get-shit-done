# フェーズプロンプトテンプレート

> **注意:** 計画手法は`agents/gsd-planner.md`にあります。
> このテンプレートはエージェントが生成するPLAN.mdの出力形式を定義します。

`.planning/phases/XX-name/{phase}-{plan}-PLAN.md` 用テンプレート - 並列実行に最適化された実行可能なフェーズプラン。

**命名規則:** `{phase}-{plan}-PLAN.md`形式を使用（例: Phase 1のPlan 2の場合`01-02-PLAN.md`）

---

## ファイルテンプレート

```markdown
---
phase: XX-name
plan: NN
type: execute
wave: N                     # 実行ウェーブ（1, 2, 3...）。プラン時に事前計算。
depends_on: []              # このプランが必要とするプランID（例: ["01-01"]）。
files_modified: []          # このプランが変更するファイル。
autonomous: true            # プランにユーザーインタラクションを必要とするチェックポイントがある場合はfalse
requirements: []            # 必須 — このプランが対処するROADMAPの要件ID。空であってはならない。
user_setup: []              # Claudeが自動化できない人間が必要なセットアップ（下記参照）

# ゴールからの逆算検証（計画中に導出、実行後に検証）
must_haves:
  truths: []                # ゴール達成のために真でなければならない観察可能な動作
  artifacts: []             # 実際の実装で存在しなければならないファイル
  key_links: []             # アーティファクト間の重要な接続
---

<objective>
[このプランが達成すること]

Purpose: [プロジェクトにとってなぜこれが重要か]
Output: [作成されるアーティファクト]
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/execute-plan.md
@~/.claude/get-shit-done/templates/summary.md
[プランにチェックポイントタスク（type="checkpoint:*"）が含まれる場合、追加:]
@~/.claude/get-shit-done/references/checkpoints.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md

# 本当に必要な場合のみ、前のプランのSUMMARYを参照:
# - このプランが前のプランの型/エクスポートを使用する
# - 前のプランがこのプランに影響する判断をした
# 反射的にチェーンしない: Plan 02がPlan 01を参照、Plan 03がPlan 02を参照...

[関連するソースファイル:]
@src/path/to/relevant.ts
</context>

<tasks>

<task type="auto">
  <name>Task 1: [アクション指向の名前]</name>
  <files>path/to/file.ext, another/file.ext</files>
  <read_first>path/to/reference.ext, path/to/source-of-truth.ext</read_first>
  <action>[具体的な実装 - 何をするか、どのようにするか、何を避けるべきか、その理由。具体的な値を含める: 正確な識別子、パラメータ、期待される出力、ファイルパス、コマンド引数。正確なターゲット状態を指定せずに「XをYに合わせる」と言わないこと。]</action>
  <verify>[動作を証明するコマンドまたはチェック]</verify>
  <acceptance_criteria>
    - [grep検証可能な条件: "file.extに'exact string'が含まれる"]
    - [測定可能な条件: "output.extが'expected-value'を使用し、'wrong-value'ではない"]
  </acceptance_criteria>
  <done>[測定可能な受け入れ基準]</done>
</task>

<task type="auto">
  <name>Task 2: [アクション指向の名前]</name>
  <files>path/to/file.ext</files>
  <read_first>path/to/reference.ext</read_first>
  <action>[具体的な値を含む具体的な実装]</action>
  <verify>[コマンドまたはチェック]</verify>
  <acceptance_criteria>
    - [grep検証可能な条件]
  </acceptance_criteria>
  <done>[受け入れ基準]</done>
</task>

<!-- チェックポイントタスクの例とパターンについては、@~/.claude/get-shit-done/references/checkpoints.md を参照 -->
<!-- 重要なルール: Claudeはhuman-verifyチェックポイントの前に開発サーバーを起動します。ユーザーはURLにアクセスするだけです。 -->

<task type="checkpoint:decision" gate="blocking">
  <decision>[決定が必要なこと]</decision>
  <context>[この決定がなぜ重要か]</context>
  <options>
    <option id="option-a"><name>[名前]</name><pros>[メリット]</pros><cons>[トレードオフ]</cons></option>
    <option id="option-b"><name>[名前]</name><pros>[メリット]</pros><cons>[トレードオフ]</cons></option>
  </options>
  <resume-signal>Select: option-a or option-b</resume-signal>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[Claudeが構築したもの] - サーバーが[URL]で稼働中</what-built>
  <how-to-verify>[URL]にアクセスして確認: [視覚的なチェックのみ、CLIコマンドなし]</how-to-verify>
  <resume-signal>「approved」と入力するか、問題を説明してください</resume-signal>
</task>

</tasks>

<verification>
プラン完了を宣言する前に:
- [ ] [具体的なテストコマンド]
- [ ] [ビルド/型チェックの通過]
- [ ] [動作の検証]
</verification>

<success_criteria>

- すべてのタスクが完了
- すべての検証チェックが通過
- エラーや警告が新たに発生していない
- [プラン固有の基準]
  </success_criteria>

<output>
完了後、`.planning/phases/XX-name/{phase}-{plan}-SUMMARY.md`を作成
</output>
```

---

## フロントマターフィールド

| フィールド | 必須 | 目的 |
|-------|----------|---------|
| `phase` | はい | フェーズ識別子（例: `01-foundation`） |
| `plan` | はい | フェーズ内のプラン番号（例: `01`、`02`） |
| `type` | はい | 標準プランは常に`execute`、TDDプランは`tdd` |
| `wave` | はい | 実行ウェーブ番号（1, 2, 3...）。プラン時に事前計算。 |
| `depends_on` | はい | このプランが必要とするプランIDの配列。 |
| `files_modified` | はい | このプランが変更するファイル。 |
| `autonomous` | はい | チェックポイントがなければ`true`、あれば`false` |
| `requirements` | はい | ROADMAPの要件IDを**必ず**リストする。すべてのロードマップ要件は少なくとも1つのプランに含まれる必要がある。 |
| `user_setup` | いいえ | 人間が必要なセットアップ項目の配列（外部サービス） |
| `must_haves` | はい | ゴールからの逆算検証基準（下記参照） |

**ウェーブは事前計算:** ウェーブ番号は`/gsd:plan-phase`中に割り当てられます。execute-phaseはフロントマターから`wave`を直接読み取り、ウェーブ番号でプランをグループ化します。ランタイムの依存関係分析は不要です。

**must_havesが検証を可能にする:** `must_haves`フィールドは計画から実行へのゴールからの逆算要件を伝達します。すべてのプランが完了した後、execute-phaseはこれらの基準を実際のコードベースに対してチェックする検証サブエージェントを起動します。

---

## 並列 vs 順次

<parallel_examples>

**ウェーブ1候補（並列）:**

```yaml
# Plan 01 - ユーザー機能
wave: 1
depends_on: []
files_modified: [src/models/user.ts, src/api/users.ts]
autonomous: true

# Plan 02 - 製品機能（Plan 01と重複なし）
wave: 1
depends_on: []
files_modified: [src/models/product.ts, src/api/products.ts]
autonomous: true

# Plan 03 - 注文機能（重複なし）
wave: 1
depends_on: []
files_modified: [src/models/order.ts, src/api/orders.ts]
autonomous: true
```

3つすべてが並列で実行（ウェーブ1）- 依存関係なし、ファイルの競合なし。

**順次（本当の依存関係）:**

```yaml
# Plan 01 - 認証基盤
wave: 1
depends_on: []
files_modified: [src/lib/auth.ts, src/middleware/auth.ts]
autonomous: true

# Plan 02 - 保護された機能（認証が必要）
wave: 2
depends_on: ["01"]
files_modified: [src/features/dashboard.ts]
autonomous: true
```

Plan 02はウェーブ2でウェーブ1のPlan 01を待つ - 認証の型/ミドルウェアへの本当の依存関係。

**チェックポイントプラン:**

```yaml
# Plan 03 - 検証付きUI
wave: 3
depends_on: ["01", "02"]
files_modified: [src/components/Dashboard.tsx]
autonomous: false  # checkpoint:human-verifyあり
```

ウェーブ3はウェーブ1と2の後に実行。チェックポイントで一時停止し、オーケストレーターがユーザーに提示、承認後に再開。

</parallel_examples>

---

## コンテキストセクション

**並列対応のコンテキスト:**

```markdown
<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md

# 本当に必要な場合のみSUMMARY参照を含める:
# - このプランが前のプランの型をインポートする
# - 前のプランがこのプランに影響する判断をした
# - 前のプランの出力がこのプランの入力になる
#
# 独立したプランは前のSUMMARY参照を必要としない。
# 反射的にチェーンしない: 02が01を参照、03が02を参照...

@src/relevant/source.ts
</context>
```

**悪いパターン（偽の依存関係を作る）:**
```markdown
<context>
@.planning/phases/03-features/03-01-SUMMARY.md  # 単に先に来るから
@.planning/phases/03-features/03-02-SUMMARY.md  # 反射的なチェーン
</context>
```

---

## スコープのガイダンス

**プランのサイズ:**

- プランごとに2〜3タスク
- 最大約50%のコンテキスト使用量
- 複雑なフェーズ: 1つの大きなプランではなく、複数の焦点を絞ったプラン

**分割するタイミング:**

- 異なるサブシステム（認証 vs API vs UI）
- 3タスク以上
- コンテキストオーバーフローのリスク
- TDD候補 - 別のプラン

**垂直スライスが望ましい:**

```
推奨: Plan 01 = User（モデル + API + UI）
      Plan 02 = Product（モデル + API + UI）

避ける: Plan 01 = すべてのモデル
        Plan 02 = すべてのAPI
        Plan 03 = すべてのUI
```

---

## TDDプラン

TDD機能は`type: tdd`の専用プランを得ます。

**ヒューリスティック:** 実装前に`expect(fn(input)).toBe(output)`を書けますか？
→ はい: TDDプランを作成
→ いいえ: 標準プランの標準タスク

詳細は`~/.claude/get-shit-done/references/tdd.md`のTDDプラン構造を参照。

---

## タスクタイプ

| タイプ | 用途 | 自律性 |
|------|---------|----------|
| `auto` | Claudeが独立して実行できるすべて | 完全自律 |
| `checkpoint:human-verify` | 視覚的/機能的な検証 | 一時停止、オーケストレーターに返す |
| `checkpoint:decision` | 実装の選択 | 一時停止、オーケストレーターに返す |
| `checkpoint:human-action` | 本当に避けられない手動ステップ（まれ） | 一時停止、オーケストレーターに返す |

**並列実行でのチェックポイント動作:**
- プランはチェックポイントまで実行
- エージェントがチェックポイントの詳細 + agent_idを返す
- オーケストレーターがユーザーに提示
- ユーザーが応答
- オーケストレーターが`resume: agent_id`でエージェントを再開

---

## 例

**自律的な並列プラン:**

```markdown
---
phase: 03-features
plan: 01
type: execute
wave: 1
depends_on: []
files_modified: [src/features/user/model.ts, src/features/user/api.ts, src/features/user/UserList.tsx]
autonomous: true
---

<objective>
完全なUser機能を垂直スライスとして実装。

Purpose: 他の機能と並列で実行できる自己完結型のユーザー管理。
Output: Userモデル、APIエンドポイント、UIコンポーネント。
</objective>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md
</context>

<tasks>
<task type="auto">
  <name>Task 1: Userモデルの作成</name>
  <files>src/features/user/model.ts</files>
  <action>id、email、name、createdAtを持つUser型を定義。TypeScriptインターフェースをエクスポート。</action>
  <verify>tsc --noEmit passes</verify>
  <done>User型がエクスポートされ使用可能</done>
</task>

<task type="auto">
  <name>Task 2: User APIエンドポイントの作成</name>
  <files>src/features/user/api.ts</files>
  <action>GET /users（一覧）、GET /users/:id（単体）、POST /users（作成）。モデルのUser型を使用。</action>
  <verify>すべてのエンドポイントのcurlテストが通過</verify>
  <done>すべてのCRUD操作が動作</done>
</task>
</tasks>

<verification>
- [ ] npm run buildが成功
- [ ] APIエンドポイントが正しく応答
</verification>

<success_criteria>
- すべてのタスクが完了
- User機能がエンドツーエンドで動作
</success_criteria>

<output>
完了後、`.planning/phases/03-features/03-01-SUMMARY.md`を作成
</output>
```

**チェックポイント付きプラン（非自律的）:**

```markdown
---
phase: 03-features
plan: 03
type: execute
wave: 2
depends_on: ["03-01", "03-02"]
files_modified: [src/components/Dashboard.tsx]
autonomous: false
---

<objective>
視覚的検証付きダッシュボードの構築。

Purpose: ユーザーと製品機能を統合ビューに統合。
Output: 動作するダッシュボードコンポーネント。
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/execute-plan.md
@~/.claude/get-shit-done/templates/summary.md
@~/.claude/get-shit-done/references/checkpoints.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/phases/03-features/03-01-SUMMARY.md
@.planning/phases/03-features/03-02-SUMMARY.md
</context>

<tasks>
<task type="auto">
  <name>Task 1: ダッシュボードレイアウトの構築</name>
  <files>src/components/Dashboard.tsx</files>
  <action>UserListとProductListコンポーネントを使用したレスポンシブグリッドを作成。スタイリングにTailwindを使用。</action>
  <verify>npm run buildが成功</verify>
  <done>ダッシュボードがエラーなしでレンダリング</done>
</task>

<!-- チェックポイントパターン: Claudeがサーバーを起動、ユーザーがURLにアクセス。完全なパターンはcheckpoints.mdを参照。 -->
<task type="auto">
  <name>開発サーバーの起動</name>
  <action>バックグラウンドで`npm run dev`を実行、準備完了を待つ</action>
  <verify>curl localhost:3000が200を返す</verify>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>ダッシュボード - サーバーがhttp://localhost:3000で稼働中</what-built>
  <how-to-verify>localhost:3000/dashboardにアクセス。確認: デスクトップグリッド、モバイルスタック、スクロールの問題なし。</how-to-verify>
  <resume-signal>「approved」と入力するか、問題を説明してください</resume-signal>
</task>
</tasks>

<verification>
- [ ] npm run buildが成功
- [ ] 視覚的検証が通過
</verification>

<success_criteria>
- すべてのタスクが完了
- ユーザーが視覚的レイアウトを承認
</success_criteria>

<output>
完了後、`.planning/phases/03-features/03-03-SUMMARY.md`を作成
</output>
```

---

## アンチパターン

**悪い例: 反射的な依存関係チェーン**
```yaml
depends_on: ["03-01"]  # 単に01が02の前に来るから
```

**悪い例: 水平レイヤーグループ化**
```
Plan 01: すべてのモデル
Plan 02: すべてのAPI（01に依存）
Plan 03: すべてのUI（02に依存）
```

**悪い例: 自律フラグの欠落**
```yaml
# チェックポイントがあるのにautonomous: falseがない
depends_on: []
files_modified: [...]
# autonomous: ???  <- 欠落！
```

**悪い例: 曖昧なタスク**
```xml
<task type="auto">
  <name>認証のセットアップ</name>
  <action>アプリに認証を追加</action>
</task>
```

**Bad: Missing read_first (executor modifies files it hasn't read)**
```xml
<task type="auto">
  <name>Update database config</name>
  <files>src/config/database.ts</files>
  <!-- No read_first! Executor doesn't know current state or conventions -->
  <action>Update the database config to match production settings</action>
</task>
```

**Bad: Vague acceptance criteria (not verifiable)**
```xml
<acceptance_criteria>
  - Config is properly set up
  - Database connection works correctly
</acceptance_criteria>
```

**Good: Concrete with read_first + verifiable criteria**
```xml
<task type="auto">
  <name>Update database config for connection pooling</name>
  <files>src/config/database.ts</files>
  <read_first>src/config/database.ts, .env.example, docker-compose.yml</read_first>
  <action>Add pool configuration: min=2, max=20, idleTimeoutMs=30000. Add SSL config: rejectUnauthorized=true when NODE_ENV=production. Add .env.example entry: DATABASE_POOL_MAX=20.</action>
  <acceptance_criteria>
    - database.ts contains "max: 20" and "idleTimeoutMillis: 30000"
    - database.ts contains SSL conditional on NODE_ENV
    - .env.example contains DATABASE_POOL_MAX
  </acceptance_criteria>
</task>
```

---

## ガイドライン

- Claudeのパース用に常にXML構造を使用
- すべてのプランに`wave`、`depends_on`、`files_modified`、`autonomous`を含める
- 水平レイヤーより垂直スライスを優先
- 本当に必要な場合のみ前のSUMMARYを参照
- 関連するautoタスクと同じプランにチェックポイントをグループ化
- プランごとに2〜3タスク、最大約50%のコンテキスト

---

## ユーザーセットアップ（外部サービス）

プランが人間の設定を必要とする外部サービスを導入する場合、フロントマターに宣言:

```yaml
user_setup:
  - service: stripe
    why: "決済処理にはAPIキーが必要"
    env_vars:
      - name: STRIPE_SECRET_KEY
        source: "Stripe Dashboard → Developers → API keys → Secret key"
      - name: STRIPE_WEBHOOK_SECRET
        source: "Stripe Dashboard → Developers → Webhooks → Signing secret"
    dashboard_config:
      - task: "Webhookエンドポイントの作成"
        location: "Stripe Dashboard → Developers → Webhooks → Add endpoint"
        details: "URL: https://[your-domain]/api/webhooks/stripe"
    local_dev:
      - "stripe listen --forward-to localhost:3000/api/webhooks/stripe"
```

**自動化優先ルール:** `user_setup`にはClaudeが文字通りできないことだけを含める:
- アカウント作成（人間のサインアップが必要）
- シークレット取得（ダッシュボードアクセスが必要）
- ダッシュボード設定（ブラウザでの人間の操作が必要）

**含めないもの:** パッケージインストール、コード変更、ファイル作成、Claudeが実行できるCLIコマンド。

**結果:** execute-planがユーザー向けチェックリスト付きの`{phase}-USER-SETUP.md`を生成。

完全なスキーマと例については`~/.claude/get-shit-done/templates/user-setup.md`を参照

---

## Must-Haves（ゴールからの逆算検証）

`must_haves`フィールドはフェーズゴールの達成のために真でなければならないことを定義します。計画中に導出され、実行後に検証されます。

**構造:**

```yaml
must_haves:
  truths:
    - "User can see existing messages"
    - "User can send a message"
    - "Messages persist across refresh"
  artifacts:
    - path: "src/components/Chat.tsx"
      provides: "Message list rendering"
      min_lines: 30
    - path: "src/app/api/chat/route.ts"
      provides: "Message CRUD operations"
      exports: ["GET", "POST"]
    - path: "prisma/schema.prisma"
      provides: "Message model"
      contains: "model Message"
  key_links:
    - from: "src/components/Chat.tsx"
      to: "/api/chat"
      via: "fetch in useEffect"
      pattern: "fetch.*api/chat"
    - from: "src/app/api/chat/route.ts"
      to: "prisma.message"
      via: "database query"
      pattern: "prisma\\.message\\.(find|create)"
```

**フィールドの説明:**

| フィールド | 目的 |
|-------|---------|
| `truths` | ユーザー視点からの観察可能な動作。それぞれテスト可能でなければならない。 |
| `artifacts` | 実際の実装で存在しなければならないファイル。 |
| `artifacts[].path` | プロジェクトルートからの相対ファイルパス。 |
| `artifacts[].provides` | このアーティファクトが提供するもの。 |
| `artifacts[].min_lines` | オプション。実質的とみなされるための最小行数。 |
| `artifacts[].exports` | オプション。検証する期待されるエクスポート。 |
| `artifacts[].contains` | オプション。ファイル内に存在しなければならないパターン。 |
| `key_links` | アーティファクト間の重要な接続。 |
| `key_links[].from` | ソースアーティファクト。 |
| `key_links[].to` | ターゲットアーティファクトまたはエンドポイント。 |
| `key_links[].via` | 接続方法（説明）。 |
| `key_links[].pattern` | オプション。接続の存在を検証する正規表現。 |

**なぜこれが重要か:**

タスク完了 ≠ ゴール達成。タスク「チャットコンポーネントの作成」はプレースホルダーを作成することで完了できます。`must_haves`フィールドは実際に動作しなければならないものを記録し、検証がギャップが複合する前にそれを捕捉できるようにします。

**検証フロー:**

1. plan-phaseがフェーズゴールからmust_havesを導出（ゴールからの逆算）
2. must_havesがPLAN.mdフロントマターに記述される
3. execute-phaseがすべてのプランを実行
4. 検証サブエージェントがmust_havesをコードベースに対してチェック
5. ギャップ発見 → 修正プランの作成 → 実行 → 再検証
6. すべてのmust_havesが通過 → フェーズ完了

検証ロジックについては`~/.claude/get-shit-done/workflows/verify-phase.md`を参照。
</output>