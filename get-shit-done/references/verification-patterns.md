# 検証パターン

さまざまな種類の成果物が、スタブやプレースホルダーではなく実際の実装であることを検証する方法。

<core_principle>
**存在 ≠ 実装**

ファイルが存在することは機能が動作することを意味しません。検証は以下を確認する必要があります:
1. **存在** - ファイルが期待されるパスに存在する
2. **実質的** - 内容がプレースホルダーではなく実際の実装
3. **接続済み** - システムの残りの部分に接続されている
4. **機能的** - 呼び出されたときに実際に動作する

レベル1-3はプログラム的に確認できます。レベル4は多くの場合、人間による検証が必要です。
</core_principle>

<stub_detection>

## ユニバーサルスタブパターン

これらのパターンは、ファイルタイプに関係なくプレースホルダーコードを示します:

**コメントベースのスタブ:**
```bash
# スタブコメントのgrepパターン
grep -E "(TODO|FIXME|XXX|HACK|PLACEHOLDER)" "$file"
grep -E "implement|add later|coming soon|will be" "$file" -i
grep -E "// \.\.\.|/\* \.\.\. \*/|# \.\.\." "$file"
```

**出力のプレースホルダーテキスト:**
```bash
# UIプレースホルダーパターン
grep -E "placeholder|lorem ipsum|coming soon|under construction" "$file" -i
grep -E "sample|example|test data|dummy" "$file" -i
grep -E "\[.*\]|<.*>|\{.*\}" "$file"  # 残されたテンプレートブラケット
```

**空または些細な実装:**
```bash
# 何もしない関数
grep -E "return null|return undefined|return \{\}|return \[\]" "$file"
grep -E "pass$|\.\.\.|\bnothing\b" "$file"
grep -E "console\.(log|warn|error).*only" "$file"  # ログのみの関数
```

**動的であるべきところにハードコードされた値:**
```bash
# ハードコードされたID、カウント、コンテンツ
grep -E "id.*=.*['\"].*['\"]" "$file"  # ハードコードされた文字列ID
grep -E "count.*=.*\d+|length.*=.*\d+" "$file"  # ハードコードされたカウント
grep -E "\\\$\d+\.\d{2}|\d+ items" "$file"  # ハードコードされた表示値
```

</stub_detection>

<react_components>

## React/Next.jsコンポーネント

**存在チェック:**
```bash
# ファイルが存在しコンポーネントをエクスポート
[ -f "$component_path" ] && grep -E "export (default |)function|export const.*=.*\(" "$component_path"
```

**実質性チェック:**
```bash
# プレースホルダーではなく実際のJSXを返す
grep -E "return.*<" "$component_path" | grep -v "return.*null" | grep -v "placeholder" -i

# 意味のあるコンテンツがある（ラッパーdivだけではない）
grep -E "<[A-Z][a-zA-Z]+|className=|onClick=|onChange=" "$component_path"

# propsまたはstateを使用（静的ではない）
grep -E "props\.|useState|useEffect|useContext|\{.*\}" "$component_path"
```

**Reactに特有のスタブパターン:**
```javascript
// 警告 - これらはスタブです:
return <div>Component</div>
return <div>Placeholder</div>
return <div>{/* TODO */}</div>
return <p>Coming soon</p>
return null
return <></>

// これもスタブ - 空のハンドラー:
onClick={() => {}}
onChange={() => console.log('clicked')}
onSubmit={(e) => e.preventDefault()}  // デフォルトを防ぐだけで何もしない
```

**接続チェック:**
```bash
# コンポーネントが必要なものをインポート
grep -E "^import.*from" "$component_path"

# propsが実際に使用されている（受け取るだけでなく）
# デストラクチャリングまたはprops.Xの使用を確認
grep -E "\{ .* \}.*props|\bprops\.[a-zA-Z]+" "$component_path"

# API呼び出しが存在する（データフェッチコンポーネント用）
grep -E "fetch\(|axios\.|useSWR|useQuery|getServerSideProps|getStaticProps" "$component_path"
```

