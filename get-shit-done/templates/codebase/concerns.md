# コードベース懸念事項テンプレート

`.planning/codebase/CONCERNS.md` 用テンプレート - 既知の問題と注意が必要な領域を記録します。

**目的:** コードベースに関する実用的な警告を提示します。「変更を行う際に注意すべきこと」に焦点を当てています。

---

## ファイルテンプレート

```markdown
# コードベース懸念事項

**分析日:** [YYYY-MM-DD]

## 技術的負債

**[領域/コンポーネント]:**
- Issue: [ショートカット/回避策の内容]
- Why: [なぜこのようにされたか]
- Impact: [何が壊れるか、または劣化するか]
- Fix approach: [適切に対処する方法]

**[領域/コンポーネント]:**
- Issue: [ショートカット/回避策の内容]
- Why: [なぜこのようにされたか]
- Impact: [何が壊れるか、または劣化するか]
- Fix approach: [適切に対処する方法]

## 既知のバグ

**[バグの説明]:**
- Symptoms: [何が起こるか]
- Trigger: [再現方法]
- Workaround: [一時的な緩和策（あれば）]
- Root cause: [判明している場合]
- Blocked by: [何かを待っている場合]

**[バグの説明]:**
- Symptoms: [何が起こるか]
- Trigger: [再現方法]
- Workaround: [一時的な緩和策（あれば）]
- Root cause: [判明している場合]

## セキュリティ考慮事項

**[セキュリティ上の注意が必要な領域]:**
- Risk: [何が問題になりうるか]
- Current mitigation: [現在の対策]
- Recommendations: [追加すべきもの]

**[セキュリティ上の注意が必要な領域]:**
- Risk: [何が問題になりうるか]
- Current mitigation: [現在の対策]
- Recommendations: [追加すべきもの]

## パフォーマンスボトルネック

**[遅い操作/エンドポイント]:**
- Problem: [何が遅いか]
- Measurement: [実際の数値: "500ms p95", "2秒のロード時間"]
- Cause: [なぜ遅いか]
- Improvement path: [高速化の方法]

**[遅い操作/エンドポイント]:**
- Problem: [何が遅いか]
- Measurement: [実際の数値]
- Cause: [なぜ遅いか]
- Improvement path: [高速化の方法]

## 脆弱な領域

**[コンポーネント/モジュール]:**
- Why fragile: [なぜ壊れやすいか]
- Common failures: [典型的に何が問題になるか]
- Safe modification: [壊さずに変更する方法]
- Test coverage: [テストされているか？ギャップは？]

**[コンポーネント/モジュール]:**
- Why fragile: [なぜ壊れやすいか]
- Common failures: [典型的に何が問題になるか]
- Safe modification: [壊さずに変更する方法]
- Test coverage: [テストされているか？ギャップは？]

## スケーリング限界

**[リソース/システム]:**
- Current capacity: [数値: "100 req/sec", "10kユーザー"]
- Limit: [どこで破綻するか]
- Symptoms at limit: [限界時に何が起こるか]
- Scaling path: [キャパシティを増やす方法]

## リスクのある依存関係

**[パッケージ/サービス]:**
- Risk: [例: "非推奨", "メンテナンスされていない", "破壊的変更が予定"]
- Impact: [失敗した場合何が壊れるか]
- Migration plan: [代替手段またはアップグレードパス]

## 不足している重要機能

**[機能のギャップ]:**
- Problem: [何が不足しているか]
- Current workaround: [ユーザーの現在の対処法]
- Blocks: [これなしでは何ができないか]
- Implementation complexity: [おおよその工数見積もり]

## テストカバレッジのギャップ

**[テストされていない領域]:**
- What's not tested: [具体的な機能]
- Risk: [気づかれずに壊れる可能性があるもの]
- Priority: [高/中/低]
- Difficulty to test: [なぜまだテストされていないか]

---

*懸念事項監査: [日付]*
*問題が修正されたり新たに発見された際に更新*
```

