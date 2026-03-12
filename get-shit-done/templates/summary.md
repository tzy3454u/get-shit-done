# サマリーテンプレート

`.planning/phases/XX-name/{phase}-{plan}-SUMMARY.md` 用テンプレート - フェーズ完了ドキュメント。

---

## ファイルテンプレート

```markdown
---
phase: XX-name
plan: YY
subsystem: [主要カテゴリ: auth, payments, ui, api, database, infra, testing, etc.]
tags: [検索可能な技術: jwt, stripe, react, postgres, prisma]

# 依存関係グラフ
requires:
  - phase: [依存する先行フェーズ]
    provides: [そのフェーズが構築し本フェーズで使用するもの]
provides:
  - [本フェーズで構築/提供したもののリスト]
affects: [このコンテキストを必要とするフェーズ名やキーワードのリスト]

# 技術追跡
tech-stack:
  added: [このフェーズで追加したライブラリ/ツール]
  patterns: [確立したアーキテクチャ/コードパターン]

key-files:
  created: [作成した重要ファイル]
  modified: [変更した重要ファイル]

key-decisions:
  - "Decision 1"
  - "Decision 2"

patterns-established:
  - "Pattern 1: description"
  - "Pattern 2: description"

requirements-completed: []  # 必須 — このプランの `requirements` フロントマターフィールドからすべての要件IDをコピー。

# メトリクス
duration: Xmin
completed: YYYY-MM-DD
---

# Phase [X]: [名前] サマリー

**[成果を説明する実質的な一行 - "フェーズ完了" や "実装完了" ではなく]**

## パフォーマンス

- **所要時間:** [時間]（例: 23 min, 1h 15m）
- **開始:** [ISOタイムスタンプ]
- **完了:** [ISOタイムスタンプ]
- **タスク:** [完了数]
- **変更ファイル:** [数]

## 成果
- [最も重要な成果]
- [2番目の主要な達成事項]
- [該当する場合3番目]

## タスクコミット

各タスクはアトミックにコミットされました:

1. **Task 1: [タスク名]** - `abc123f` (feat/fix/test/refactor)
2. **Task 2: [タスク名]** - `def456g` (feat/fix/test/refactor)
3. **Task 3: [タスク名]** - `hij789k` (feat/fix/test/refactor)

**Plan metadata:** `lmn012o` (docs: complete plan)

_備考: TDDタスクは複数のコミットを持つ場合があります（test → feat → refactor）_

## 作成/変更ファイル
- `path/to/file.ts` - 役割
- `path/to/another.ts` - 役割

## 行われた決定
[重要な決定と簡単な根拠、または "None - followed plan as specified"]

## プランからの逸脱

[逸脱がない場合: "None - plan executed exactly as written"]

[逸脱があった場合:]

### 自動修正された問題

**1. [Rule X - カテゴリ] 簡単な説明**
- **発見時点:** Task [N] ([タスク名])
- **問題:** [何が問題だったか]
- **修正:** [何を行ったか]
- **変更ファイル:** [ファイルパス]
- **検証:** [どのように検証したか]
- **コミット先:** [ハッシュ]（タスクコミットの一部）

[... 各自動修正について繰り返し ...]

---

**逸脱合計:** [N] 件自動修正（[ルール別内訳]）
**プランへの影響:** [簡単な評価 - 例: "すべての自動修正は正確性/セキュリティのために必要。スコープクリープなし。"]

## 発生した問題
[問題とその解決方法、または "None"]

[備考: "プランからの逸脱"は逸脱ルールにより自動的に処理された計画外の作業を記録します。"発生した問題"は計画された作業中に発生し問題解決が必要だった問題を記録します。]

## ユーザーセットアップが必要

[USER-SETUP.mdが生成された場合:]
**外部サービスの手動設定が必要です。** [{phase}-USER-SETUP.md](./{phase}-USER-SETUP.md) を参照:
- 追加する環境変数
- ダッシュボード設定手順
- 検証コマンド

[USER-SETUP.mdがない場合:]
なし - 外部サービスの設定は不要です。

## 次フェーズの準備状況
[次のフェーズに向けて準備できていること]
[ブロッカーや懸念事項]

---
*Phase: XX-name*
*Completed: [date]*
```

<frontmatter_guidance>
**目的:** 依存関係グラフによる自動コンテキスト組み立てを可能にする。フロントマターはサマリーのメタデータを機械可読にし、plan-phaseが依存関係に基づいてすべてのサマリーを素早くスキャンし関連するものを選択できるようにします。

**高速スキャン:** フロントマターは最初の約25行で、全サマリーのフルコンテンツを読まずに安価にスキャンできます。

