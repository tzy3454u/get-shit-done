<purpose>
並列のコードベースマッパーエージェントを統率してコードベースを分析し、.planning/codebase/ に構造化ドキュメントを生成する。

各エージェントは新しいコンテキストを持ち、特定の焦点領域を探索し、**ドキュメントを直接書き込む**。オーケストレーターは確認と行数のみを受け取り、サマリーを書き込む。

出力: コードベースの状態に関する7つの構造化ドキュメントを含む .planning/codebase/ フォルダ。
</purpose>

<philosophy>
**専用マッパーエージェントを使う理由：**
- ドメインごとに新しいコンテキスト（トークンの汚染なし）
- エージェントがドキュメントを直接書き込む（オーケストレーターへのコンテキスト転送なし）
- オーケストレーターは作成されたものを要約するのみ（最小限のコンテキスト使用）
- より高速な実行（エージェントが同時に実行される）

**長さよりドキュメントの品質：**
参考資料として有用な十分な詳細を含める。任意な簡潔さよりも実践的な例（特にコードパターン）を優先する。

**常にファイルパスを含める：**
ドキュメントは計画/実行時にClaudeが参照する資料である。バッククォートでフォーマットした実際のファイルパスを常に含める：`src/services/user.ts`。
</philosophy>

<process>

<step name="init_context" priority="first">
コードベースマッピングのコンテキストを読み込む：

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init map-codebase)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから抽出する：`mapper_model`、`commit_docs`、`codebase_dir`、`existing_maps`、`has_maps`、`codebase_dir_exists`。
</step>

<step name="check_existing">
initコンテキストの `has_maps` を使用して .planning/codebase/ が既に存在するか確認する。

`codebase_dir_exists` がtrueの場合：
```bash
ls -la .planning/codebase/
```

**存在する場合：**

```
.planning/codebase/ は既に以下のドキュメントで存在しています：
[見つかったファイルの一覧]

次のアクション：
1. リフレッシュ - 既存を削除してコードベースを再マッピング
2. 更新 - 既存を保持し、特定のドキュメントのみ更新
3. スキップ - 既存のコードベースマップをそのまま使用
```

ユーザーの返信を待つ。

"リフレッシュ" の場合：.planning/codebase/ を削除し、create_structureに進む
"更新" の場合：どのドキュメントを更新するか確認し、spawn_agents に進む（フィルタ済み）
"スキップ" の場合：ワークフローを終了する

**存在しない場合：**
create_structureに進む。
</step>

<step name="create_structure">
.planning/codebase/ ディレクトリを作成する：

```bash
mkdir -p .planning/codebase
```

**期待される出力ファイル：**
- STACK.md（techマッパーから）
- INTEGRATIONS.md（techマッパーから）
- ARCHITECTURE.md（archマッパーから）
- STRUCTURE.md（archマッパーから）
- CONVENTIONS.md（qualityマッパーから）
- TESTING.md（qualityマッパーから）
- CONCERNS.md（concernsマッパーから）

spawn_agentsに進む。
</step>

<step name="spawn_agents">
4つの並列gsd-codebase-mapperエージェントを生成する。

Task toolを `subagent_type="gsd-codebase-mapper"`、`model="{mapper_model}"`、`run_in_background=true` で使用して並列実行する。

**重要：** 専用の `gsd-codebase-mapper` エージェントを使用する。`Explore` ではない。マッパーエージェントはドキュメントを直接書き込む。

**Agent 1: Tech Focus**

```
Task(
  subagent_type="gsd-codebase-mapper",
  model="{mapper_model}",
  run_in_background=true,
  description="Map codebase tech stack",
  prompt="Focus: tech

このコードベースの技術スタックと外部連携を分析する。

以下のドキュメントを .planning/codebase/ に書き込む：
- STACK.md - 言語、ランタイム、フレームワーク、依存関係、設定
- INTEGRATIONS.md - 外部API、データベース、認証プロバイダー、Webhook

徹底的に探索する。テンプレートを使ってドキュメントを直接書き込む。確認のみ返す。"
)
```

**Agent 2: Architecture Focus**

```
Task(
  subagent_type="gsd-codebase-mapper",
  model="{mapper_model}",
  run_in_background=true,
  description="Map codebase architecture",
  prompt="Focus: arch

このコードベースのアーキテクチャとディレクトリ構造を分析する。

以下のドキュメントを .planning/codebase/ に書き込む：
- ARCHITECTURE.md - パターン、レイヤー、データフロー、抽象化、エントリポイント
- STRUCTURE.md - ディレクトリレイアウト、主要な場所、命名規則

徹底的に探索する。テンプレートを使ってドキュメントを直接書き込む。確認のみ返す。"
)
```

**Agent 3: Quality Focus**

```
Task(
  subagent_type="gsd-codebase-mapper",
  model="{mapper_model}",
  run_in_background=true,
  description="Map codebase conventions",
  prompt="Focus: quality

このコードベースのコーディング規約とテストパターンを分析する。

以下のドキュメントを .planning/codebase/ に書き込む：
- CONVENTIONS.md - コードスタイル、命名、パターン、エラーハンドリング
- TESTING.md - フレームワーク、構造、モック、カバレッジ

徹底的に探索する。テンプレートを使ってドキュメントを直接書き込む。確認のみ返す。"
)
```

**Agent 4: Concerns Focus**

