# 検証レポートテンプレート

`.planning/phases/XX-name/{phase_num}-VERIFICATION.md` 用テンプレート — フェーズ目標の検証結果。

---

## ファイルテンプレート

```markdown
---
phase: XX-name
verified: YYYY-MM-DDTHH:MM:SSZ
status: passed | gaps_found | human_needed
score: N/M must-haves verified
---

# Phase {X}: {名前} 検証レポート

**Phase Goal:** {ROADMAP.mdからの目標}
**Verified:** {タイムスタンプ}
**Status:** {passed | gaps_found | human_needed}

## 目標達成

### 観察可能な真実

| # | 真実 | ステータス | エビデンス |
|---|-------|--------|----------|
| 1 | {must_havesからの真実} | ✓ VERIFIED | {確認した内容} |
| 2 | {must_havesからの真実} | ✗ FAILED | {何が問題か} |
| 3 | {must_havesからの真実} | ? UNCERTAIN | {検証できない理由} |

**スコア:** {N}/{M} 件の真実を検証

### 必須アーティファクト

| アーティファクト | 期待値 | ステータス | 詳細 |
|----------|----------|--------|---------|
| `src/components/Chat.tsx` | メッセージリストコンポーネント | ✓ EXISTS + SUBSTANTIVE | ChatListをエクスポート、Message[]をレンダリング、スタブなし |
| `src/app/api/chat/route.ts` | メッセージCRUD | ✗ STUB | ファイルは存在するがPOSTはプレースホルダーを返す |
| `prisma/schema.prisma` | メッセージモデル | ✓ EXISTS + SUBSTANTIVE | モデルがすべてのフィールドで定義済み |

**アーティファクト:** {N}/{M} 件検証済み

### 主要リンク検証

| From | To | Via | ステータス | 詳細 |
|------|----|----|--------|---------|
| Chat.tsx | /api/chat | fetch in useEffect | ✓ WIRED | Line 23: `fetch('/api/chat')` でレスポンス処理 |
| ChatInput | /api/chat POST | onSubmit handler | ✗ NOT WIRED | onSubmitはconsole.logのみ呼び出し |
| /api/chat POST | database | prisma.message.create | ✗ NOT WIRED | ハードコードされたレスポンスを返す、DB呼び出しなし |

**接続:** {N}/{M} 件の接続を検証

## 要件カバレッジ

| 要件 | ステータス | ブロッキング問題 |
|-------------|--------|----------------|
| {REQ-01}: {説明} | ✓ SATISFIED | - |
| {REQ-02}: {説明} | ✗ BLOCKED | APIルートがスタブ |
| {REQ-03}: {説明} | ? NEEDS HUMAN | WebSocketをプログラム的に検証不可 |

**カバレッジ:** {N}/{M} 件の要件を充足

## 検出されたアンチパターン

| ファイル | 行 | パターン | 重大度 | 影響 |
|------|------|---------|----------|--------|
| src/app/api/chat/route.ts | 12 | `// TODO: implement` | ⚠️ Warning | 未完了を示す |
| src/components/Chat.tsx | 45 | `return <div>Placeholder</div>` | 🛑 Blocker | コンテンツがレンダリングされない |
| src/hooks/useChat.ts | - | ファイル不在 | 🛑 Blocker | 期待されるhookが存在しない |

**アンチパターン:** {N} 件検出（{blockers} 件ブロッカー、{warnings} 件警告）

## 人手による検証が必要

{人手による検証が不要な場合:}
なし — すべての検証可能項目をプログラム的にチェック済み。

{人手による検証が必要な場合:}

### 1. {テスト名}
**テスト:** {実行すること}
**期待値:** {起こるべきこと}
**人手が必要な理由:** {プログラム的に検証できない理由}

### 2. {テスト名}
**テスト:** {実行すること}
**期待値:** {起こるべきこと}
**人手が必要な理由:** {プログラム的に検証できない理由}

## ギャップサマリー

{ギャップがない場合:}
**ギャップは見つかりませんでした。** フェーズ目標達成。続行の準備完了。

{ギャップが見つかった場合:}

### クリティカルギャップ（進行をブロック）

