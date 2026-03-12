# 小数フェーズの計算

緊急挿入のための次の小数フェーズ番号を計算します。

## gsd-toolsの使用

```bash
# フェーズ6の後の次の小数フェーズを取得
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase next-decimal 6
```

出力:
```json
{
  "found": true,
  "base_phase": "06",
  "next": "06.1",
  "existing": []
}
```

既存の小数がある場合:
```json
{
  "found": true,
  "base_phase": "06",
  "next": "06.3",
  "existing": ["06.1", "06.2"]
}
```

## 値の抽出

```bash
DECIMAL_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase next-decimal "${AFTER_PHASE}")
DECIMAL_PHASE=$(printf '%s\n' "$DECIMAL_INFO" | jq -r '.next')
BASE_PHASE=$(printf '%s\n' "$DECIMAL_INFO" | jq -r '.base_phase')
```

または--rawフラグ付き:
```bash
DECIMAL_PHASE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase next-decimal "${AFTER_PHASE}" --raw)
# 返されるのは: 06.1
```

## 例

| 既存フェーズ | 次のフェーズ |
|-----------------|------------|
| 06のみ | 06.1 |
| 06, 06.1 | 06.2 |
| 06, 06.1, 06.2 | 06.3 |
| 06, 06.1, 06.3（ギャップあり） | 06.4 |

## ディレクトリ命名

小数フェーズのディレクトリは完全な小数番号を使用します:

```bash
SLUG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" generate-slug "$DESCRIPTION" --raw)
PHASE_DIR=".planning/phases/${DECIMAL_PHASE}-${SLUG}"
mkdir -p "$PHASE_DIR"
```

例: `.planning/phases/06.1-fix-critical-auth-bug/`
