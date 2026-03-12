# 外部インテグレーションテンプレート

`.planning/codebase/INTEGRATIONS.md` 用テンプレート - 外部サービスの依存関係を記録します。

**目的:** このコードベースが通信する外部システムを文書化します。「コードの外部にある依存先」に焦点を当てています。

---

## ファイルテンプレート

```markdown
# 外部インテグレーション

**分析日:** [YYYY-MM-DD]

## API・外部サービス

**決済処理:**
- [サービス] - [用途: 例: "サブスクリプション課金、単発決済"]
  - SDK/Client: [例: "stripe npmパッケージ v14.x"]
  - Auth: [例: "STRIPE_SECRET_KEY環境変数のAPIキー"]
  - Endpoints used: [例: "チェックアウトセッション、Webhook"]

**メール/SMS:**
- [サービス] - [用途: 例: "トランザクションメール"]
  - SDK/Client: [例: "sendgrid/mail v8.x"]
  - Auth: [例: "SENDGRID_API_KEY環境変数のAPIキー"]
  - Templates: [例: "SendGridダッシュボードで管理"]

**外部API:**
- [サービス] - [用途]
  - Integration method: [例: "fetchによるREST API", "GraphQLクライアント"]
  - Auth: [例: "AUTH_TOKEN環境変数のOAuth2トークン"]
  - Rate limits: [該当する場合]

## データストレージ

**データベース:**
- [種類/プロバイダー] - [例: "Supabase上のPostgreSQL"]
  - Connection: [例: "DATABASE_URL環境変数経由"]
  - Client: [例: "Prisma ORM v5.x"]
  - Migrations: [例: "migrations/内のprisma migrate"]

**ファイルストレージ:**
- [サービス] - [例: "ユーザーアップロード用AWS S3"]
  - SDK/Client: [例: "@aws-sdk/client-s3"]
  - Auth: [例: "AWS_*環境変数のIAM認証情報"]
  - Buckets: [例: "prod-uploads, dev-uploads"]

**キャッシュ:**
- [サービス] - [例: "セッションストレージ用Redis"]
  - Connection: [例: "REDIS_URL環境変数"]
  - Client: [例: "ioredis v5.x"]

## 認証・アイデンティティ

**認証プロバイダー:**
- [サービス] - [例: "Supabase Auth", "Auth0", "カスタムJWT"]
  - Implementation: [例: "Supabaseクライアント SDK"]
  - Token storage: [例: "httpOnlyクッキー", "localStorage"]
  - Session management: [例: "JWTリフレッシュトークン"]

**OAuthインテグレーション:**
- [プロバイダー] - [例: "サインイン用Google OAuth"]
  - Credentials: [例: "GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET"]
  - Scopes: [例: "email, profile"]

## モニタリング・可観測性

**エラートラッキング:**
- [サービス] - [例: "Sentry"]
  - DSN: [例: "SENTRY_DSN環境変数"]
  - Release tracking: [例: "SENTRY_RELEASE経由"]

**アナリティクス:**
- [サービス] - [例: "プロダクトアナリティクス用Mixpanel"]
  - Token: [例: "MIXPANEL_TOKEN環境変数"]
  - Events tracked: [例: "ユーザーアクション、ページビュー"]

**ログ:**
- [サービス] - [例: "CloudWatch", "Datadog", "なし（stdoutのみ）"]
  - Integration: [例: "AWS Lambda組み込み"]

## CI/CD・デプロイ

**ホスティング:**
- [プラットフォーム] - [例: "Vercel", "AWS Lambda", "ECS上のDockerコンテナ"]
  - Deployment: [例: "mainブランチプッシュ時に自動"]
  - Environment vars: [例: "Vercelダッシュボードで設定"]

**CIパイプライン:**
- [サービス] - [例: "GitHub Actions"]
  - Workflows: [例: "test.yml, deploy.yml"]
  - Secrets: [例: "GitHubリポジトリシークレットに保存"]

## 環境設定

**開発環境:**
- Required env vars: [重要な変数をリスト]
- Secrets location: [例: ".env.local（gitignore済み）", "1Passwordボールト"]
- Mock/stub services: [例: "Stripeテストモード", "ローカルPostgreSQL"]

**ステージング:**
- Environment-specific differences: [例: "ステージングStripeアカウントを使用"]
- Data: [例: "別のステージングデータベース"]

**本番環境:**
- Secrets management: [例: "Vercel環境変数"]
- Failover/redundancy: [例: "マルチリージョンDBレプリケーション"]

## Webhook・コールバック

**受信:**
- [サービス] - [エンドポイント: 例: "/api/webhooks/stripe"]
  - Verification: [例: "stripe.webhooks.constructEvent経由の署名検証"]
  - Events: [例: "payment_intent.succeeded, customer.subscription.updated"]

**送信:**
- [サービス] - [トリガー]
  - Endpoint: [例: "ユーザーサインアップ時の外部CRM Webhook"]
  - Retry logic: [該当する場合]

---

*インテグレーション監査: [日付]*
*外部サービスの追加/削除時に更新*
```