1. **{ギャップ名}**
   - 不足: {不足しているもの}
   - 影響: {目標をブロックする理由}
   - 修正: {必要なアクション}

2. **{ギャップ名}**
   - 不足: {不足しているもの}
   - 影響: {目標をブロックする理由}
   - 修正: {必要なアクション}

### 非クリティカルギャップ（延期可能）

1. **{ギャップ名}**
   - 問題: {何が問題か}
   - 影響: {影響が限定的な理由...}
   - 推奨: {今すぐ修正するか延期するか}

## 推奨修正プラン

{ギャップが見つかった場合、修正プランの推奨を生成:}

### {phase}-{next}-PLAN.md: {修正名}

**目的:** {修正内容}

**タスク:**
1. {ギャップ1を修正するタスク}
2. {ギャップ2を修正するタスク}
3. {検証タスク}

**推定規模:** {Small / Medium}

---

### {phase}-{next+1}-PLAN.md: {修正名}

**目的:** {修正内容}

**タスク:**
1. {タスク}
2. {タスク}

**推定規模:** {Small / Medium}

---

## 検証メタデータ

**検証アプローチ:** 目標逆算（フェーズ目標から導出）
**Must-havesソース:** {PLAN.md frontmatter | ROADMAP.md goalから導出}
**自動チェック:** {N} 件通過、{M} 件失敗
**人手チェック必要:** {N} 件
**総検証時間:** {duration}

---
*Verified: {タイムスタンプ}*
*Verifier: Claude (subagent)*
```

---

## ガイドライン

**ステータス値:**
- `passed` — すべてのmust-havesが検証済み、ブロッカーなし
- `gaps_found` — 1つ以上のクリティカルギャップが見つかった
- `human_needed` — 自動チェックは通過だが人手による検証が必要

**エビデンスタイプ:**
- EXISTS: "パスにファイルあり、Xをエクスポート"
- SUBSTANTIVE: "N行、パターンX, Y, Zを持つ"
- WIRED: "行N: AとBを接続するコード"
- FAILED: "Xが理由で不足" または "Yが理由でスタブ"

**重大度レベル:**
- 🛑 Blocker: 目標達成を妨げる、修正必須
- ⚠️ Warning: 未完了を示すが目標はブロックしない
- ℹ️ Info: 注目すべきだが問題はない

**修正プラン生成:**
- gaps_foundの場合のみ生成
- 関連する修正を1つのプランにグループ化
- プランごとに2〜3タスクに抑える
- 各プランに検証タスクを含める

---

## 例

```markdown
---
phase: 03-chat
verified: 2025-01-15T14:30:00Z
status: gaps_found
score: 2/5 must-haves verified
---

# Phase 3: チャットインターフェース 検証レポート

**Phase Goal:** ユーザーがメッセージを送受信できる動作するチャットインターフェース
**Verified:** 2025-01-15T14:30:00Z
**Status:** gaps_found

## 目標達成

### 観察可能な真実

| # | 真実 | ステータス | エビデンス |
|---|-------|--------|----------|
| 1 | ユーザーが既存のメッセージを見られる | ✗ FAILED | コンポーネントがメッセージデータではなくプレースホルダーをレンダリング |
| 2 | ユーザーがメッセージを入力できる | ✓ VERIFIED | onChangeハンドラー付きの入力フィールドが存在 |
| 3 | ユーザーがメッセージを送信できる | ✗ FAILED | onSubmitハンドラーがconsole.logのみ |
| 4 | 送信メッセージがリストに表示される | ✗ FAILED | 送信後のステート更新なし |
| 5 | メッセージがリフレッシュ後も持続する | ? UNCERTAIN | 検証不可 - 送信が機能しない |

**スコア:** 1/5 件の真実を検証

### 必須アーティファクト

| アーティファクト | 期待値 | ステータス | 詳細 |
|----------|----------|--------|---------|
| `src/components/Chat.tsx` | メッセージリストコンポーネント | ✗ STUB | `<div>Chat will be here</div>` を返す |
| `src/components/ChatInput.tsx` | メッセージ入力 | ✓ EXISTS + SUBSTANTIVE | 入力フィールド、送信ボタン、ハンドラー付きフォーム |
| `src/app/api/chat/route.ts` | メッセージCRUD | ✗ STUB | GETは[]を返す、POSTは{ ok: true }を返す |
| `prisma/schema.prisma` | メッセージモデル | ✓ EXISTS + SUBSTANTIVE | id, content, userId, createdAt付きMessageモデル |