**機能検証（人間が必要）:**
- コンポーネントは可視コンテンツをレンダリングするか？
- インタラクティブ要素はクリックに応答するか？
- データは読み込まれて表示されるか？
- エラー状態は適切に表示されるか？

</react_components>

<api_routes>

## APIルート（Next.js App Router / Express / etc.）

**存在チェック:**
```bash
# ルートファイルが存在
[ -f "$route_path" ]

# HTTPメソッドハンドラーをエクスポート（Next.js App Router）
grep -E "export (async )?(function|const) (GET|POST|PUT|PATCH|DELETE)" "$route_path"

# またはExpress形式のハンドラー
grep -E "\.(get|post|put|patch|delete)\(" "$route_path"
```

**実質性チェック:**
```bash
# 単なるreturn文ではなく実際のロジックがある
wc -l "$route_path"  # 10-15行以上なら実際の実装を示唆

# データソースとやり取り
grep -E "prisma\.|db\.|mongoose\.|sql|query|find|create|update|delete" "$route_path" -i

# エラーハンドリングがある
grep -E "try|catch|throw|error|Error" "$route_path"

# 意味のあるレスポンスを返す
grep -E "Response\.json|res\.json|res\.send|return.*\{" "$route_path" | grep -v "message.*not implemented" -i
```

**APIルートに特有のスタブパターン:**
```typescript
// 警告 - これらはスタブです:
export async function POST() {
  return Response.json({ message: "Not implemented" })
}

export async function GET() {
  return Response.json([])  // DBクエリなしの空配列
}

export async function PUT() {
  return new Response()  // 空のレスポンス
}

// コンソールログのみ:
export async function POST(req) {
  console.log(await req.json())
  return Response.json({ ok: true })
}
```

**接続チェック:**
```bash
# データベース/サービスクライアントをインポート
grep -E "^import.*prisma|^import.*db|^import.*client" "$route_path"

# リクエストボディを実際に使用（POST/PUT用）
grep -E "req\.json\(\)|req\.body|request\.json\(\)" "$route_path"

# 入力を検証（リクエストを信頼するだけでなく）
grep -E "schema\.parse|validate|zod|yup|joi" "$route_path"
```

**機能検証（人間または自動化）:**
- GETはデータベースから実際のデータを返すか？
- POSTは実際にレコードを作成するか？
- エラーレスポンスは正しいステータスコードか？
- 認証チェックは実際に実施されているか？

</api_routes>

<database_schema>

## データベーススキーマ（Prisma / Drizzle / SQL）

**存在チェック:**
```bash
# スキーマファイルが存在
[ -f "prisma/schema.prisma" ] || [ -f "drizzle/schema.ts" ] || [ -f "src/db/schema.sql" ]

# モデル/テーブルが定義されている
grep -E "^model $model_name|CREATE TABLE $table_name|export const $table_name" "$schema_path"
```

**実質性チェック:**
```bash
# 期待されるフィールドがある（idだけではない）
grep -A 20 "model $model_name" "$schema_path" | grep -E "^\s+\w+\s+\w+"

# 期待される場合リレーションがある
grep -E "@relation|REFERENCES|FOREIGN KEY" "$schema_path"

# 適切なフィールドタイプがある（すべてStringではない）
grep -A 20 "model $model_name" "$schema_path" | grep -E "Int|DateTime|Boolean|Float|Decimal|Json"
```

**スキーマに特有のスタブパターン:**
```prisma
// 警告 - これらはスタブです:
model User {
  id String @id
  // TODO: add fields
}

model Message {
  id        String @id
  content   String  // 実際のフィールドが1つだけ
}

// 重要なフィールドが欠落:
model Order {
  id     String @id
  // ない: userId, items, total, status, createdAt
}
```

**接続チェック:**
```bash
# マイグレーションが存在し適用されている
ls prisma/migrations/ 2>/dev/null | wc -l  # 0より大きいべき
npx prisma migrate status 2>/dev/null | grep -v "pending"

# クライアントが生成されている
[ -d "node_modules/.prisma/client" ]
```

