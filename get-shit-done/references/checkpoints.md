<overview>
プランは自律的に実行されます。チェックポイントは、人間による検証や意思決定が必要なインタラクションポイントを形式化します。

**基本原則:** ClaudeはCLI/APIですべてを自動化します。チェックポイントは検証と意思決定のためであり、手動作業のためではありません。

**ゴールデンルール:**
1. **Claudeが実行できるなら、Claudeが実行する** - ユーザーにCLIコマンドの実行、サーバーの起動、ビルドの実行を依頼しない
2. **Claudeが検証環境をセットアップする** - 開発サーバーの起動、データベースのシード、環境変数の設定
3. **ユーザーは人間の判断が必要なことだけを行う** - 視覚的チェック、UX評価、「これで正しいか？」
4. **シークレットはユーザーから、自動化はClaudeから** - APIキーを聞いて、ClaudeがCLIで使用する
5. **自動モードは検証/意思決定チェックポイントをバイパスする** — `workflow._auto_chain_active`または`workflow.auto_advance`がconfigでtrueの場合：human-verifyは自動承認、decisionは最初のオプションを自動選択、human-actionは停止する（認証ゲートは自動化できない）
</overview>

<checkpoint_types>

<type name="human-verify">
## checkpoint:human-verify（最も一般的 - 90%）

**使用タイミング:** Claudeが自動化作業を完了し、人間が正しく動作するか確認する。

**使用対象:**
- 視覚的なUIチェック（レイアウト、スタイリング、レスポンシブ対応）
- インタラクティブフロー（ウィザードのクリック操作、ユーザーフローのテスト）
- 機能検証（機能が期待通りに動作するか）
- オーディオ/ビデオの再生品質
- アニメーションの滑らかさ
- アクセシビリティテスト

**構造:**
```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[Claudeが自動化・デプロイ/ビルドした内容]</what-built>
  <how-to-verify>
    [テストの正確な手順 - URL、コマンド、期待される動作]
  </how-to-verify>
  <resume-signal>[続行方法 - "approved"、"yes"、または問題を説明]</resume-signal>
</task>
```

**例: UIコンポーネント（重要なパターン：Claudeがチェックポイントの前にサーバーを起動する）**
```xml
<task type="auto">
  <name>Build responsive dashboard layout</name>
  <files>src/components/Dashboard.tsx, src/app/dashboard/page.tsx</files>
  <action>Create dashboard with sidebar, header, and content area. Use Tailwind responsive classes for mobile.</action>
  <verify>npm run build succeeds, no TypeScript errors</verify>
  <done>Dashboard component builds without errors</done>
</task>

<task type="auto">
  <name>Start dev server for verification</name>
  <action>Run `npm run dev` in background, wait for "ready" message, capture port</action>
  <verify>curl http://localhost:3000 returns 200</verify>
  <done>Dev server running at http://localhost:3000</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Responsive dashboard layout - dev server running at http://localhost:3000</what-built>
  <how-to-verify>
    Visit http://localhost:3000/dashboard and verify:
    1. Desktop (>1024px): Sidebar left, content right, header top
    2. Tablet (768px): Sidebar collapses to hamburger menu
    3. Mobile (375px): Single column layout, bottom nav appears
    4. No layout shift or horizontal scroll at any size
  </how-to-verify>
  <resume-signal>Type "approved" or describe layout issues</resume-signal>
</task>
```

**例: Xcodeビルド**
```xml
<task type="auto">
  <name>Build macOS app with Xcode</name>
  <files>App.xcodeproj, Sources/</files>
  <action>Run `xcodebuild -project App.xcodeproj -scheme App build`. Check for compilation errors in output.</action>
  <verify>Build output contains "BUILD SUCCEEDED", no errors</verify>
  <done>App builds successfully</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Built macOS app at DerivedData/Build/Products/Debug/App.app</what-built>
  <how-to-verify>
    Open App.app and test:
    - App launches without crashes
    - Menu bar icon appears
    - Preferences window opens correctly
    - No visual glitches or layout issues
  </how-to-verify>
  <resume-signal>Type "approved" or describe issues</resume-signal>
</task>
```
</type>

<type name="decision">
## checkpoint:decision（9%）

**使用タイミング:** 人間が実装方針に影響する選択をしなければならない。