**アーティファクト:** 2/4 件検証済み

### 主要リンク検証

| From | To | Via | ステータス | 詳細 |
|------|----|----|--------|---------|
| Chat.tsx | /api/chat GET | fetch | ✗ NOT WIRED | コンポーネントにfetch呼び出しなし |
| ChatInput | /api/chat POST | onSubmit | ✗ NOT WIRED | ハンドラーはログのみ、fetchなし |
| /api/chat GET | database | prisma.message.findMany | ✗ NOT WIRED | ハードコードされた[]を返す |
| /api/chat POST | database | prisma.message.create | ✗ NOT WIRED | { ok: true }を返す、DB呼び出しなし |

**接続:** 0/4 件の接続を検証

## 要件カバレッジ

| 要件 | ステータス | ブロッキング問題 |
|-------------|--------|----------------|
| CHAT-01: ユーザーがメッセージを送信できる | ✗ BLOCKED | API POSTがスタブ |
| CHAT-02: ユーザーがメッセージを閲覧できる | ✗ BLOCKED | コンポーネントがプレースホルダー |
| CHAT-03: メッセージが永続化される | ✗ BLOCKED | データベース統合なし |

**カバレッジ:** 0/3 件の要件を充足

## 検出されたアンチパターン

| ファイル | 行 | パターン | 重大度 | 影響 |
|------|------|---------|----------|--------|
| src/components/Chat.tsx | 8 | `<div>Chat will be here</div>` | 🛑 Blocker | 実際のコンテンツなし |
| src/app/api/chat/route.ts | 5 | `return Response.json([])` | 🛑 Blocker | ハードコードされた空 |
| src/app/api/chat/route.ts | 12 | `// TODO: save to database` | ⚠️ Warning | 未完了 |

**アンチパターン:** 3件検出（2件ブロッカー、1件警告）

## 人手による検証が必要

自動化されたギャップが修正されるまで不要。

## ギャップサマリー

### クリティカルギャップ（進行をブロック）

1. **チャットコンポーネントがプレースホルダー**
   - 不足: 実際のメッセージリストレンダリング
   - 影響: ユーザーにメッセージの代わりに "Chat will be here" が表示される
   - 修正: Chat.tsxを実装しメッセージをフェッチしてレンダリング

2. **APIルートがスタブ**
   - 不足: GETとPOSTのデータベース統合
   - 影響: データ永続化なし、実際の機能なし
   - 修正: ルートハンドラーにprisma呼び出しを接続

3. **フロントエンドとバックエンド間の接続なし**
   - 不足: コンポーネントでのfetch呼び出し
   - 影響: APIが動作してもUIがそれを呼び出さない
   - 修正: ChatにuseEffect fetchを、ChatInputにonSubmit fetchを追加

## 推奨修正プラン

### 03-04-PLAN.md: チャットAPIの実装

**目的:** APIルートをデータベースに接続

**タスク:**
1. GET /api/chatをprisma.message.findManyで実装
2. POST /api/chatをprisma.message.createで実装
3. 検証: APIが実データを返す、POSTがレコードを作成

**推定規模:** Small

---

### 03-05-PLAN.md: チャットUIの実装

**目的:** ChatコンポーネントをAPIに接続

**タスク:**
1. Chat.tsxをuseEffect fetchとメッセージレンダリングで実装
2. ChatInputのonSubmitをPOST /api/chatに接続
3. 検証: メッセージが表示される、送信後に新メッセージが表示される

**推定規模:** Small

---

## 検証メタデータ

**検証アプローチ:** 目標逆算（フェーズ目標から導出）
**Must-havesソース:** 03-01-PLAN.md frontmatter
**自動チェック:** 2件通過、8件失敗
**人手チェック必要:** 0件（自動化の失敗によりブロック）
**総検証時間:** 2 min

---
*Verified: 2025-01-15T14:30:00Z*
*Verifier: Claude (subagent)*
```