**機能検証:**
```bash
# テーブルをクエリできる（自動化）
npx prisma db execute --stdin <<< "SELECT COUNT(*) FROM $table_name"
```

</database_schema>

<hooks_utilities>

## カスタムフックとユーティリティ

**存在チェック:**
```bash
# ファイルが存在し関数をエクスポート
[ -f "$hook_path" ] && grep -E "export (default )?(function|const)" "$hook_path"
```

**実質性チェック:**
```bash
# フックがReactフックを使用（カスタムフック用）
grep -E "useState|useEffect|useCallback|useMemo|useRef|useContext" "$hook_path"

# 意味のある戻り値がある
grep -E "return \{|return \[" "$hook_path"

# 些細な長さ以上
[ $(wc -l < "$hook_path") -gt 10 ]
```

**フックに特有のスタブパターン:**
```typescript
// 警告 - これらはスタブです:
export function useAuth() {
  return { user: null, login: () => {}, logout: () => {} }
}

export function useCart() {
  const [items, setItems] = useState([])
  return { items, addItem: () => console.log('add'), removeItem: () => {} }
}

// ハードコードされた戻り値:
export function useUser() {
  return { name: "Test User", email: "test@example.com" }
}
```

**接続チェック:**
```bash
# フックが実際にどこかでインポートされている
grep -r "import.*$hook_name" src/ --include="*.tsx" --include="*.ts" | grep -v "$hook_path"

# フックが実際に呼び出されている
grep -r "$hook_name()" src/ --include="*.tsx" --include="*.ts" | grep -v "$hook_path"
```

</hooks_utilities>

<environment_config>

## 環境変数と設定

**存在チェック:**
```bash
# .envファイルが存在
[ -f ".env" ] || [ -f ".env.local" ]

# 必要な変数が定義されている
grep -E "^$VAR_NAME=" .env .env.local 2>/dev/null
```

**実質性チェック:**
```bash
# 変数にプレースホルダーではなく実際の値がある
grep -E "^$VAR_NAME=.+" .env .env.local 2>/dev/null | grep -v "your-.*-here|xxx|placeholder|TODO" -i

# 値がタイプに対して妥当に見える:
# - URLはhttpで始まるべき
# - キーは十分な長さであるべき
# - ブーリアンはtrue/falseであるべき
```

**envに特有のスタブパターン:**
```bash
# 警告 - これらはスタブです:
DATABASE_URL=your-database-url-here
STRIPE_SECRET_KEY=sk_test_xxx
API_KEY=placeholder
NEXT_PUBLIC_API_URL=http://localhost:3000  # 本番でもまだlocalhostを指す
```

**接続チェック:**
```bash
# 変数がコード内で実際に使用されている
grep -r "process\.env\.$VAR_NAME|env\.$VAR_NAME" src/ --include="*.ts" --include="*.tsx"

# 変数がバリデーションスキーマにある（envにzod等を使用している場合）
grep -E "$VAR_NAME" src/env.ts src/env.mjs 2>/dev/null
```

</environment_config>

<wiring_verification>

## 接続検証パターン

接続検証はコンポーネントが実際に通信していることを確認します。ここがほとんどのスタブが隠れる場所です。

### パターン: コンポーネント → API

**確認:** コンポーネントは実際にAPIを呼び出しているか？

```bash
# fetch/axios呼び出しを見つける
grep -E "fetch\(['\"].*$api_path|axios\.(get|post).*$api_path" "$component_path"

# コメントアウトされていないことを確認
grep -E "fetch\(|axios\." "$component_path" | grep -v "^.*//.*fetch"

# レスポンスが使用されていることを確認
grep -E "await.*fetch|\.then\(|setData|setState" "$component_path"
```

**警告:**
```typescript
// fetchは存在するがレスポンスが無視されている:
fetch('/api/messages')  // awaitなし、.thenなし、代入なし

// コメント内のfetch:
// fetch('/api/messages').then(r => r.json()).then(setMessages)

// 間違ったエンドポイントへのfetch:
fetch('/api/message')  // タイポ - /api/messagesであるべき
```

