# フェーズ引数の解析

フェーズを操作するコマンドのフェーズ引数を解析・正規化します。

## 抽出

`$ARGUMENTS`から:
- フェーズ番号を抽出（最初の数値引数）
- フラグを抽出（`--`プレフィックス付き）
- 残りのテキストは説明（insert/addコマンド用）

## gsd-toolsの使用

`find-phase`コマンドは正規化と検証を一度に処理します:

```bash
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" find-phase "${PHASE}")
```

以下のJSONを返します:
- `found`: true/false
- `directory`: フェーズディレクトリのフルパス
- `phase_number`: 正規化された番号（例: "06", "06.1"）
- `phase_name`: 名前部分（例: "foundation"）
- `plans`: PLAN.mdファイルの配列
- `summaries`: SUMMARY.mdファイルの配列

## 手動正規化（レガシー）

整数フェーズを2桁にゼロパディングします。小数サフィックスは保持します。

```bash
# フェーズ番号の正規化
if [[ "$PHASE" =~ ^[0-9]+$ ]]; then
  # 整数: 8 → 08
  PHASE=$(printf "%02d" "$PHASE")
elif [[ "$PHASE" =~ ^([0-9]+)\.([0-9]+)$ ]]; then
  # 小数: 2.1 → 02.1
  PHASE=$(printf "%02d.%s" "${BASH_REMATCH[1]}" "${BASH_REMATCH[2]}")
fi
```

## 検証

`roadmap get-phase`を使用してフェーズの存在を検証します:

```bash
PHASE_CHECK=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
if [ "$(printf '%s\n' "$PHASE_CHECK" | jq -r '.found')" = "false" ]; then
  echo "ERROR: Phase ${PHASE} not found in roadmap"
  exit 1
fi
```

## ディレクトリ検索

ディレクトリ検索には`find-phase`を使用します:

```bash
PHASE_DIR=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" find-phase "${PHASE}" --raw)
```
