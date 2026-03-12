<overview>
TDDは、カバレッジメトリクスではなく設計品質のためのものです。Red-Green-Refactorサイクルは、実装前に動作について考えることを強制し、よりクリーンなインターフェースとよりテスト可能なコードを生み出します。

**原則:** `expect(fn(input)).toBe(output)`で動作を`fn`を書く前に記述できるなら、TDDは結果を改善します。

**重要な洞察:** TDDの作業は標準的なタスクよりも根本的に重いです — 2-3回の実行サイクル（RED → GREEN → REFACTOR）が必要で、それぞれにファイル読み取り、テスト実行、潜在的なデバッグが含まれます。TDDの機能はサイクル全体を通じて完全なコンテキストを確保するために専用のプランを取得します。
</overview>

<when_to_use_tdd>
## TDDが品質を改善する場合

**TDD候補（TDDプランを作成）:**
- 定義された入力/出力のあるビジネスロジック
- リクエスト/レスポンス契約のあるAPIエンドポイント
- データ変換、パース、フォーマット
- バリデーションルールと制約
- テスト可能な動作のあるアルゴリズム
- ステートマシンとワークフロー
- 明確な仕様のあるユーティリティ関数

**TDDをスキップ（`type="auto"`タスクの標準プランを使用）:**
- UIレイアウト、スタイリング、ビジュアルコンポーネント
- 設定変更
- 既存コンポーネントを接続するグルーコード
- ワンタイムスクリプトとマイグレーション
- ビジネスロジックのないシンプルなCRUD
- 探索的プロトタイピング

**ヒューリスティック:** `fn`を書く前に`expect(fn(input)).toBe(output)`と書けますか？
→ はい: TDDプランを作成
→ いいえ: 標準プランを使用、必要に応じて後でテストを追加
</when_to_use_tdd>

<tdd_plan_structure>
## TDDプラン構造

各TDDプランは、完全なRED-GREEN-REFACTORサイクルを通じて**1つの機能**を実装します。

```markdown
---
phase: XX-name
plan: NN
type: tdd
---

<objective>
[どの機能でなぜか]
Purpose: [この機能へのTDDの設計上のメリット]
Output: [動作する、テスト済みの機能]
</objective>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@relevant/source/files.ts
</context>

<feature>
  <name>[機能名]</name>
  <files>[ソースファイル, テストファイル]</files>
  <behavior>
    [テスト可能な用語での期待される動作]
    Cases: input → expected output
  </behavior>
  <implementation>[テストがパスした後の実装方法]</implementation>
</feature>

<verification>
[機能の動作を証明するテストコマンド]
</verification>

<success_criteria>
- 失敗するテストが書かれてコミットされた
- 実装がテストをパスする
- リファクタリング完了（必要な場合）
- 2-3のコミットがすべて存在
</success_criteria>

<output>
完了後、以下の内容でSUMMARY.mdを作成:
- RED: どのテストが書かれ、なぜ失敗したか
- GREEN: どの実装がテストをパスさせたか
- REFACTOR: どのクリーンアップが行われたか（もしあれば）
- Commits: 生成されたコミットのリスト
</output>
```

**1つのTDDプランにつき1つの機能。** 機能がバッチ処理できるほど些細なら、TDDをスキップするほど些細です — 標準プランを使用して後でテストを追加してください。
</tdd_plan_structure>

<execution_flow>
## Red-Green-Refactorサイクル

**RED - 失敗するテストを書く:**
1. プロジェクトの慣例に従ってテストファイルを作成
2. 期待される動作を記述するテストを書く（`<behavior>`要素から）
3. テストを実行 - 必ず失敗すること
4. テストがパスする場合: 機能が既に存在するかテストが間違っている。調査する。
5. コミット: `test({phase}-{plan}): add failing test for [feature]`

**GREEN - パスするよう実装:**
1. テストをパスさせるための最小限のコードを書く
2. 賢いことはしない、最適化もしない - とにかく動かす
3. テストを実行 - 必ずパスすること
4. コミット: `feat({phase}-{plan}): implement [feature]`

**REFACTOR（必要な場合）:**
1. 明らかな改善がある場合、実装をクリーンアップ
2. テストを実行 - 依然としてパスすること
3. 変更があった場合のみコミット: `refactor({phase}-{plan}): clean up [feature]`

**結果:** 各TDDプランは2-3のアトミックコミットを生成します。
</execution_flow>

<test_quality>
## 良いテストと悪いテスト

**実装ではなく動作をテストする:**
- 良い: 「フォーマットされた日付文字列を返す」
- 悪い: 「正しいパラメータでformatDateヘルパーを呼び出す」
- テストはリファクタリングを生き延びるべき