### パターン: API → データベース

**確認:** APIルートは実際にデータベースをクエリしているか？

```bash
# データベース呼び出しを見つける
grep -E "prisma\.$model|db\.query|Model\.find" "$route_path"

# awaitされていることを確認
grep -E "await.*prisma|await.*db\." "$route_path"

# 結果が返されていることを確認
grep -E "return.*json.*data|res\.json.*result" "$route_path"
```

**警告:**
```typescript
// クエリは存在するが結果が返されていない:
await prisma.message.findMany()
return Response.json({ ok: true })  // クエリ結果ではなく静的を返す

// クエリがawaitされていない:
const messages = prisma.message.findMany()  // awaitが欠落
return Response.json(messages)  // データではなくPromiseを返す
```

### パターン: フォーム → ハンドラー

**確認:** フォーム送信は実際に何かをするか？

```bash
# onSubmitハンドラーを見つける
grep -E "onSubmit=\{|handleSubmit" "$component_path"

# ハンドラーに内容があることを確認
grep -A 10 "onSubmit.*=" "$component_path" | grep -E "fetch|axios|mutate|dispatch"

# preventDefaultだけでないことを確認
grep -A 5 "onSubmit" "$component_path" | grep -v "only.*preventDefault" -i
```

**警告:**
```typescript
// ハンドラーがデフォルトを防ぐだけ:
onSubmit={(e) => e.preventDefault()}

// ハンドラーがログを取るだけ:
const handleSubmit = (data) => {
  console.log(data)
}

// ハンドラーが空:
onSubmit={() => {}}
```

### パターン: ステート → レンダー

**確認:** コンポーネントはハードコードされたコンテンツではなくステートをレンダリングしているか？

```bash
# JSX内のステート使用を見つける
grep -E "\{.*messages.*\}|\{.*data.*\}|\{.*items.*\}" "$component_path"

# ステートのmap/レンダーを確認
grep -E "\.map\(|\.filter\(|\.reduce\(" "$component_path"

# 動的コンテンツを確認
grep -E "\{[a-zA-Z_]+\." "$component_path"  # 変数補間
```

**警告:**
```tsx
// ステートではなくハードコード:
return <div>
  <p>Message 1</p>
  <p>Message 2</p>
</div>

// ステートが存在するがレンダリングされていない:
const [messages, setMessages] = useState([])
return <div>No messages</div>  // 常に"no messages"を表示

// 間違ったステートがレンダリングされている:
const [messages, setMessages] = useState([])
return <div>{otherData.map(...)}</div>  // 別のデータを使用
```

</wiring_verification>

<verification_checklist>

## クイック検証チェックリスト

各成果物タイプに対して、このチェックリストを実行してください:

### コンポーネントチェックリスト
- [ ] ファイルが期待されるパスに存在
- [ ] function/constコンポーネントをエクスポート
- [ ] JSXを返す（nullや空ではない）
- [ ] レンダーにプレースホルダーテキストがない
- [ ] propsまたはstateを使用（静的ではない）
- [ ] イベントハンドラーに実際の実装がある
- [ ] インポートが正しく解決される
- [ ] アプリのどこかで使用されている

### APIルートチェックリスト
- [ ] ファイルが期待されるパスに存在
- [ ] HTTPメソッドハンドラーをエクスポート
- [ ] ハンドラーが5行以上ある
- [ ] データベースまたはサービスをクエリ
- [ ] 意味のあるレスポンスを返す（空/プレースホルダーではない）
- [ ] エラーハンドリングがある
- [ ] 入力を検証
- [ ] フロントエンドから呼び出されている

### スキーマチェックリスト
- [ ] モデル/テーブルが定義されている
- [ ] 期待されるすべてのフィールドがある
- [ ] フィールドに適切なタイプがある
- [ ] 必要な場合リレーションが定義されている
- [ ] マイグレーションが存在し適用されている
- [ ] クライアントが生成されている

### フック/ユーティリティチェックリスト
- [ ] ファイルが期待されるパスに存在
- [ ] 関数をエクスポート
- [ ] 意味のある実装がある（空の戻り値ではない）
- [ ] アプリのどこかで使用されている
- [ ] 戻り値が消費されている