**使用対象:**
- 技術選定（どの認証プロバイダー、どのデータベース）
- アーキテクチャの決定（モノレポかリポジトリ分割か）
- デザインの選択（カラースキーム、レイアウトアプローチ）
- 機能の優先順位付け（どのバリアントをビルドするか）
- データモデルの決定（スキーマ構造）

**構造:**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>[決定事項]</decision>
  <context>[この決定が重要な理由]</context>
  <options>
    <option id="option-a">
      <name>[オプション名]</name>
      <pros>[メリット]</pros>
      <cons>[トレードオフ]</cons>
    </option>
    <option id="option-b">
      <name>[オプション名]</name>
      <pros>[メリット]</pros>
      <cons>[トレードオフ]</cons>
    </option>
  </options>
  <resume-signal>[選択の示し方]</resume-signal>
</task>
```

**例: 認証プロバイダーの選択**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>Select authentication provider</decision>
  <context>
    Need user authentication for the app. Three solid options with different tradeoffs.
  </context>
  <options>
    <option id="supabase">
      <name>Supabase Auth</name>
      <pros>Built-in with Supabase DB we're using, generous free tier, row-level security integration</pros>
      <cons>Less customizable UI, tied to Supabase ecosystem</cons>
    </option>
    <option id="clerk">
      <name>Clerk</name>
      <pros>Beautiful pre-built UI, best developer experience, excellent docs</pros>
      <cons>Paid after 10k MAU, vendor lock-in</cons>
    </option>
    <option id="nextauth">
      <name>NextAuth.js</name>
      <pros>Free, self-hosted, maximum control, widely adopted</pros>
      <cons>More setup work, you manage security updates, UI is DIY</cons>
    </option>
  </options>
  <resume-signal>Select: supabase, clerk, or nextauth</resume-signal>
</task>
```

**例: データベースの選択**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>Select database for user data</decision>
  <context>
    App needs persistent storage for users, sessions, and user-generated content.
    Expected scale: 10k users, 1M records first year.
  </context>
  <options>
    <option id="supabase">
      <name>Supabase (Postgres)</name>
      <pros>Full SQL, generous free tier, built-in auth, real-time subscriptions</pros>
      <cons>Vendor lock-in for real-time features, less flexible than raw Postgres</cons>
    </option>
    <option id="planetscale">
      <name>PlanetScale (MySQL)</name>
      <pros>Serverless scaling, branching workflow, excellent DX</pros>
      <cons>MySQL not Postgres, no foreign keys in free tier</cons>
    </option>
    <option id="convex">
      <name>Convex</name>
      <pros>Real-time by default, TypeScript-native, automatic caching</pros>
      <cons>Newer platform, different mental model, less SQL flexibility</cons>
    </option>
  </options>
  <resume-signal>Select: supabase, planetscale, or convex</resume-signal>
</task>
```
</type>

<type name="human-action">
## checkpoint:human-action（1% - まれ）

**使用タイミング:** アクションにCLI/APIがなく人間のみの操作が必要、またはClaudeが自動化中に認証ゲートにヒットした。

**使用対象（これらのみ）:**
- **認証ゲート** - ClaudeがCLI/APIを試みたが認証情報が必要（これは失敗ではない）
- メール認証リンク（メールのクリック）
- SMS 2FAコード（電話認証）
- 手動アカウント承認（プラットフォームが人間のレビューを要求）
- クレジットカード3Dセキュアフロー（Webベースの支払い認可）
- OAuthアプリの承認（Webベースの承認）

**事前に計画された手動作業には使用しない:**
- デプロイ（CLIを使用 - 必要に応じて認証ゲート）
- Webhook/データベースの作成（API/CLIを使用 - 必要に応じて認証ゲート）
- ビルド/テストの実行（Bashツールを使用）
- ファイルの作成（Writeツールを使用）

**構造:**
```xml
<task type="checkpoint:human-action" gate="blocking">
  <action>[人間が行うべきこと - Claudeは自動化可能なすべてを完了済み]</action>
  <instructions>
    [Claudeが既に自動化した内容]
    [人間のアクションが必要な一つのこと]
  </instructions>
  <verification>[Claudeが後で確認できること]</verification>
  <resume-signal>[続行方法]</resume-signal>
</task>
```

**例: メール認証**
```xml
<task type="auto">
  <name>Create SendGrid account via API</name>
  <action>Use SendGrid API to create subuser account with provided email. Request verification email.</action>
  <verify>API returns 201, account created</verify>
  <done>Account created, verification email sent</done>