```
Task(
  subagent_type="gsd-codebase-mapper",
  model="{mapper_model}",
  run_in_background=true,
  description="Map codebase concerns",
  prompt="Focus: concerns

このコードベースの技術的負債、既知の問題、懸念領域を分析する。

以下のドキュメントを .planning/codebase/ に書き込む：
- CONCERNS.md - 技術的負債、バグ、セキュリティ、パフォーマンス、脆弱な領域

徹底的に探索する。テンプレートを使ってドキュメントを直接書き込む。確認のみ返す。"
)
```

collect_confirmationsに進む。
</step>

<step name="collect_confirmations">
4つのエージェントすべてが完了するのを待つ。

各エージェントの出力ファイルを読み込み、確認を収集する。

**各エージェントからの期待される確認フォーマット：**
```
## マッピング完了

**Focus:** {focus}
**書き込まれたドキュメント：**
- `.planning/codebase/{DOC1}.md` ({N} 行)
- `.planning/codebase/{DOC2}.md` ({N} 行)

オーケストレーターのサマリー準備完了。
```

**受け取るもの：** ファイルパスと行数のみ。ドキュメントの内容ではない。

エージェントが失敗した場合、失敗を記録し、成功したドキュメントで続行する。

verify_outputに進む。
</step>

<step name="verify_output">
すべてのドキュメントが正常に作成されたか検証する：

```bash
ls -la .planning/codebase/
wc -l .planning/codebase/*.md
```

**検証チェックリスト：**
- 7つのドキュメントすべてが存在する
- 空のドキュメントがない（各20行以上であるべき）

ドキュメントが不足または空の場合、どのエージェントが失敗した可能性があるか記録する。

scan_for_secretsに進む。
</step>

<step name="scan_for_secrets">
**重要なセキュリティチェック：** コミット前に生成されたファイルに誤って漏洩したシークレットがないかスキャンする。

シークレットパターン検出を実行する：

```bash
# 生成されたドキュメント内の一般的なAPIキーパターンを確認
grep -E '(sk-[a-zA-Z0-9]{20,}|sk_live_[a-zA-Z0-9]+|sk_test_[a-zA-Z0-9]+|ghp_[a-zA-Z0-9]{36}|gho_[a-zA-Z0-9]{36}|glpat-[a-zA-Z0-9_-]+|AKIA[A-Z0-9]{16}|xox[baprs]-[a-zA-Z0-9-]+|-----BEGIN.*PRIVATE KEY|eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.)' .planning/codebase/*.md 2>/dev/null && SECRETS_FOUND=true || SECRETS_FOUND=false
```

**SECRETS_FOUND=trueの場合：**

```
⚠️  セキュリティ警告: コードベースドキュメントに潜在的なシークレットが検出されました！

以下にAPIキーまたはトークンのようなパターンが見つかりました：
[grep出力を表示]

コミットすると認証情報が露出します。

**必要なアクション：**
1. 上記のフラグが立ったコンテンツを確認してください
2. これらが本物のシークレットの場合、コミット前に削除する必要があります
3. 機密ファイルをClaude Codeの "Deny" パーミッションに追加することを検討してください

コミット前に一時停止しています。フラグが立ったコンテンツが実際には機密でない場合は "safe to proceed" と返信するか、先にファイルを編集してください。
```

続行する前にユーザーの確認を待つ。

**SECRETS_FOUND=falseの場合：**

commit_codebase_mapに進む。
</step>

<step name="commit_codebase_map">
コードベースマップをコミットする：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: map existing codebase" --files .planning/codebase/*.md
```

offer_nextに進む。
</step>

<step name="offer_next">
完了サマリーと次のステップを表示する。

**行数を取得する：**
```bash
wc -l .planning/codebase/*.md
```

**出力フォーマット：**

```
コードベースマッピング完了。

.planning/codebase/ を作成しました：
- STACK.md ([N] 行) - 技術と依存関係
- ARCHITECTURE.md ([N] 行) - システム設計とパターン
- STRUCTURE.md ([N] 行) - ディレクトリレイアウトと構成
- CONVENTIONS.md ([N] 行) - コードスタイルとパターン
- TESTING.md ([N] 行) - テスト構造とプラクティス
- INTEGRATIONS.md ([N] 行) - 外部サービスとAPI
- CONCERNS.md ([N] 行) - 技術的負債と問題


---

## ▶ Next Up

**プロジェクトを初期化** — コードベースのコンテキストを計画に活用

`/gsd:new-project`

<sub>`/clear` first → fresh context window</sub>

---

**その他のオプション：**
- マッピングを再実行: `/gsd:map-codebase`
- 特定のファイルを確認: `cat .planning/codebase/STACK.md`
- 続行前に任意のドキュメントを編集

---
```

ワークフローを終了する。
</step>

</process>

<success_criteria>
- .planning/codebase/ ディレクトリが作成されている
- 4つの並列gsd-codebase-mapperエージェントがrun_in_background=trueで生成されている
- エージェントがドキュメントを直接書き込む（オーケストレーターはドキュメントの内容を受け取らない）
- エージェントの出力ファイルを読み込み確認を収集する
- 7つのコードベースドキュメントすべてが存在する
- 行数付きの明確な完了サマリー
- GSDスタイルの明確な次のステップがユーザーに提示されている
</success_criteria>
</output>
