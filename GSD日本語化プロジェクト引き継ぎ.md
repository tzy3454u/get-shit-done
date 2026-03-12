# GSD (Get Shit Done) 日本語化プロジェクト 引き継ぎ文書

## 背景

[GSD (Get Shit Done)](https://github.com/gsd-build/get-shit-done) は Claude Code 向けのメタプロンプティング＆Spec-Driven Development システム。GitHub Stars 約23k（2026年3月時点）で、SDD系ツールの中で最も人気がある。

類似ツールの [cc-sdd](https://github.com/gotalab/cc-sdd) は13言語対応（日本語含む）だが、GSD は英語のみ。GSD のプロンプトを日本語化することで、英語が苦手なユーザーでも使えるようにする。

## GSD の仕組み（翻訳に必要な理解）

GSD は**コードを実行するアプリではなく、プロンプト（指示文）の集合体**。

```
npx get-shit-done-cc@latest
  ↓ インストーラーがプロンプトファイルを ~/.claude/ にコピー
  ↓
ユーザーが /gsd:new-project 等のスラッシュコマンドを実行
  ↓
Claude Code が対応するプロンプト（Markdown）を読み込む
  ↓
Claude がプロンプトの指示に従って作業する
```

つまり、**プロンプトの中の英語テキストを日本語に翻訳すれば、そのまま動く**。

## リポジトリ構成と翻訳対象

### 翻訳対象ファイル（優先度順）

#### 1. コマンド定義（32ファイル）— 最優先
- パス: `commands/gsd/*.md`
- 役割: スラッシュコマンドの定義。YAML frontmatter + XML構造のプロンプト
- 翻訳箇所: `description`, `<objective>`, `<process>` 内の英語テキスト
- 注意: `name` フィールド（コマンド名）は英語のまま維持

主なコマンド:
| コマンド | 役割 |
|---------|------|
| `new-project.md` | プロジェクト初期化 |
| `plan-phase.md` | フェーズ計画 |
| `execute-phase.md` | フェーズ実行 |
| `research-phase.md` | リサーチ |
| `discuss-phase.md` | ディスカッション |
| `validate-phase.md` | 検証 |
| `verify-work.md` | 作業確認 |
| `debug.md` | デバッグ |
| `help.md` | ヘルプ表示 |
| `progress.md` | 進捗確認 |
| 他22ファイル | |

#### 2. ワークフロー定義（35ファイル）— 最優先
- パス: `get-shit-done/workflows/*.md`
- 役割: 各コマンドの詳細な実行手順。コマンドから `@` 参照で読み込まれる
- 翻訳箇所: 指示文、説明文、ユーザーへの出力テンプレート

#### 3. エージェント定義（12ファイル）— 高優先度
- パス: `agents/*.md`
- 役割: サブエージェント（executor, planner, debugger等）の振る舞い定義
- 翻訳箇所: `<role>`, 指示文、出力フォーマット

エージェント一覧:
- `gsd-executor.md` — 計画実行
- `gsd-planner.md` — 計画作成
- `gsd-debugger.md` — デバッグ
- `gsd-verifier.md` — 検証
- `gsd-phase-researcher.md` — フェーズリサーチ
- `gsd-project-researcher.md` — プロジェクトリサーチ
- `gsd-research-synthesizer.md` — リサーチ統合
- `gsd-roadmapper.md` — ロードマップ作成
- `gsd-plan-checker.md` — 計画チェック
- `gsd-codebase-mapper.md` — コードベース分析
- `gsd-integration-checker.md` — 統合チェック
- `gsd-nyquist-auditor.md` — 品質監査

#### 4. テンプレート（25ファイル）— 中優先度
- パス: `get-shit-done/templates/*.md`
- 役割: 生成されるドキュメント（PROJECT.md, REQUIREMENTS.md等）のテンプレート
- 翻訳箇所: 見出し、説明文、プレースホルダーテキスト

#### 5. リファレンス（14ファイル）— 中優先度
- パス: `get-shit-done/references/*.md`
- 役割: Git連携、検証パターン、プロファイル設定等のルール定義
- 翻訳箇所: 説明文、コメント

#### 6. フック（3ファイル）— 翻訳不要
- パス: `hooks/*.js`
- 役割: JavaScript実行コード（コンテキスト監視、アップデートチェック等）
- 翻訳不要: 実行コードであり、ユーザーに見える文字列はほぼない

#### 7. インストーラー/ビルド — 翻訳不要
- パス: `bin/`, `scripts/`, `package.json`
- 翻訳不要: ファイルを配置するだけのコード

### 翻訳してはいけない部分

- YAML frontmatter の `name` フィールド（コマンド名/エージェント名）
- XMLタグ名（`<objective>`, `<process>`, `<task>`, `<verify>` 等）
- `@` ファイル参照パス（`@~/.claude/get-shit-done/...`）
- `allowed-tools` のツール名（`Read`, `Write`, `Bash` 等）
- コード例・コマンド例の中身
- ファイル名・ディレクトリ名

## 翻訳ファイル例

### 翻訳前（commands/gsd/new-project.md 抜粋）
```markdown
---
name: gsd:new-project
description: Initialize a new project with deep context gathering and PROJECT.md
---
<objective>
Initialize a new project through unified flow: questioning → research → requirements → roadmap.
</objective>
```

### 翻訳後
```markdown
---
name: gsd:new-project
description: プロジェクトの初期化。深いコンテキスト収集とPROJECT.mdの作成
---
<objective>
統合フローによるプロジェクト初期化: 質問 → リサーチ → 要件定義 → ロードマップ作成。
</objective>
```

## 推奨アプローチ

1. **Fork して作業する** — `gsd-build/get-shit-done` を Fork
2. **最小セットから始める** — まず以下の5コマンド + 関連ワークフロー + 関連エージェントを翻訳して動作確認:
   - `new-project.md` → プロジェクト初期化
   - `plan-phase.md` → 計画
   - `execute-phase.md` → 実行
   - `help.md` → ヘルプ
   - `progress.md` → 進捗
3. **動作確認してから残りを翻訳** — 日本語プロンプトで Claude が正しく動くか確認
4. **テンプレート・リファレンスを翻訳** — 生成ドキュメントも日本語化

## 翻訳量の見積もり

| カテゴリ | ファイル数 | 備考 |
|---------|-----------|------|
| コマンド定義 | 32 | 各ファイル小〜中サイズ |
| ワークフロー | 35 | 各ファイル中〜大サイズ（メイン作業） |
| エージェント | 12 | 各ファイル中サイズ |
| テンプレート | 25 | 各ファイル小〜中サイズ |
| リファレンス | 14 | 各ファイル小〜中サイズ |
| **合計** | **約120ファイル** | フック・スクリプト除く |

## 参考リンク

- GSD リポジトリ: https://github.com/gsd-build/get-shit-done
- GSD 公式サイト: https://gsd.build/
- cc-sdd（日本語対応の競合）: https://github.com/gotalab/cc-sdd
- GSD Deep Dive: https://www.codecentric.de/en/knowledge-hub/blog/the-anatomy-of-claude-code-workflows-turning-slash-commands-into-an-ai-development-system

## 次セッション開始チェックリスト（第4パス）

### 1) 最初に再読する対象

- [ ] `get-shit-done/workflows/add-phase.md`
- [ ] `get-shit-done/workflows/add-tests.md`
- [ ] `get-shit-done/workflows/research-phase.md`

### 2) 第4パスの方針

- [ ] 「常体への統一」を継続する
- [ ] ユーザー対話文（質問文・選択肢）は自然な丁寧表現を許容する
- [ ] 機械置換は行わず、手修正で進める

優先順:
- [ ] 手順・命令文（すること／終了）
- [ ] `required_reading` / `process` の指示文
- [ ] 説明段落（です・ます混在の最小修正）

### 3) 残件の扱い

現状認識:
- [ ] 「ください」残件は3件（対話文中心）
- [ ] 「です・ます」残件は多数（説明文に広く残存）
- [ ] 全置換は行わず、対話文を除外しながら段階的に減らす

直近の第4パス候補:
- [ ] `get-shit-done/workflows/list-phase-assumptions.md:120`
- [ ] `get-shit-done/workflows/new-milestone.md:327`
- [ ] `get-shit-done/workflows/new-project.md:984`

補足:
- 上記3件は現時点でいずれも対話文（質問文・選択肢）に該当するため、方針上は許容範囲。

### 4) 品質ゲート

修正後に毎回確認する:
- [ ] タグ整合（`<output>` タグの開閉不一致なし）
- [ ] 壊れトークンなし（例: `output` 終端の崩れ）
- [ ] 対話文の自然さを維持

### 5) フェーズ状況

- [x] 第1パス: 完了（構造破損の修正）
- [x] 第2パス: 完了（高優先の意味ずれ・壊れ箇所修正）
- [x] 第3パス: 完了（文体統一の一次対応）
- [~] 第4パス: 進行中（commands/workflowsの広範囲修正と品質ゲート再実施まで完了）

## 第4パス進捗メモ（2026-03-12 更新）

### A) commands 再スイープ結果

- `commands/gsd/*.md` の `です|ます` 残件: 6件
- 分類結果: `DIALOG_LIKE=6`, `NON_DIALOG_LIKE=0`
- 結論: すべて対話文・質問文・選択肢のため、方針上は編集不要

### B) workflows 最終品質ゲート結果

- `get-shit-done/workflows/*.md` で `<output>` 開閉数を全件チェック
- 孤立 `</output>` を5ファイルで解消:
  - `cleanup.md`, `complete-milestone.md`, `diagnose-issues.md`, `execute-phase.md`, `map-codebase.md`
- 壊れトークン `</output>",` を1ファイルで修正:
  - `research-phase.md`
- 破損トークン再スキャン後、タグ断片の崩れは検出なし
- 変更ファイル diagnostics: エラーなし

### C) workflows 残件分類（最新）

- `です|ます` 総数: `TOTAL=96`
- 対話系: `DIALOG_LIKE=96`
- 非対話候補: `NON_DIALOG_LIKE=0`

補足:
- 非対話候補は解消済み。残件は対話文・質問文・選択肢のみ。
- 第4パスは、方針（対話文の自然さ維持）に沿ってクローズ可能な状態。