</task>

<task type="checkpoint:human-action" gate="blocking">
  <action>Complete email verification for SendGrid account</action>
  <instructions>
    I created the account and requested verification email.
    Check your inbox for SendGrid verification link and click it.
  </instructions>
  <verification>SendGrid API key works: curl test succeeds</verification>
  <resume-signal>Type "done" when email verified</resume-signal>
</task>
```

**例: 認証ゲート（動的チェックポイント）**
```xml
<task type="auto">
  <name>Deploy to Vercel</name>
  <files>.vercel/, vercel.json</files>
  <action>Run `vercel --yes` to deploy</action>
  <verify>vercel ls shows deployment, curl returns 200</verify>
</task>

<!-- vercelが"Error: Not authenticated"を返した場合、Claudeがその場でチェックポイントを作成 -->

<task type="checkpoint:human-action" gate="blocking">
  <action>Authenticate Vercel CLI so I can continue deployment</action>
  <instructions>
    I tried to deploy but got authentication error.
    Run: vercel login
    This will open your browser - complete the authentication flow.
  </instructions>
  <verification>vercel whoami returns your account email</verification>
  <resume-signal>Type "done" when authenticated</resume-signal>
</task>

<!-- 認証後、Claudeがデプロイを再試行 -->

<task type="auto">
  <name>Retry Vercel deployment</name>
  <action>Run `vercel --yes` (now authenticated)</action>
  <verify>vercel ls shows deployment, curl returns 200</verify>
</task>
```

**重要な区別:** 認証ゲートは、Claudeが認証エラーに遭遇したときに動的に作成されます。事前に計画されるものではなく、Claudeがまず自動化を試み、ブロックされた場合にのみ認証情報を求めます。
</type>
</checkpoint_types>

<execution_protocol>

Claudeが`type="checkpoint:*"`に遭遇した場合:

1. **直ちに停止する** - 次のタスクに進まない
2. **チェックポイントを明確に表示する** - 以下のフォーマットを使用
3. **ユーザーの応答を待つ** - 完了をハルシネーションしない
4. **可能であれば検証する** - ファイルの確認、テストの実行、指定された内容
5. **実行を再開する** - 確認後にのみ次のタスクに進む

**checkpoint:human-verifyの場合:**
```
╔═══════════════════════════════════════════════════════╗
║  CHECKPOINT: Verification Required                    ║
╚═══════════════════════════════════════════════════════╝

Progress: 5/8 tasks complete
Task: Responsive dashboard layout

Built: Responsive dashboard at /dashboard

How to verify:
  1. Visit: http://localhost:3000/dashboard
  2. Desktop (>1024px): Sidebar visible, content fills remaining space
  3. Tablet (768px): Sidebar collapses to icons
  4. Mobile (375px): Sidebar hidden, hamburger menu appears

────────────────────────────────────────────────────────
→ YOUR ACTION: Type "approved" or describe issues
────────────────────────────────────────────────────────
```

**checkpoint:decisionの場合:**
```
╔═══════════════════════════════════════════════════════╗
║  CHECKPOINT: Decision Required                        ║
╚═══════════════════════════════════════════════════════╝

Progress: 2/6 tasks complete
Task: Select authentication provider

Decision: Which auth provider should we use?

Context: Need user authentication. Three options with different tradeoffs.

Options:
  1. supabase - Built-in with our DB, free tier
     Pros: Row-level security integration, generous free tier
     Cons: Less customizable UI, ecosystem lock-in

  2. clerk - Best DX, paid after 10k users
     Pros: Beautiful pre-built UI, excellent documentation
     Cons: Vendor lock-in, pricing at scale

  3. nextauth - Self-hosted, maximum control
     Pros: Free, no vendor lock-in, widely adopted
     Cons: More setup work, DIY security updates

────────────────────────────────────────────────────────
→ YOUR ACTION: Select supabase, clerk, or nextauth
────────────────────────────────────────────────────────
```

**checkpoint:human-actionの場合:**
```
╔═══════════════════════════════════════════════════════╗
║  CHECKPOINT: Action Required                          ║
╚═══════════════════════════════════════════════════════╝

Progress: 3/8 tasks complete
Task: Deploy to Vercel

Attempted: vercel --yes
Error: Not authenticated. Please run 'vercel login'

What you need to do:
  1. Run: vercel login
  2. Complete browser authentication when it opens
  3. Return here when done

