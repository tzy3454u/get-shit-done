<purpose>
npm経由でGSDの更新を確認し、インストール済みバージョンと最新バージョン間の変更ログを表示し、ユーザーの確認を得て、キャッシュクリアを含むクリーンインストールを実行します。
</purpose>

<required_reading>
開始前に、呼び出しプロンプトのexecution_contextで参照されているすべてのファイルを読み込んでください。
</required_reading>

<process>

<step name="get_installed_version">
両方の場所を確認しインストールの整合性を検証して、GSDがローカルまたはグローバルにインストールされているかを検出します：

```bash
# Check local first (takes priority only if valid)
# Detect runtime config directory (supports Claude, OpenCode, Gemini)
LOCAL_VERSION_FILE="" LOCAL_MARKER_FILE="" LOCAL_DIR=""
for dir in .claude .config/opencode .opencode .gemini; do
  if [ -f "./$dir/get-shit-done/VERSION" ]; then
    LOCAL_VERSION_FILE="./$dir/get-shit-done/VERSION"
    LOCAL_MARKER_FILE="./$dir/get-shit-done/workflows/update.md"
    LOCAL_DIR="$(cd "./$dir" 2>/dev/null && pwd)"
    break
  fi
done
GLOBAL_VERSION_FILE="" GLOBAL_MARKER_FILE="" GLOBAL_DIR=""
for dir in .claude .config/opencode .opencode .gemini; do
  if [ -f "$HOME/$dir/get-shit-done/VERSION" ]; then
    GLOBAL_VERSION_FILE="$HOME/$dir/get-shit-done/VERSION"
    GLOBAL_MARKER_FILE="$HOME/$dir/get-shit-done/workflows/update.md"
    GLOBAL_DIR="$(cd "$HOME/$dir" 2>/dev/null && pwd)"
    break
  fi
done

# Only treat as LOCAL if the resolved paths differ (prevents misdetection when CWD=$HOME)
IS_LOCAL=false
if [ -n "$LOCAL_VERSION_FILE" ] && [ -f "$LOCAL_VERSION_FILE" ] && [ -f "$LOCAL_MARKER_FILE" ] && grep -Eq '^[0-9]+\.[0-9]+\.[0-9]+' "$LOCAL_VERSION_FILE"; then
  if [ -z "$GLOBAL_DIR" ] || [ "$LOCAL_DIR" != "$GLOBAL_DIR" ]; then
    IS_LOCAL=true
  fi
fi

if [ "$IS_LOCAL" = true ]; then
  cat "$LOCAL_VERSION_FILE"
  echo "LOCAL"
elif [ -n "$GLOBAL_VERSION_FILE" ] && [ -f "$GLOBAL_VERSION_FILE" ] && [ -f "$GLOBAL_MARKER_FILE" ] && grep -Eq '^[0-9]+\.[0-9]+\.[0-9]+' "$GLOBAL_VERSION_FILE"; then
  cat "$GLOBAL_VERSION_FILE"
  echo "GLOBAL"
else
  echo "UNKNOWN"
fi
```

出力を解析します：
- 最後の行が「LOCAL」の場合：ローカルインストールが有効、インストール済みバージョンは最初の行、`--local`を使用
- 最後の行が「GLOBAL」の場合：ローカルが存在しない/無効、グローバルインストールが有効、インストール済みバージョンは最初の行、`--global`を使用
- 「UNKNOWN」の場合：インストールステップに進む（バージョン0.0.0として扱う）

**VERSIONファイルが存在しない場合：**
```
## GSD アップデート

**インストール済みバージョン:** 不明

インストールにバージョン追跡が含まれていません。

新規インストールを実行中...
```

インストールステップに進みます（比較用にバージョン0.0.0として扱う）。
</step>

<step name="check_latest_version">
npmで最新バージョンを確認します：

```bash
npm view get-shit-done-cc version 2>/dev/null
```

**npmチェックが失敗した場合：**
```
更新を確認できませんでした（オフラインまたはnpmが利用不可）。

手動で更新するには: `npx get-shit-done-cc --global`
```

終了します。
</step>

<step name="compare_versions">
インストール済みと最新を比較します：

**インストール済み == 最新の場合：**
```
## GSD アップデート

**インストール済み:** X.Y.Z
**最新:** X.Y.Z

既に最新バージョンです。
```