<good_examples>
```markdown
# 外部インテグレーション

**分析日:** 2025-01-20

## API・外部サービス

**決済処理:**
- Stripe - サブスクリプション課金と単発コース決済
  - SDK/Client: stripe npmパッケージ v14.8
  - Auth: STRIPE_SECRET_KEY環境変数のAPIキー
  - Endpoints used: チェックアウトセッション、カスタマーポータル、Webhook

**メール/SMS:**
- SendGrid - トランザクションメール（領収書、パスワードリセット）
  - SDK/Client: @sendgrid/mail v8.1
  - Auth: SENDGRID_API_KEY環境変数のAPIキー
  - Templates: SendGridダッシュボードで管理（テンプレートIDはコード内）

**外部API:**
- OpenAI API - コースコンテンツ生成
  - Integration method: openai npmパッケージ v4.x経由のREST API
  - Auth: OPENAI_API_KEY環境変数のBearerトークン
  - Rate limits: 3500リクエスト/分（ティア3）

## データストレージ

**データベース:**
- Supabase上のPostgreSQL - プライマリデータストア
  - Connection: DATABASE_URL環境変数経由
  - Client: Prisma ORM v5.8
  - Migrations: prisma/migrations/内のprisma migrate

**ファイルストレージ:**
- Supabase Storage - ユーザーアップロード（プロフィール画像、コース教材）
  - SDK/Client: @supabase/supabase-js v2.x
  - Auth: SUPABASE_SERVICE_ROLE_KEYのサービスロールキー
  - Buckets: avatars（パブリック）、course-materials（プライベート）

**キャッシュ:**
- 現在なし（すべてデータベースクエリ、Redisなし）

## 認証・アイデンティティ

**認証プロバイダー:**
- Supabase Auth - メール/パスワード + OAuth
  - Implementation: サーバーサイドセッション管理を伴うSupabaseクライアントSDK
  - Token storage: @supabase/ssr経由のhttpOnlyクッキー
  - Session management: Supabaseが処理するJWTリフレッシュトークン

**OAuthインテグレーション:**
- Google OAuth - ソーシャルサインイン
  - Credentials: GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET（Supabaseダッシュボード）
  - Scopes: email, profile

## モニタリング・可観測性

**エラートラッキング:**
- Sentry - サーバーおよびクライアントエラー
  - DSN: SENTRY_DSN環境変数
  - Release tracking: SENTRY_RELEASE経由のGitコミットSHA

**アナリティクス:**
- なし（予定: Mixpanel）

**ログ:**
- Vercelログ - stdout/stderrのみ
  - Retention: Proプランで7日間

## CI/CD・デプロイ

**ホスティング:**
- Vercel - Next.jsアプリホスティング
  - Deployment: mainブランチプッシュ時に自動
  - Environment vars: Vercelダッシュボードで設定（.env.exampleと同期）

**CIパイプライン:**
- GitHub Actions - テストと型チェック
  - Workflows: .github/workflows/ci.yml
  - Secrets: 不要（パブリックリポジトリのテストのみ）

## 環境設定

**開発環境:**
- Required env vars: DATABASE_URL, NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY
- Secrets location: .env.local（gitignore済み）、1Passwordボールトでチーム共有
- Mock/stub services: Stripeテストモード、Supabaseローカル開発プロジェクト

**ステージング:**
- 別のSupabaseステージングプロジェクトを使用
- Stripeテストモード
- 同じVercelアカウント、異なる環境

**本番環境:**
- Secrets management: Vercel環境変数
- Database: 毎日バックアップ付きSupabase本番プロジェクト

## Webhook・コールバック

**受信:**
- Stripe - /api/webhooks/stripe
  - Verification: stripe.webhooks.constructEvent経由の署名検証
  - Events: payment_intent.succeeded, customer.subscription.updated, customer.subscription.deleted

**送信:**
- なし

---

*インテグレーション監査: 2025-01-20*
*外部サービスの追加/削除時に更新*
```
</good_examples>

<guidelines>
**INTEGRATIONS.mdに含めるもの:**
- コードが通信する外部サービス
- 認証パターン（シークレットの保管場所、シークレット自体ではない）
- 使用されるSDKとクライアントライブラリ
- 環境変数名（値ではない）
- Webhookエンドポイントと検証方法
- データベース接続パターン
- ファイルストレージの場所
- モニタリングとロギングサービス

**ここに含めないもの:**
- 実際のAPIキーやシークレット（絶対に書かない）
- 内部アーキテクチャ（ARCHITECTURE.mdの範囲）
- コードパターン（PATTERNS.mdの範囲）
- 技術選定（STACK.mdの範囲）
- パフォーマンスの問題（CONCERNS.mdの範囲）

**このテンプレートを記入する際:**
- .env.exampleまたは.env.templateで必要な環境変数を確認
- SDKインポートを探す（stripe, @sendgrid/mail等）
- ルート/エンドポイントでWebhookハンドラーを確認
- シークレットの管理場所を記録（シークレット自体ではない）
- 環境固有の違いを文書化（開発/ステージング/本番）
- 各サービスの認証パターンを含める

**フェーズ計画で役立つ場面:**
- 新しい外部サービスインテグレーションの追加
- 認証の問題のデバッグ
- アプリケーション外部のデータフローの理解
- 新しい環境のセットアップ
- サードパーティ依存関係の監査
- サービス障害や移行の計画

**セキュリティに関する注意:**
シークレットが「どこに」あるか（環境変数、Vercelダッシュボード、1Password）を文書化し、シークレットが「何であるか」は絶対に記載しない。
</guidelines>
