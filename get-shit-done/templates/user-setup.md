# ユーザーセットアップテンプレート

`.planning/phases/XX-name/{phase}-USER-SETUP.md` 用テンプレート - Claudeが自動化できない人手による設定。

**目的:** 文字通り人手が必要なセットアップタスクを記録 - アカウント作成、ダッシュボード設定、シークレット取得。Claudeは可能なすべてを自動化します。このファイルは残った作業のみを記録します。

---

## ファイルテンプレート

```markdown
# Phase {X}: ユーザーセットアップが必要

**生成日:** [YYYY-MM-DD]
**Phase:** {phase-name}
**Status:** Incomplete

以下の項目を完了してインテグレーションを機能させてください。Claudeは可能なすべてを自動化しました。これらの項目は外部ダッシュボード/アカウントへの人手によるアクセスが必要です。

## 環境変数

| Status | Variable | Source | Add to |
|--------|----------|--------|--------|
| [ ] | `ENV_VAR_NAME` | [サービスダッシュボード → パス → 値] | `.env.local` |
| [ ] | `ANOTHER_VAR` | [サービスダッシュボード → パス → 値] | `.env.local` |

## アカウントセットアップ

[新しいアカウント作成が必要な場合のみ]

- [ ] **[サービス]アカウントを作成**
  - URL: [サインアップURL]
  - スキップ条件: 既にアカウントがある場合

## ダッシュボード設定

[ダッシュボード設定が必要な場合のみ]

- [ ] **[設定タスク]**
  - 場所: [サービスダッシュボード → パス → 設定]
  - 設定値: [必要な値や設定]
  - 備考: [重要な詳細]

## 検証

セットアップ完了後、以下で検証:

```bash
# [検証コマンド]
```

期待される結果:
- [成功した場合の表示]

---

**すべての項目が完了したら:** ファイル先頭のステータスを "Complete" に変更してください。
```

---

## 生成タイミング

プランのフロントマターに `user_setup` フィールドがある場合に `{phase}-USER-SETUP.md` を生成します。

**トリガー:** PLAN.mdフロントマターに `user_setup` が存在し項目がある場合。

**場所:** PLAN.mdおよびSUMMARY.mdと同じディレクトリ。

**タイミング:** execute-plan.md中、タスク完了後、SUMMARY.md作成前に生成。

---

## フロントマタースキーマ

PLAN.mdで `user_setup` は人手による設定を宣言します:

```yaml
user_setup:
  - service: stripe
    why: "Payment processing requires API keys"
    env_vars:
      - name: STRIPE_SECRET_KEY
        source: "Stripe Dashboard → Developers → API keys → Secret key"
      - name: STRIPE_WEBHOOK_SECRET
        source: "Stripe Dashboard → Developers → Webhooks → Signing secret"
    dashboard_config:
      - task: "Create webhook endpoint"
        location: "Stripe Dashboard → Developers → Webhooks → Add endpoint"
        details: "URL: https://[your-domain]/api/webhooks/stripe, Events: checkout.session.completed, customer.subscription.*"
    local_dev:
      - "Run: stripe listen --forward-to localhost:3000/api/webhooks/stripe"
      - "Use the webhook secret from CLI output for local testing"
```

---

## オートメーションファーストルール

**USER-SETUP.mdにはClaudeが文字通りできないことのみを含めます。**

| Claudeができること（USER-SETUPに含めない） | Claudeができないこと（→ USER-SETUP） |
|-----------------------------------|--------------------------------|
| `npm install stripe` | Stripeアカウント作成 |
| Webhookハンドラーコードの作成 | ダッシュボードからAPIキーを取得 |
| `.env.local` ファイル構造の作成 | 実際のシークレット値のコピー |
| `stripe listen` の実行 | Stripe CLIの認証（ブラウザOAuth） |
| package.jsonの設定 | 外部サービスダッシュボードへのアクセス |
| あらゆるコードの作成 | サードパーティシステムからのシークレット取得 |

**判定基準:** "これはブラウザで人間が、Claudeが認証情報を持たないアカウントにアクセスする必要があるか？"
- はい → USER-SETUP.md
- いいえ → Claudeが自動的に実行

---

## サービス固有の例

<stripe_example>
```markdown
# Phase 10: ユーザーセットアップが必要

**生成日:** 2025-01-14
**Phase:** 10-monetization
**Status:** Incomplete

Stripeインテグレーションを機能させるために以下の項目を完了してください。

## 環境変数

| Status | Variable | Source | Add to |
|--------|----------|--------|--------|
| [ ] | `STRIPE_SECRET_KEY` | Stripe Dashboard → Developers → API keys → Secret key | `.env.local` |
| [ ] | `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Stripe Dashboard → Developers → API keys → Publishable key | `.env.local` |
| [ ] | `STRIPE_WEBHOOK_SECRET` | Stripe Dashboard → Developers → Webhooks → [endpoint] → Signing secret | `.env.local` |

## アカウントセットアップ

- [ ] **Stripeアカウントを作成**（必要な場合）
  - URL: https://dashboard.stripe.com/register
  - スキップ条件: 既にStripeアカウントがある場合

## ダッシュボード設定