終了します。

**インストール済み > 最新の場合：**
```
## GSD アップデート

**インストール済み:** X.Y.Z
**最新:** A.B.C

最新リリースより先のバージョンです（開発バージョン？）。
```

終了します。
</step>

<step name="show_changes_and_confirm">
**更新が利用可能な場合**、更新前に新機能を取得して表示します：

1. GitHubのraw URLから変更ログを取得
2. インストール済みバージョンと最新バージョン間のエントリを抽出
3. プレビューを表示して確認を求めます：

```
## GSD アップデートが利用可能

**インストール済み:** 1.5.10
**最新:** 1.5.15

### 新機能
────────────────────────────────────────────────────────────

## [1.5.15] - 2026-01-20

### Added
- Feature X

## [1.5.14] - 2026-01-18

### Fixed
- Bug fix Y

────────────────────────────────────────────────────────────

⚠️  **注意:** インストーラーはGSDフォルダのクリーンインストールを実行します:
- `commands/gsd/`は消去され置き換えられます
- `get-shit-done/`は消去され置き換えられます
- `agents/gsd-*`ファイルは置き換えられます

（パスはインストール場所に対して相対的です: グローバルは`~/.claude/`、ローカルは`./.claude/`）

他の場所のカスタムファイルは保持されます:
- `commands/gsd/`にないカスタムコマンド ✓
- `gsd-`プレフィックスのないカスタムエージェント ✓
- カスタムフック ✓
- CLAUDE.mdファイル ✓

GSDファイルを直接変更している場合、自動的に`gsd-local-patches/`にバックアップされ、更新後に`/gsd:reapply-patches`で再適用できます。
```

AskUserQuestionを使用：
- 質問: 「更新を続行しますか？」
- オプション:
  - 「はい、今すぐ更新」
  - 「いいえ、キャンセル」

**ユーザーがキャンセルした場合:** 終了します。
</step>

<step name="run_update">
ステップ1で検出されたインストールタイプを使用して更新を実行します：

**ローカルインストールの場合：**
```bash
npx -y get-shit-done-cc@latest --local
```

**グローバルインストール（または不明）の場合：**
```bash
npx -y get-shit-done-cc@latest --global
```

出力をキャプチャします。インストールが失敗した場合、エラーを表示して終了します。

ステータスラインのインジケータが消えるよう更新キャッシュをクリアします：

```bash
# Clear update cache across all runtime directories
for dir in .claude .config/opencode .opencode .gemini; do
  rm -f "./$dir/cache/gsd-update-check.json"
  rm -f "$HOME/$dir/cache/gsd-update-check.json"
done
```

SessionStartフック（`gsd-check-update.js`）は検出されたランタイムのキャッシュディレクトリに書き込むため、古い更新インジケータを防ぐためにすべてのパスをクリアする必要があります。
</step>

<step name="display_result">
完了メッセージをフォーマットします（変更ログは確認ステップで既に表示済み）：

```
╔═══════════════════════════════════════════════════════════╗
║  GSD 更新完了: v1.5.10 → v1.5.15                          ║
╚═══════════════════════════════════════════════════════════╝

⚠️  新しいコマンドを反映するにはClaude Codeを再起動してください。

[完全な変更ログを見る](https://github.com/glittercowboy/get-shit-done/blob/main/CHANGELOG.md)
```
</step>


<step name="check_local_patches">
更新完了後、インストーラーがローカルで変更されたファイルを検出しバックアップしたかを確認します：

設定ディレクトリ内のgsd-local-patches/backup-meta.jsonを確認します。

**パッチが見つかった場合：**

```
更新前にローカルパッチがバックアップされました。
/gsd:reapply-patches を実行して、変更を新しいバージョンにマージしてください。
```

**パッチがない場合：** 通常通り続行します。
</step>
</process>

<success_criteria>
- [ ] インストール済みバージョンが正しく読み取られた
- [ ] npmで最新バージョンが確認された
- [ ] 既に最新の場合は更新がスキップされた
- [ ] 更新前に変更ログが取得・表示された
- [ ] クリーンインストールの警告が表示された
- [ ] ユーザーの確認が得られた
- [ ] 更新が正常に実行された
- [ ] 再起動リマインダーが表示された
</success_criteria>