### 接続チェックリスト
- [ ] コンポーネント → API: fetch/axios呼び出しが存在しレスポンスを使用
- [ ] API → データベース: クエリが存在し結果が返されている
- [ ] フォーム → ハンドラー: onSubmitがAPI/mutationを呼び出す
- [ ] ステート → レンダー: ステート変数がJSXに現れる

</verification_checklist>

<automated_verification_script>

## 自動検証アプローチ

検証サブエージェント用に、このパターンを使用してください:

```bash
# 1. 存在チェック
check_exists() {
  [ -f "$1" ] && echo "EXISTS: $1" || echo "MISSING: $1"
}

# 2. スタブパターンのチェック
check_stubs() {
  local file="$1"
  local stubs=$(grep -c -E "TODO|FIXME|placeholder|not implemented" "$file" 2>/dev/null || echo 0)
  [ "$stubs" -gt 0 ] && echo "STUB_PATTERNS: $stubs in $file"
}

# 3. 接続チェック（コンポーネントがAPIを呼び出す）
check_wiring() {
  local component="$1"
  local api_path="$2"
  grep -q "$api_path" "$component" && echo "WIRED: $component → $api_path" || echo "NOT_WIRED: $component → $api_path"
}

# 4. 実質性チェック（N行以上、期待されるパターンがある）
check_substantive() {
  local file="$1"
  local min_lines="$2"
  local pattern="$3"
  local lines=$(wc -l < "$file" 2>/dev/null || echo 0)
  local has_pattern=$(grep -c -E "$pattern" "$file" 2>/dev/null || echo 0)
  [ "$lines" -ge "$min_lines" ] && [ "$has_pattern" -gt 0 ] && echo "SUBSTANTIVE: $file" || echo "THIN: $file ($lines lines, $has_pattern matches)"
}
```

これらのチェックを必須の各成果物に対して実行してください。結果をVERIFICATION.mdに集約します。

</automated_verification_script>

<human_verification_triggers>

## 人間による検証が必要な場合

プログラム的に検証できないものがあります。これらを人間によるテスト用にフラグ付けしてください:

**常に人間が必要:**
- 外観（正しく見えるか？）
- ユーザーフローの完了（実際にそのことができるか？）
- リアルタイム動作（WebSocket、SSE）
- 外部サービス統合（Stripe、メール送信）
- エラーメッセージの明確さ（メッセージは役立つか？）
- パフォーマンスの体感（速く感じるか？）

**不確かな場合は人間:**
- grepで追跡できない複雑な接続
- 状態に依存する動的動作
- エッジケースとエラー状態
- モバイルレスポンシブ対応
- アクセシビリティ

**人間による検証リクエストのフォーマット:**
```markdown
## Human Verification Required

### 1. Chat message sending
**Test:** Type a message and click Send
**Expected:** Message appears in list, input clears
**Check:** Does message persist after refresh?

### 2. Error handling
**Test:** Disconnect network, try to send
**Expected:** Error message appears, message not lost
**Check:** Can retry after reconnect?
```

</human_verification_triggers>

<checkpoint_automation_reference>

## チェックポイント前の自動化

自動化ファーストのチェックポイントパターン、サーバーライフサイクル管理、CLIインストール処理、エラー回復プロトコルについては、以下を参照:

**@~/.claude/get-shit-done/references/checkpoints.md** → `<automation_reference>`セクション

主要な原則:
- Claudeはチェックポイントを提示する前に検証環境をセットアップする
- ユーザーはCLIコマンドを実行しない（URLにアクセスするのみ）
- サーバーライフサイクル: チェックポイント前に起動、ポート競合を処理、期間中は実行し続ける
- CLIインストール: 安全な場合は自動インストール、それ以外はユーザーの選択をチェックポイント
- エラーハンドリング: チェックポイント前に壊れた環境を修正、セットアップ失敗でチェックポイントを提示しない

</checkpoint_automation_reference>
