# 構造テンプレート

`.planning/codebase/STRUCTURE.md` 用テンプレート - 物理的なファイル構成を記録します。

**目的:** コードベース内でファイルが物理的にどこに配置されているかを文書化します。「Xはどこに置くべきか？」に回答します。

---

## ファイルテンプレート

```markdown
# コードベース構造

**分析日:** [YYYY-MM-DD]

## ディレクトリレイアウト

[トップレベルディレクトリの目的を示すASCIIボックスドローイングツリー - ツリー構造には ├── └── │ 文字を使用]

```
[project-root]/
├── [dir]/          # [目的]
├── [dir]/          # [目的]
├── [dir]/          # [目的]
└── [file]          # [目的]
```

## ディレクトリの目的

**[ディレクトリ名]:**
- Purpose: [ここに何があるか]
- Contains: [ファイルの種類: 例: "*.tsソースファイル", "コンポーネントディレクトリ"]
- Key files: [このディレクトリの重要なファイル]
- Subdirectories: [ネストがある場合、構造を説明]

**[ディレクトリ名]:**
- Purpose: [ここに何があるか]
- Contains: [ファイルの種類]
- Key files: [重要なファイル]
- Subdirectories: [構造]

## 主要なファイルの場所

**エントリポイント:**
- [パス]: [目的: 例: "CLIエントリポイント"]
- [パス]: [目的: 例: "サーバー起動"]

**設定:**
- [パス]: [目的: 例: "TypeScript設定"]
- [パス]: [目的: 例: "ビルド設定"]
- [パス]: [目的: 例: "環境変数"]

**コアロジック:**
- [パス]: [目的: 例: "ビジネスサービス"]
- [パス]: [目的: 例: "データベースモデル"]
- [パス]: [目的: 例: "APIルート"]

**テスト:**
- [パス]: [目的: 例: "ユニットテスト"]
- [パス]: [目的: 例: "テストフィクスチャ"]

**ドキュメント:**
- [パス]: [目的: 例: "ユーザー向けドキュメント"]
- [パス]: [目的: 例: "開発者ガイド"]

## 命名規約

**ファイル:**
- [パターン]: [例: 例: "モジュールにkebab-case.ts"]
- [パターン]: [例: 例: "ReactコンポーネントにPascalCase.tsx"]
- [パターン]: [例: 例: "テストファイルに*.test.ts"]

**ディレクトリ:**
- [パターン]: [例: 例: "機能ディレクトリにkebab-case"]
- [パターン]: [例: 例: "コレクションに複数形"]

**特殊パターン:**
- [パターン]: [例: 例: "ディレクトリエクスポートにindex.ts"]
- [パターン]: [例: 例: "テストディレクトリに__tests__"]

## 新しいコードの追加場所

**新機能:**
- Primary code: [ディレクトリパス]
- Tests: [ディレクトリパス]
- Config if needed: [ディレクトリパス]

**新しいコンポーネント/モジュール:**
- Implementation: [ディレクトリパス]
- Types: [ディレクトリパス]
- Tests: [ディレクトリパス]

**新しいルート/コマンド:**
- Definition: [ディレクトリパス]
- Handler: [ディレクトリパス]
- Tests: [ディレクトリパス]

**ユーティリティ:**
- Shared helpers: [ディレクトリパス]
- Type definitions: [ディレクトリパス]

## 特殊ディレクトリ

[特別な意味や生成方法を持つディレクトリ]

**[ディレクトリ]:**
- Purpose: [例: "生成されたコード", "ビルド出力"]
- Source: [例: "Xにより自動生成", "ビルドアーティファクト"]
- Committed: [はい/いいえ - .gitignoreに含まれるか？]

---

*構造分析: [日付]*
*ディレクトリ構造が変更された際に更新*
```