I'll verify: vercel whoami returns your account

────────────────────────────────────────────────────────
→ YOUR ACTION: Type "done" when authenticated
────────────────────────────────────────────────────────
```
</execution_protocol>

<authentication_gates>

**認証ゲート = ClaudeがCLI/APIを試みて、認証エラーが発生した。** 失敗ではなく、ブロック解除のために人間の入力が必要なゲートです。

**パターン:** Claudeが自動化を試みる → 認証エラー → checkpoint:human-actionを作成 → ユーザーが認証 → Claudeが再試行 → 続行

**ゲートプロトコル:**
1. 失敗ではないことを認識する - 認証がないのは想定内
2. 現在のタスクを停止する - 繰り返しリトライしない
3. checkpoint:human-actionを動的に作成する
4. 正確な認証手順を提供する
5. 認証が機能することを確認する
6. 元のタスクを再試行する
7. 通常通り続行する

**重要な区別:**
- 事前計画されたチェックポイント: 「Xをしてください」（間違い - Claudeが自動化すべき）
- 認証ゲート: 「Xを自動化しようとしましたが認証情報が必要です」（正しい - 自動化のブロック解除）

</authentication_gates>

<automation_reference>

**ルール:** CLI/APIがあれば、Claudeが実行する。自動化可能な作業を人間に依頼しない。

## サービスCLIリファレンス

| サービス | CLI/API | 主要コマンド | 認証ゲート |
|---------|---------|--------------|-----------|
| Vercel | `vercel` | `--yes`, `env add`, `--prod`, `ls` | `vercel login` |
| Railway | `railway` | `init`, `up`, `variables set` | `railway login` |
| Fly | `fly` | `launch`, `deploy`, `secrets set` | `fly auth login` |
| Stripe | `stripe` + API | `listen`, `trigger`, API calls | API key in .env |
| Supabase | `supabase` | `init`, `link`, `db push`, `gen types` | `supabase login` |
| Upstash | `upstash` | `redis create`, `redis get` | `upstash auth login` |
| PlanetScale | `pscale` | `database create`, `branch create` | `pscale auth login` |
| GitHub | `gh` | `repo create`, `pr create`, `secret set` | `gh auth login` |
| Node | `npm`/`pnpm` | `install`, `run build`, `test`, `run dev` | N/A |
| Xcode | `xcodebuild` | `-project`, `-scheme`, `build`, `test` | N/A |
| Convex | `npx convex` | `dev`, `deploy`, `env set`, `env get` | `npx convex login` |

## 環境変数の自動化

**環境ファイル:** Write/Editツールを使用する。ユーザーに手動で.envを作成させない。

**CLIによるダッシュボード環境変数:**

| プラットフォーム | CLIコマンド | 例 |
|----------|-------------|---------|
| Convex | `npx convex env set` | `npx convex env set OPENAI_API_KEY sk-...` |
| Vercel | `vercel env add` | `vercel env add STRIPE_KEY production` |
| Railway | `railway variables set` | `railway variables set API_KEY=value` |
| Fly | `fly secrets set` | `fly secrets set DATABASE_URL=...` |
| Supabase | `supabase secrets set` | `supabase secrets set MY_SECRET=value` |

**シークレット収集パターン:**
```xml
<!-- 間違い: ダッシュボードで環境変数を追加するようユーザーに依頼 -->
<task type="checkpoint:human-action">
  <action>Add OPENAI_API_KEY to Convex dashboard</action>
  <instructions>Go to dashboard.convex.dev → Settings → Environment Variables → Add</instructions>
</task>

<!-- 正しい: Claudeが値を聞いて、CLIで追加 -->
<task type="checkpoint:human-action">
  <action>Provide your OpenAI API key</action>
  <instructions>
    I need your OpenAI API key for Convex backend.
    Get it from: https://platform.openai.com/api-keys
    Paste the key (starts with sk-)
  </instructions>
  <verification>I'll add it via `npx convex env set` and verify</verification>
  <resume-signal>Paste your API key</resume-signal>
</task>

<task type="auto">
  <name>Configure OpenAI key in Convex</name>
  <action>Run `npx convex env set OPENAI_API_KEY {user-provided-key}`</action>
  <verify>`npx convex env get OPENAI_API_KEY` returns the key (masked)</verify>
