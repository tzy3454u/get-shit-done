---
description: GSD アップデート後にローカル変更を再適用する
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

<purpose>
GSD アップデートによりファイルが消去・再インストールされた後、ユーザーが以前保存したローカル変更を新しいバージョンにマージする。上流ファイルも変更されたケースに対応するためインテリジェントな比較を使用する。
</purpose>

<process>

## ステップ 1: バックアップされたパッチの検出

ローカルパッチディレクトリを確認する：

```bash
# Global install — detect runtime config directory
if [ -d "$HOME/.config/opencode/gsd-local-patches" ]; then
  PATCHES_DIR="$HOME/.config/opencode/gsd-local-patches"
elif [ -d "$HOME/.opencode/gsd-local-patches" ]; then
  PATCHES_DIR="$HOME/.opencode/gsd-local-patches"
elif [ -d "$HOME/.gemini/gsd-local-patches" ]; then
  PATCHES_DIR="$HOME/.gemini/gsd-local-patches"
else
  PATCHES_DIR="$HOME/.claude/gsd-local-patches"
fi
# Local install fallback — check all runtime directories
if [ ! -d "$PATCHES_DIR" ]; then
  for dir in .config/opencode .opencode .gemini .claude; do
    if [ -d "./$dir/gsd-local-patches" ]; then
      PATCHES_DIR="./$dir/gsd-local-patches"
      break
    fi
  done
fi
```

パッチディレクトリから `backup-meta.json` を読み取る。

**パッチが見つからない場合：**
```
ローカルパッチが見つかりません。再適用するものはありません。

ローカルパッチは、GSD のワークフロー、コマンド、またはエージェントファイルを
変更した後に /gsd:update を実行すると自動的に保存される。
```
終了。

## ステップ 2: パッチの概要を表示

```
## 再適用するローカルパッチ

**バックアップ元：** v{from_version}
**現在のバージョン：** {read VERSION file}
**変更ファイル数：** {count}

| # | ファイル | ステータス |
|---|----------|------------|
| 1 | {file_path} | 保留中 |
| 2 | {file_path} | 保留中 |
```

## ステップ 3: 各ファイルをマージ

`backup-meta.json` 内の各ファイルについて：

1. **バックアップされたバージョンを読み取る**（`gsd-local-patches/` からのユーザーの変更済みコピー）
2. **新しくインストールされたバージョンを読み取る**（アップデート後の現在のファイル）
3. **比較してマージ：**

   - 新しいファイルがバックアップされたファイルと同一の場合：スキップ（変更は上流に取り込み済み）
   - 新しいファイルが異なる場合：ユーザーの変更を特定し、新しいバージョンに適用する

   **マージ戦略：**
   - 両方のバージョンを完全に読み取る
   - ユーザーが追加または変更したセクションを特定する（パス置換からの差分だけでなく、追加を探す）
   - ユーザーの追加・変更を新しいバージョンに適用する
   - ユーザーが変更したセクションが上流でも変更されていた場合：コンフリクトとしてフラグを立て、両方のバージョンを表示し、どちらを残すかユーザーに確認する

4. **マージ結果を書き込む**（インストール先の場所へ）
5. **ステータスを報告：**
   - `Merged` — ユーザーの変更がクリーンに適用されました
  - `Skipped` — 変更は既に上流に含まれている
   - `Conflict` — ユーザーが解決を選択しました

## ステップ 4: マニフェストの更新

再適用後、将来のアップデートでこれらをユーザーの変更として正しく検出できるようにファイルマニフェストを再生成する：

```bash
# The manifest will be regenerated on next /gsd:update
# For now, just note which files were modified
```

## ステップ 5: クリーンアップオプション

ユーザーに確認する：
- 「パッチバックアップを参照用に保持しますか？」 → `gsd-local-patches/` を保持
- 「パッチバックアップをクリーンアップしますか？」 → `gsd-local-patches/` ディレクトリを削除

## ステップ 6: レポート

```
## パッチ再適用完了

| # | ファイル | ステータス |
|---|----------|------------|
| 1 | {file_path} | ✓ マージ済み |
| 2 | {file_path} | ○ スキップ（上流に取り込み済み） |
| 3 | {file_path} | ⚠ コンフリクト解決済み |

{count} ファイルが更新されました。ローカルの変更が再び有効になりました。
```

</process>

<success_criteria>
- [ ] バックアップされたすべてのパッチが処理された
- [ ] ユーザーの変更が新しいバージョンにマージされた
- [ ] コンフリクトがユーザーの入力により解決された
- [ ] 各ファイルのステータスが報告された
</success_criteria>