**テストごとに1つの概念:**
- 良い: 有効な入力、空の入力、不正な入力それぞれに個別のテスト
- 悪い: 複数のアサーションですべてのエッジケースをチェックする単一テスト

**説明的な名前:**
- 良い: "should reject empty email", "returns null for invalid ID"
- 悪い: "test1", "handles error", "works correctly"

**実装の詳細を含めない:**
- 良い: パブリックAPI、観察可能な動作をテスト
- 悪い: 内部をモック、プライベートメソッドをテスト、内部状態にアサート
</test_quality>

<framework_setup>
## テストフレームワークセットアップ（存在しない場合）

TDDプランを実行するがテストフレームワークが設定されていない場合、REDフェーズの一部としてセットアップします:

**1. プロジェクトタイプを検出:**
```bash
# JavaScript/TypeScript
if [ -f package.json ]; then echo "node"; fi

# Python
if [ -f requirements.txt ] || [ -f pyproject.toml ]; then echo "python"; fi

# Go
if [ -f go.mod ]; then echo "go"; fi

# Rust
if [ -f Cargo.toml ]; then echo "rust"; fi
```

**2. 最小限のフレームワークをインストール:**
| プロジェクト | フレームワーク | インストール |
|---------|-----------|---------|
| Node.js | Jest | `npm install -D jest @types/jest ts-jest` |
| Node.js (Vite) | Vitest | `npm install -D vitest` |
| Python | pytest | `pip install pytest` |
| Go | testing | 組み込み |
| Rust | cargo test | 組み込み |

**3. 必要に応じて設定を作成:**
- Jest: ts-jestプリセットの`jest.config.js`
- Vitest: テストグローバルの`vitest.config.ts`
- pytest: `pytest.ini`または`pyproject.toml`セクション

**4. セットアップを検証:**
```bash
# 空のテストスイートを実行 - 0テストでパスすべき
npm test  # Node
pytest    # Python
go test ./...  # Go
cargo test    # Rust
```

**5. 最初のテストファイルを作成:**
テストの場所はプロジェクトの慣例に従う:
- ソースの隣の`*.test.ts` / `*.spec.ts`
- `__tests__/`ディレクトリ
- ルートの`tests/`ディレクトリ

フレームワークセットアップは最初のTDDプランのREDフェーズに含まれるワンタイムコストです。
</framework_setup>

<error_handling>
## エラーハンドリング

**REDフェーズでテストが失敗しない:**
- 機能が既に存在する可能性 - 調査する
- テストが間違っている可能性（思っていることをテストしていない）
- 先に進む前に修正する

**GREENフェーズでテストがパスしない:**
- 実装をデバッグする
- リファクタリングにスキップしない
- グリーンになるまで繰り返す

**REFACTORフェーズでテストが失敗:**
- リファクタリングを元に戻す
- コミットが早すぎた
- より小さなステップでリファクタリング

**関係のないテストが壊れる:**
- 停止して調査する
- カップリングの問題を示している可能性
- 先に進む前に修正する
</error_handling>

<commit_pattern>
## TDDプランのコミットパターン

TDDプランは2-3のアトミックコミットを生成します（フェーズごとに1つ）:

```
test(08-02): add failing test for email validation

- Tests valid email formats accepted
- Tests invalid formats rejected
- Tests empty input handling

feat(08-02): implement email validation

- Regex pattern matches RFC 5322
- Returns boolean for validity
- Handles edge cases (empty, null)

refactor(08-02): extract regex to constant (optional)

- Moved pattern to EMAIL_REGEX constant
- No behavior changes
- Tests still pass
```

**標準プランとの比較:**
- 標準プラン: タスクごとに1コミット、プランごとに2-4コミット
- TDDプラン: 単一機能に2-3コミット

両方とも同じフォーマットに従う: `{type}({phase}-{plan}): {description}`

**メリット:**
- 各コミットが独立して取り消し可能
- Git bisectがコミットレベルで動作
- TDDの規律を示す明確な履歴
- 全体的なコミット戦略と一貫
</commit_pattern>

<context_budget>
## コンテキストバジェット

TDDプランは**約40%のコンテキスト使用量**を目標とします（標準プランの約50%より低い）。

低い理由:
- REDフェーズ: テストを書く、テストを実行、なぜ失敗しなかったかのデバッグの可能性
- GREENフェーズ: 実装する、テストを実行、失敗に対する反復の可能性
- REFACTORフェーズ: コードを修正、テストを実行、リグレッションがないことを確認

各フェーズにはファイル読み取り、コマンド実行、出力分析が含まれます。このやり取りは線形的なタスク実行よりも本質的に重いです。

単一機能に焦点を当てることで、サイクル全体を通じた完全な品質を確保します。
</context_budget>