</task>
```

## 開発サーバーの自動化

| フレームワーク | 起動コマンド | 準備完了シグナル | デフォルトURL |
|-----------|---------------|--------------|-------------|
| Next.js | `npm run dev` | "Ready in" or "started server" | http://localhost:3000 |
| Vite | `npm run dev` | "ready in" | http://localhost:5173 |
| Convex | `npx convex dev` | "Convex functions ready" | N/A（バックエンドのみ） |
| Express | `npm start` | "listening on port" | http://localhost:3000 |
| Django | `python manage.py runserver` | "Starting development server" | http://localhost:8000 |

**サーバーのライフサイクル:**
```bash
# バックグラウンドで実行、PIDを取得
npm run dev &
DEV_SERVER_PID=$!

# 準備完了を待つ（最大30秒）
timeout 30 bash -c 'until curl -s localhost:3000 > /dev/null 2>&1; do sleep 1; done'
```

**ポート競合:** 古いプロセスをkill（`lsof -ti:3000 | xargs kill`）するか、別のポートを使用する（`--port 3001`）。

**サーバーはチェックポイント中も実行し続ける。** プランが完了した時、本番環境に切り替える時、またはポートが別のサービスに必要な時のみkillする。

## CLIインストール処理

| CLI | 自動インストール? | コマンド |
|-----|---------------|---------|
| npm/pnpm/yarn | いいえ - ユーザーに確認 | ユーザーがパッケージマネージャーを選択 |
| vercel | はい | `npm i -g vercel` |
| gh (GitHub) | はい | `brew install gh`（macOS）または`apt install gh`（Linux） |
| stripe | はい | `npm i -g stripe` |
| supabase | はい | `npm i -g supabase` |
| convex | いいえ - npxを使用 | `npx convex`（インストール不要） |
| fly | はい | `brew install flyctl`またはcurlインストーラー |
| railway | はい | `npm i -g @railway/cli` |

**プロトコル:** コマンド実行 → "command not found" → 自動インストール可能？ → はい: サイレントインストール、再試行 → いいえ: ユーザーにインストールを求めるチェックポイント。

## チェックポイント前の自動化失敗

| 失敗 | 対応 |
|---------|----------|
| サーバーが起動しない | エラーを確認、問題を修正、再試行（チェックポイントに進まない） |
| ポート使用中 | 古いプロセスをkillするか別のポートを使用 |
| 依存関係が不足 | `npm install`を実行、再試行 |
| ビルドエラー | まずエラーを修正（バグであり、チェックポイントの問題ではない） |
| 認証エラー | 認証ゲートチェックポイントを作成 |
| ネットワークタイムアウト | バックオフで再試行、持続する場合はチェックポイント |

**壊れた検証環境でチェックポイントを提示しない。** `curl localhost:3000`が失敗するなら、ユーザーに「localhost:3000にアクセスしてください」と依頼しない。

```xml
<!-- 間違い: 壊れた環境でのチェックポイント -->
<task type="checkpoint:human-verify">
  <what-built>Dashboard (server failed to start)</what-built>
  <how-to-verify>Visit http://localhost:3000...</how-to-verify>
</task>

<!-- 正しい: まず修正してからチェックポイント -->
<task type="auto">
  <name>Fix server startup issue</name>
  <action>Investigate error, fix root cause, restart server</action>
  <verify>curl http://localhost:3000 returns 200</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Dashboard - server running at http://localhost:3000</what-built>
  <how-to-verify>Visit http://localhost:3000/dashboard...</how-to-verify>