<good_examples>
```markdown
# コードベース懸念事項

**分析日:** 2025-01-20

## 技術的負債

**Reactコンポーネント内のデータベースクエリ:**
- Issue: サーバーアクションではなく、15以上のページコンポーネントで直接Supabaseクエリを実行
- Files: `app/dashboard/page.tsx`, `app/profile/page.tsx`, `app/courses/[id]/page.tsx`, `app/settings/page.tsx`（`app/`内にさらに11ファイル）
- Why: MVPフェーズでの高速プロトタイピング
- Impact: RLSを適切に実装できない、DB構造がクライアントに露出
- Fix approach: すべてのクエリを `app/actions/` のサーバーアクションに移動し、適切なRLSポリシーを追加

**手動のWebhook署名検証:**
- Issue: 3つの異なるエンドポイントにコピー＆ペーストされたStripe Webhook検証コード
- Files: `app/api/webhooks/stripe/route.ts`, `app/api/webhooks/checkout/route.ts`, `app/api/webhooks/subscription/route.ts`
- Why: 各Webhookが抽象化なしにアドホックで追加された
- Impact: 新しいWebhookで検証を見落としやすい（セキュリティリスク）
- Fix approach: 共有の `lib/stripe/validate-webhook.ts` ミドルウェアを作成

## 既知のバグ

**サブスクリプション更新の競合状態:**
- Symptoms: 支払い成功後、5〜10秒間ユーザーが「無料」ティアとして表示される
- Trigger: Stripeチェックアウトリダイレクト後、Webhookが処理される前の高速ナビゲーション
- Files: `app/checkout/success/page.tsx`（リダイレクトハンドラー）、`app/api/webhooks/stripe/route.ts`（Webhook）
- Workaround: Stripe Webhookが最終的にステータスを更新（自己回復）
- Root cause: Webhook処理がユーザーナビゲーションより遅い、楽観的UI更新がない
- Fix: リダイレクト後に `app/checkout/success/page.tsx` でポーリングを追加

**ログアウト後の不整合なセッション状態:**
- Symptoms: ログアウト後、/loginではなく/dashboardにリダイレクトされる
- Trigger: モバイルナビのボタンからログアウト（デスクトップは正常）
- File: `components/MobileNav.tsx`（45行目付近、ログアウトハンドラー）
- Workaround: 手動で/loginにURL遷移すれば動作
- Root cause: モバイルナビコンポーネントがsupabase.auth.signOut()をawaitしていない
- Fix: `components/MobileNav.tsx` のログアウトハンドラーにawaitを追加

## セキュリティ考慮事項

**管理者ロールチェックがクライアントサイドのみ:**
- Risk: 管理者ダッシュボードページがSupabaseクライアントからisAdminをチェック、サーバー検証なし
- Files: `app/admin/page.tsx`, `app/admin/users/page.tsx`, `components/AdminGuard.tsx`
- Current mitigation: なし（UI非表示に依存）
- Recommendations: `middleware.ts` で管理者ルートにミドルウェアを追加、サーバーサイドでロールを検証

**未検証のファイルアップロード:**
- Risk: ユーザーがアバターバケットに任意のファイルタイプをアップロード可能（サイズ/タイプの検証なし）
- File: `components/AvatarUpload.tsx`（アップロードハンドラー）
- Current mitigation: Supabaseバケットが2MBに制限（ダッシュボードで設定）
- Recommendations: `lib/storage/validate.ts` にファイルタイプ検証（image/*のみ）を追加

## パフォーマンスボトルネック

**/api/coursesエンドポイント:**
- Problem: ネストされたレッスンと著者を含むすべてのコースを取得
- File: `app/api/courses/route.ts`
- Measurement: 50以上のコースでp95レスポンスタイム1.2秒
- Cause: N+1クエリパターン（コースごとにレッスンの個別クエリ）
- Improvement path: `lib/db/courses.ts` でPrisma includeを使用してレッスンをイーガーロード、Redisキャッシュを追加

**ダッシュボード初期ロード:**
- Problem: マウント時に5つのシリアルAPIコールがウォーターフォール
- File: `app/dashboard/page.tsx`
- Measurement: 低速3Gでインタラクティブになるまで3.5秒
- Cause: 各コンポーネントが独立してデータを取得
- Improvement path: 単一の並列フェッチを持つServer Componentに変換

## 脆弱な領域

**認証ミドルウェアチェーン:**
- File: `middleware.ts`
- Why fragile: 4つの異なるミドルウェア関数が特定の順序で実行（auth → role → subscription → logging）
- Common failures: ミドルウェアの順序変更ですべてが壊れる、デバッグが困難
- Safe modification: 順序変更前にテストを追加、コメントで依存関係を文書化
- Test coverage: ミドルウェアチェーンの統合テストなし（ユニットテストのみ）

**Stripe Webhookイベント処理:**
- File: `app/api/webhooks/stripe/route.ts`
- Why fragile: 12のイベントタイプを持つ巨大なswitch文、共有トランザクションロジック
- Common failures: 新しいイベントタイプが処理なしで追加される、エラー時の部分的なDB更新
- Safe modification: 各イベントハンドラーを `lib/stripe/handlers/*.ts` に抽出
- Test coverage: 12のイベントタイプのうち3つのみテストあり

## スケーリング限界

**Supabase無料ティア:**
- Current capacity: 500MBデータベース、1GBファイルストレージ、2GB帯域幅/月
- Limit: 限界到達まで推定約5000ユーザー
- Symptoms at limit: 429レート制限エラー、DB書き込み失敗
- Scaling path: Proにアップグレード（$25/月）で8GB DB、100GBストレージに拡張

**サーバーサイドレンダリングのブロッキング:**
- Current capacity: 速度低下までに約50同時ユーザー
- Limit: Vercel Hobbyプラン（10秒の関数タイムアウト、100GB-hrs/月）
- Symptoms at limit: コースページで504ゲートウェイタイムアウト
- Scaling path: Vercel Pro（$20/月）にアップグレード、エッジキャッシュを追加

## リスクのある依存関係

**react-hot-toast:**
- Risk: メンテナンスされていない（最終更新18ヶ月前）、React 19互換性不明
- Impact: トースト通知が壊れる、グレースフルデグラデーションなし
- Migration plan: sonnerに切り替え（アクティブにメンテナンス、類似API）

## 不足している重要機能

**支払い失敗の処理:**
- Problem: サブスクリプション支払い失敗時のリトライ機構やユーザー通知がない
- Current workaround: ユーザーが手動で支払い情報を再入力（気づいた場合）
- Blocks: 期限切れカードのユーザーを維持できない、督促プロセスなし
- Implementation complexity: 中（Stripe Webhook + メールフロー + UI）

**コース進捗トラッキング:**
- Problem: 完了したレッスンの永続的な状態がない
- Current workaround: ユーザーが手動で進捗を追跡
- Blocks: 完了率を表示できない、次のレッスンを推奨できない
- Implementation complexity: 低（completed_lessons中間テーブルを追加）

## テストカバレッジのギャップ

**支払いフローのエンドツーエンド:**
- What's not tested: 完全なStripeチェックアウト → Webhook → サブスクリプション有効化フロー
- Risk: 支払い処理がサイレントに壊れる可能性（過去2回発生）
- Priority: 高
- Difficulty to test: Stripeテストフィクスチャとwebhookシミュレーションセットアップが必要

**エラーバウンダリの動作:**
- What's not tested: コンポーネントがエラーをスローした際のアプリの動作
- Risk: ユーザーに白い画面が表示される、エラーレポートなし
- Priority: 中
- Difficulty to test: テスト環境で意図的にエラーを発生させる必要がある

---

*懸念事項監査: 2025-01-20*
*問題が修正されたり新たに発見された際に更新*
```
</good_examples>

<guidelines>
**CONCERNS.mdに含めるもの:**
- 明確な影響と修正アプローチを伴う技術的負債
- 再現手順を含む既知のバグ
- セキュリティギャップと緩和策の推奨
- 測定値を伴うパフォーマンスボトルネック
- 壊れやすいコード
- 数値を伴うスケーリング限界
- 注意が必要な依存関係
- ワークフローをブロックする不足機能
- テストカバレッジのギャップ

**ここに含めないもの:**
- 根拠のない意見（「コードが汚い」）
- 解決策のない不満（「認証がダメ」）
- 将来の機能アイデア（プロダクト計画の範囲）
- 通常のTODO（コードコメントに記載）
- 正常に機能しているアーキテクチャ上の決定
- 軽微なコードスタイルの問題

**このテンプレートを記入する際:**
- **常にファイルパスを含める** - 場所のない懸念事項は実用的ではありません。バッククォートを使用: `src/file.ts`
- 測定値は具体的に（「遅い」ではなく「500ms p95」）
- バグには再現手順を含める
- 問題だけでなく修正アプローチを提案
- 実用的な項目に焦点を当てる
- リスク/影響で優先順位付け
- 問題が解決されたら更新
- 発見されたら新しい懸念事項を追加

**トーンのガイドライン:**
- 感情的ではなくプロフェッショナルに（「ひどいクエリ」ではなく「N+1クエリパターン」）
- 解決志向で（「修正が必要」ではなく「修正: インデックスを追加」）
- リスクに焦点を当てて（「セキュリティが悪い」ではなく「ユーザーデータが露出する可能性」）
- 事実に基づいて（「本当に遅い」ではなく「3.5秒のロード時間」）

**フェーズ計画で役立つ場面:**
- 次に取り組むべきことの決定
- 変更のリスク見積もり
- 注意すべき場所の理解
- 改善の優先順位付け
- 新しいClaudeコンテキストのオンボーディング
- リファクタリング作業の計画

**このデータの収集方法:**
探索エージェントがコードベースマッピング中にこれらを検出します。人が発見した問題の手動追加も歓迎です。これは生きたドキュメントであり、不満リストではありません。
</guidelines>