<good_examples>
```markdown
# コードベース構造

**分析日:** 2025-01-20

## ディレクトリレイアウト

```
get-shit-done/
├── bin/                # 実行可能エントリポイント
├── commands/           # スラッシュコマンド定義
│   └── gsd/           # GSD固有のコマンド
├── get-shit-done/     # スキルリソース
│   ├── references/    # 原則ドキュメント
│   ├── templates/     # ファイルテンプレート
│   └── workflows/     # 複数ステップの手順
├── src/               # ソースコード（該当する場合）
├── tests/             # テストファイル
├── package.json       # プロジェクトマニフェスト
└── README.md          # ユーザードキュメント
```

## ディレクトリの目的

**bin/**
- Purpose: CLIエントリポイント
- Contains: install.js（インストーラースクリプト）
- Key files: install.js - npxインストールを処理
- Subdirectories: なし

**commands/gsd/**
- Purpose: Claude Code用のスラッシュコマンド定義
- Contains: *.mdファイル（コマンドごとに1つ）
- Key files: new-project.md, plan-phase.md, execute-plan.md
- Subdirectories: なし（フラット構造）

**get-shit-done/references/**
- Purpose: コア哲学とガイダンスドキュメント
- Contains: principles.md, questioning.md, plan-format.md
- Key files: principles.md - システム哲学
- Subdirectories: なし

**get-shit-done/templates/**
- Purpose: .planning/ファイル用のドキュメントテンプレート
- Contains: フロントマター付きテンプレート定義
- Key files: project.md, roadmap.md, plan.md, summary.md
- Subdirectories: codebase/（新規 - スタック/アーキテクチャ/構造テンプレート用）

**get-shit-done/workflows/**
- Purpose: 再利用可能な複数ステップの手順
- Contains: コマンドから呼び出されるワークフロー定義
- Key files: execute-plan.md, research-phase.md
- Subdirectories: なし

## 主要なファイルの場所

**エントリポイント:**
- `bin/install.js` - インストールスクリプト（npxエントリ）

**設定:**
- `package.json` - プロジェクトメタデータ、依存関係、binエントリ
- `.gitignore` - 除外ファイル

**コアロジック:**
- `bin/install.js` - すべてのインストールロジック（ファイルコピー、パス置換）

**テスト:**
- `tests/` - テストファイル（存在する場合）

**ドキュメント:**
- `README.md` - ユーザー向けインストールおよび使用ガイド
- `CLAUDE.md` - このリポジトリで作業する際のClaude Code向け指示

## 命名規約

**ファイル:**
- kebab-case.md: Markdownドキュメント
- kebab-case.js: JavaScriptソースファイル
- UPPERCASE.md: 重要なプロジェクトファイル（README, CLAUDE, CHANGELOG）

**ディレクトリ:**
- kebab-case: すべてのディレクトリ
- コレクションに複数形: templates/, commands/, workflows/

**特殊パターン:**
- {command-name}.md: スラッシュコマンド定義
- *-template.md: 使用可能だがtemplates/ディレクトリを優先

## 新しいコードの追加場所

**新しいスラッシュコマンド:**
- Primary code: `commands/gsd/{command-name}.md`
- Tests: `tests/commands/{command-name}.test.js`（テストが実装されている場合）
- Documentation: 新しいコマンドで `README.md` を更新

**新しいテンプレート:**
- Implementation: `get-shit-done/templates/{name}.md`
- Documentation: テンプレートは自己文書化（ガイドラインを含む）

**新しいワークフロー:**
- Implementation: `get-shit-done/workflows/{name}.md`
- Usage: コマンドから `@~/.claude/get-shit-done/workflows/{name}.md` で参照

**新しいリファレンスドキュメント:**
- Implementation: `get-shit-done/references/{name}.md`
- Usage: 必要に応じてコマンド/ワークフローから参照

**ユーティリティ:**
- まだユーティリティなし（`install.js`はモノリシック）
- 抽出する場合: `src/utils/`

## 特殊ディレクトリ

**get-shit-done/**
- Purpose: ~/.claude/にインストールされるリソース
- Source: インストール時にbin/install.jsがコピー
- Committed: はい（信頼できる唯一のソース）

**commands/**
- Purpose: ~/.claude/commands/にインストールされるスラッシュコマンド
- Source: インストール時にbin/install.jsがコピー
- Committed: はい（信頼できる唯一のソース）

---

*構造分析: 2025-01-20*
*ディレクトリ構造が変更された際に更新*
```
</good_examples>

<guidelines>
**STRUCTURE.mdに含めるもの:**
- ディレクトリレイアウト（構造の視覚化にASCIIボックスドローイングツリー）
- 各ディレクトリの目的
- 主要なファイルの場所（エントリポイント、設定、コアロジック）
- 命名規約
- 新しいコードの追加場所（種類別）
- 特殊/生成ディレクトリ

**ここに含めないもの:**
- 概念的なアーキテクチャ（ARCHITECTURE.mdの範囲）
- 技術スタック（STACK.mdの範囲）
- コードの実装詳細（コードリーディングに委ねる）
- すべてのファイル（ディレクトリと主要ファイルに焦点）

**このテンプレートを記入する際:**
- `tree -L 2`または類似コマンドで構造を視覚化
- トップレベルディレクトリとその目的を特定
- 既存ファイルを観察して命名パターンに注目
- エントリポイント、設定、メインロジック領域を特定
- ディレクトリツリーを簡潔に保つ（最大2〜3レベル）

**ツリーフォーマット（構造のみにASCIIボックスドローイング文字）:**
```
root/
├── dir1/           # 目的
│   ├── subdir/    # 目的
│   └── file.ts    # 目的
├── dir2/          # 目的
└── file.ts        # 目的
```

**フェーズ計画で役立つ場面:**
- 新機能の追加（ファイルはどこに置くべきか？）
- プロジェクト構成の理解
- 特定のロジックがどこにあるかの発見
- 既存の規約に従う
</guidelines>