**依存関係グラフ:** `requires`/`provides`/`affects`がフェーズ間の明示的なリンクを作成し、コンテキスト選択のための推移的閉包を可能にします。

**Subsystem:** 関連フェーズを検出するための主要カテゴリ分類（auth, payments, ui, api, database, infra, testing）。

**Tags:** 技術スタック認識のための検索可能な技術キーワード（ライブラリ、フレームワーク、ツール）。

**Key-files:** PLAN.mdでの@contextリファレンス用の重要ファイル。

**Patterns:** 将来のフェーズが維持すべき確立された規約。

**Population:** フロントマターはexecute-plan.mdでのサマリー作成時に入力されます。フィールドごとのガイダンスは `<step name="create_summary">` を参照してください。
</frontmatter_guidance>

<one_liner_rules>
一行説明は実質的でなければなりません:

**良い例:**
- "JWT auth with refresh rotation using jose library"
- "Prisma schema with User, Session, and Product models"
- "Dashboard with real-time metrics via Server-Sent Events"

**悪い例:**
- "Phase complete"
- "Authentication implemented"
- "Foundation finished"
- "All tasks done"

一行説明は実際に何がリリースされたかを伝えるべきです。
</one_liner_rules>

<example>
```markdown
# Phase 1: Foundation サマリー

**joseライブラリを使用したリフレッシュローテーション付きJWT認証、Prisma Userモデル、保護APIミドルウェア**

## パフォーマンス

- **所要時間:** 28 min
- **開始:** 2025-01-15T14:22:10Z
- **完了:** 2025-01-15T14:50:33Z
- **タスク:** 5
- **変更ファイル:** 8

## 成果
- メール/パスワード認証のUserモデル
- httpOnly JWTクッキーを使用したログイン/ログアウトエンドポイント
- トークンの有効性をチェックする保護ルートミドルウェア
- リクエストごとのリフレッシュトークンローテーション

## 作成/変更ファイル
- `prisma/schema.prisma` - UserとSessionモデル
- `src/app/api/auth/login/route.ts` - ログインエンドポイント
- `src/app/api/auth/logout/route.ts` - ログアウトエンドポイント
- `src/middleware.ts` - 保護ルートチェック
- `src/lib/auth.ts` - joseを使用したJWTヘルパー

## 行われた決定
- jsonwebtokenの代わりにjoseを使用（ESMネイティブ、Edge互換）
- 15分アクセストークンと7日間リフレッシュトークン
- 失効機能のためリフレッシュトークンをデータベースに保存

## プランからの逸脱

### 自動修正された問題

**1. [Rule 2 - 重大な欠落] bcryptによるパスワードハッシュ化を追加**
- **発見時点:** Task 2（ログインエンドポイント実装）
- **問題:** プランにパスワードハッシュ化が明記されていなかった - 平文保存は重大なセキュリティ欠陥
- **修正:** 登録時にbcryptハッシュ化、ログイン時にソルトラウンド10で比較を追加
- **変更ファイル:** src/app/api/auth/login/route.ts, src/lib/auth.ts
- **検証:** パスワードハッシュテスト合格、平文は一切保存されない
- **コミット先:** abc123f（Task 2コミット）

**2. [Rule 3 - ブロッキング] 不足していたjose依存関係をインストール**
- **発見時点:** Task 4（JWTトークン生成）
- **問題:** joseパッケージがpackage.jsonになく、インポート失敗
- **修正:** `npm install jose` を実行
- **変更ファイル:** package.json, package-lock.json
- **検証:** インポート成功、ビルド通過
- **コミット先:** def456g（Task 4コミット）

---

**逸脱合計:** 2件自動修正（1件重大な欠落、1件ブロッキング）
**プランへの影響:** 両方の自動修正はセキュリティと機能性のために不可欠。スコープクリープなし。

## 発生した問題
- jsonwebtokenのCommonJSインポートがEdgeランタイムで失敗 - joseに切り替え（計画通りのライブラリ変更、期待通りに動作）

## 次フェーズの準備状況
- 認証基盤完了、機能開発の準備完了
- パブリックローンチ前にユーザー登録エンドポイントが必要

---
*Phase: 01-foundation*
*Completed: 2025-01-15*
```
</example>

<guidelines>
**フロントマター:** 必須 - すべてのフィールドを完成させること。将来の計画のための自動コンテキスト組み立てを可能にします。

**一行説明:** 実質的であること。"Authentication implemented" ではなく "JWT auth with refresh rotation using jose library"。

**決定セクション:**
- 実行中に行われた主要な決定と根拠
- STATE.mdの蓄積コンテキストに抽出
- 逸脱がない場合は "None - followed plan as specified" を使用

**作成後:** STATE.mdが位置、決定事項、問題で更新されます。
</guidelines>