</task>
```

## 自動化可能なクイックリファレンス

| アクション | 自動化可能? | Claudeが実行? |
|--------|--------------|-----------------|
| Vercelにデプロイ | はい（`vercel`） | はい |
| Stripe webhookの作成 | はい（API） | はい |
| .envファイルの記述 | はい（Writeツール） | はい |
| Upstash DBの作成 | はい（`upstash`） | はい |
| テストの実行 | はい（`npm test`） | はい |
| 開発サーバーの起動 | はい（`npm run dev`） | はい |
| Convexへの環境変数追加 | はい（`npx convex env set`） | はい |
| Vercelへの環境変数追加 | はい（`vercel env add`） | はい |
| データベースのシード | はい（CLI/API） | はい |
| メール認証リンクのクリック | いいえ | いいえ |
| 3DSクレジットカード入力 | いいえ | いいえ |
| ブラウザでのOAuth完了 | いいえ | いいえ |
| UIが正しく見えるかの視覚的検証 | いいえ | いいえ |
| インタラクティブなユーザーフローのテスト | いいえ | いいえ |

</automation_reference>

<writing_guidelines>

**すべきこと:**
- チェックポイントの前にCLI/APIですべてを自動化する
- 具体的に書く: 「デプロイメントを確認」ではなく「https://myapp.vercel.app にアクセス」
- 検証ステップに番号を付ける
- 期待される結果を記述する: 「Xが表示されるはずです」
- コンテキストを提供する: なぜこのチェックポイントが存在するか

**すべきでないこと:**
- Claudeが自動化できる作業を人間に依頼する
- 知識を前提にする: 「通常の設定を行ってください」
- ステップを省略する: 「データベースをセットアップ」（曖昧すぎる）
- 複数の検証を1つのチェックポイントに混在させる

**配置:**
- **自動化完了後** - Claudeが作業を行う前ではない
- **UIビルドアウト後** - フェーズ完了を宣言する前
- **依存する作業の前** - 実装前の意思決定
- **統合ポイント** - 外部サービスの設定後

**悪い配置:** 自動化の前 | 頻度が高すぎる | 遅すぎる（依存タスクが既に結果を必要としていた）
</writing_guidelines>

<examples>

### 例1: データベースセットアップ（チェックポイント不要）

```xml
<task type="auto">
  <name>Create Upstash Redis database</name>
  <files>.env</files>
  <action>
    1. Run `upstash redis create myapp-cache --region us-east-1`
    2. Capture connection URL from output
    3. Write to .env: UPSTASH_REDIS_URL={url}
    4. Verify connection with test command
  </action>
  <verify>
    - upstash redis list shows database
    - .env contains UPSTASH_REDIS_URL
    - Test connection succeeds
  </verify>
  <done>Redis database created and configured</done>
</task>

<!-- チェックポイント不要 - Claudeがすべてを自動化し、プログラム的に検証済み -->
```

### 例2: 完全な認証フロー（最後に1つのチェックポイント）

```xml
<task type="auto">
  <name>Create user schema</name>
  <files>src/db/schema.ts</files>
  <action>Define User, Session, Account tables with Drizzle ORM</action>
  <verify>npm run db:generate succeeds</verify>
</task>

<task type="auto">
  <name>Create auth API routes</name>
  <files>src/app/api/auth/[...nextauth]/route.ts</files>
  <action>Set up NextAuth with GitHub provider, JWT strategy</action>
  <verify>TypeScript compiles, no errors</verify>
</task>

<task type="auto">
  <name>Create login UI</name>
  <files>src/app/login/page.tsx, src/components/LoginButton.tsx</files>
  <action>Create login page with GitHub OAuth button</action>
  <verify>npm run build succeeds</verify>
</task>

<task type="auto">
  <name>Start dev server for auth testing</name>
  <action>Run `npm run dev` in background, wait for ready signal</action>
  <verify>curl http://localhost:3000 returns 200</verify>
  <done>Dev server running at http://localhost:3000</done>
</task>

<!-- 最後に1つのチェックポイントで完全なフローを検証 -->
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Complete authentication flow - dev server running at http://localhost:3000</what-built>
  <how-to-verify>
    1. Visit: http://localhost:3000/login
    2. Click "Sign in with GitHub"
    3. Complete GitHub OAuth flow
    4. Verify: Redirected to /dashboard, user name displayed
    5. Refresh page: Session persists
    6. Click logout: Session cleared
  </how-to-verify>
  <resume-signal>Type "approved" or describe issues</resume-signal>
</task>
```
</examples>

<anti_patterns>

### 悪い例: ユーザーに開発サーバーの起動を依頼

```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Dashboard component</what-built>
  <how-to-verify>
    1. Run: npm run dev
    2. Visit: http://localhost:3000/dashboard
    3. Check layout is correct
  </how-to-verify>
</task>
```

**なぜ悪いか:** Claudeが`npm run dev`を実行できる。ユーザーはURLにアクセスするだけで、コマンドを実行すべきではない。

### 良い例: Claudeがサーバーを起動、ユーザーがアクセス

```xml
<task type="auto">
  <name>Start dev server</name>
  <action>Run `npm run dev` in background</action>
  <verify>curl localhost:3000 returns 200</verify>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Dashboard at http://localhost:3000/dashboard (server running)</what-built>
  <how-to-verify>
    Visit http://localhost:3000/dashboard and verify:
    1. Layout matches design
    2. No console errors
  </how-to-verify>