- [ ] **Webhookエンドポイントを作成**
  - 場所: Stripe Dashboard → Developers → Webhooks → Add endpoint
  - エンドポイントURL: `https://[your-domain]/api/webhooks/stripe`
  - 送信するイベント:
    - `checkout.session.completed`
    - `customer.subscription.created`
    - `customer.subscription.updated`
    - `customer.subscription.deleted`

- [ ] **商品と価格を作成**（サブスクリプションティアを使用する場合）
  - 場所: Stripe Dashboard → Products → Add product
  - 各サブスクリプションティアを作成
  - 価格IDを以下にコピー:
    - `STRIPE_STARTER_PRICE_ID`
    - `STRIPE_PRO_PRICE_ID`

## ローカル開発

ローカルWebhookテスト用:
```bash
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```
CLI出力のWebhook署名シークレット（`whsec_` で始まる）を使用してください。

## 検証

セットアップ完了後:

```bash
# 環境変数の確認
grep STRIPE .env.local

# ビルドの確認
npm run build

# Webhookエンドポイントのテスト（400 bad signatureが返れば正常、500 crashではない）
curl -X POST http://localhost:3000/api/webhooks/stripe \
  -H "Content-Type: application/json" \
  -d '{}'
```

期待される結果: ビルド通過、Webhookが400を返す（署名バリデーションが機能）。

---

**すべての項目が完了したら:** ファイル先頭のステータスを "Complete" に変更してください。
```
</stripe_example>

<supabase_example>
```markdown
# Phase 2: ユーザーセットアップが必要

**生成日:** 2025-01-14
**Phase:** 02-authentication
**Status:** Incomplete

Supabase Authを機能させるために以下の項目を完了してください。

## 環境変数

| Status | Variable | Source | Add to |
|--------|----------|--------|--------|
| [ ] | `NEXT_PUBLIC_SUPABASE_URL` | Supabase Dashboard → Settings → API → Project URL | `.env.local` |
| [ ] | `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase Dashboard → Settings → API → anon public | `.env.local` |
| [ ] | `SUPABASE_SERVICE_ROLE_KEY` | Supabase Dashboard → Settings → API → service_role | `.env.local` |

## アカウントセットアップ

- [ ] **Supabaseプロジェクトを作成**
  - URL: https://supabase.com/dashboard/new
  - スキップ条件: このアプリ用のプロジェクトが既にある場合

## ダッシュボード設定

- [ ] **メール認証を有効化**
  - 場所: Supabase Dashboard → Authentication → Providers
  - 有効化: メールプロバイダー
  - 設定: メール確認（好みに応じてオン/オフ）

- [ ] **OAuthプロバイダーを設定**（ソーシャルログインを使用する場合）
  - 場所: Supabase Dashboard → Authentication → Providers
  - Google: Google Cloud ConsoleからClient IDとSecretを追加
  - GitHub: GitHub OAuth AppsからClient IDとSecretを追加

## 検証

セットアップ完了後:

```bash
# 環境変数の確認
grep SUPABASE .env.local

# 接続の確認（プロジェクトディレクトリで実行）
npx supabase status
```

---

**すべての項目が完了したら:** ファイル先頭のステータスを "Complete" に変更してください。
```
</supabase_example>

<sendgrid_example>
```markdown
# Phase 5: ユーザーセットアップが必要

**生成日:** 2025-01-14
**Phase:** 05-notifications
**Status:** Incomplete

SendGridメールを機能させるために以下の項目を完了してください。

## 環境変数

| Status | Variable | Source | Add to |
|--------|----------|--------|--------|
| [ ] | `SENDGRID_API_KEY` | SendGrid Dashboard → Settings → API Keys → Create API Key | `.env.local` |
| [ ] | `SENDGRID_FROM_EMAIL` | 確認済みの送信者メールアドレス | `.env.local` |

## アカウントセットアップ

- [ ] **SendGridアカウントを作成**
  - URL: https://signup.sendgrid.com/
  - スキップ条件: 既にアカウントがある場合

## ダッシュボード設定

- [ ] **送信者IDを確認**
  - 場所: SendGrid Dashboard → Settings → Sender Authentication
  - オプション1: Single Sender Verification（簡易、開発用）
  - オプション2: Domain Authentication（本番用）

- [ ] **APIキーを作成**
  - 場所: SendGrid Dashboard → Settings → API Keys → Create API Key
  - 権限: Restricted Access → Mail Send (Full Access)
  - キーをすぐにコピー（一度だけ表示されます）

## 検証

セットアップ完了後:

```bash
# 環境変数の確認
grep SENDGRID .env.local

# メール送信テスト（テスト用メールアドレスに置き換え）
curl -X POST http://localhost:3000/api/test-email \
  -H "Content-Type: application/json" \
  -d '{"to": "your@email.com"}'
```

---

**すべての項目が完了したら:** ファイル先頭のステータスを "Complete" に変更してください。
```
</sendgrid_example>

---

## ガイドライン

**含めないもの:** 実際のシークレット値。Claudeが自動化できる手順（パッケージインストール、コード変更）。

**命名規則:** `{phase}-USER-SETUP.md` はフェーズ番号パターンに一致。
**ステータス追跡:** ユーザーがチェックボックスをマークし、完了時にステータス行を更新。
**検索性:** `grep -r "USER-SETUP" .planning/` でユーザー要件があるすべてのフェーズを検索可能。