</task>
```

### 悪い例: 人間にデプロイを依頼 / 良い例: Claudeが自動化

```xml
<!-- 悪い: ダッシュボードでデプロイするようユーザーに依頼 -->
<task type="checkpoint:human-action" gate="blocking">
  <action>Deploy to Vercel</action>
  <instructions>Visit vercel.com/new → Import repo → Click Deploy → Copy URL</instructions>
</task>

<!-- 良い: Claudeがデプロイ、ユーザーが検証 -->
<task type="auto">
  <name>Deploy to Vercel</name>
  <action>Run `vercel --yes`. Capture URL.</action>
  <verify>vercel ls shows deployment, curl returns 200</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Deployed to {url}</what-built>
  <how-to-verify>Visit {url}, check homepage loads</how-to-verify>
  <resume-signal>Type "approved"</resume-signal>
</task>
```

### 悪い例: チェックポイントが多すぎる / 良い例: 単一チェックポイント

```xml
<!-- 悪い: 各タスクの後にチェックポイント -->
<task type="auto">Create schema</task>
<task type="checkpoint:human-verify">Check schema</task>
<task type="auto">Create API route</task>
<task type="checkpoint:human-verify">Check API</task>
<task type="auto">Create UI form</task>
<task type="checkpoint:human-verify">Check form</task>

<!-- 良い: 最後に1つのチェックポイント -->
<task type="auto">Create schema</task>
<task type="auto">Create API route</task>
<task type="auto">Create UI form</task>

<task type="checkpoint:human-verify">
  <what-built>Complete auth flow (schema + API + UI)</what-built>
  <how-to-verify>Test full flow: register, login, access protected page</how-to-verify>
  <resume-signal>Type "approved"</resume-signal>
</task>
```

### 悪い例: 曖昧な検証 / 良い例: 具体的な手順

```xml
<!-- 悪い -->
<task type="checkpoint:human-verify">
  <what-built>Dashboard</what-built>
  <how-to-verify>Check it works</how-to-verify>
</task>

<!-- 良い -->
<task type="checkpoint:human-verify">
  <what-built>Responsive dashboard - server running at http://localhost:3000</what-built>
  <how-to-verify>
    Visit http://localhost:3000/dashboard and verify:
    1. Desktop (>1024px): Sidebar visible, content area fills remaining space
    2. Tablet (768px): Sidebar collapses to icons
    3. Mobile (375px): Sidebar hidden, hamburger menu in header
    4. No horizontal scroll at any size
  </how-to-verify>
  <resume-signal>Type "approved" or describe layout issues</resume-signal>
</task>
```

### 悪い例: ユーザーにCLIコマンドの実行を依頼

```xml
<task type="checkpoint:human-action">
  <action>Run database migrations</action>
  <instructions>Run: npx prisma migrate deploy && npx prisma db seed</instructions>
</task>
```

**なぜ悪いか:** Claudeがこれらのコマンドを実行できる。ユーザーがCLIコマンドを実行すべきではない。

### 悪い例: ユーザーにサービス間で値のコピーを依頼

```xml
<task type="checkpoint:human-action">
  <action>Configure webhook URL in Stripe</action>
  <instructions>Copy deployment URL → Stripe Dashboard → Webhooks → Add endpoint → Copy secret → Add to .env</instructions>
</task>
```

**なぜ悪いか:** StripeにはAPIがある。ClaudeがAPI経由でwebhookを作成し、直接.envに書き込むべき。

</anti_patterns>

<summary>

チェックポイントは、手動作業ではなく、検証と意思決定のための人間参加型ポイントを形式化します。

**ゴールデンルール:** Claudeが自動化できるなら、Claudeが自動化しなければならない。

**チェックポイントの優先順位:**
1. **checkpoint:human-verify**（90%）- Claudeがすべてを自動化し、人間が視覚的/機能的な正しさを確認
2. **checkpoint:decision**（9%）- 人間がアーキテクチャ/技術の選択を行う
3. **checkpoint:human-action**（1%）- API/CLIのない本当に避けられない手動ステップ

**チェックポイントを使用しない場合:**
- Claudeがプログラム的に検証できるもの（テスト、ビルド）
- ファイル操作（Claudeがファイルを読める）
- コードの正しさ（テストと静的解析）
- CLI/APIで自動化可能なすべて
</summary>