<purpose>
`.planning/` ディレクトリの整合性を検証し、対処可能な問題を報告する。不足ファイル、無効な設定、不整合な状態、孤立した計画を確認する。オプションで自動修復可能な問題を修復する。
</purpose>

<required_reading>
開始前に、呼び出し元プロンプトのexecution_contextで参照されているすべてのファイルを読み込むこと。
</required_reading>

<process>

<step name="parse_args">
**引数をパースする：**

コマンド引数に `--repair` フラグが存在するか確認する。

```
REPAIR_FLAG=""
if arguments contain "--repair"; then
  REPAIR_FLAG="--repair"
fi
```
</step>

<step name="run_health_check">
**ヘルスチェックの検証を実行する：**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" validate health $REPAIR_FLAG
```

JSON出力をパースする：
- `status`: "healthy" | "degraded" | "broken"
- `errors[]`: 重大な問題 (code, message, fix, repairable)
- `warnings[]`: 重大ではない問題
- `info[]`: 情報メモ
- `repairable_count`: 自動修復可能な問題の数
- `repairs_performed[]`: --repairが使用された場合に実行されたアクション
</step>

<step name="format_output">
**結果をフォーマットして表示する：**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ヘルスチェック
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ステータス: HEALTHY | DEGRADED | BROKEN
エラー: N | 警告: N | 情報: N
```

**修復が実行された場合：**
```
## 実行された修復

- ✓ config.json: デフォルト値で作成
- ✓ STATE.md: ロードマップから再生成
```

**エラーが存在する場合：**
```
## エラー

- [E001] config.json: 5行目でJSONパースエラー
  修正方法: /gsd:health --repair を実行してデフォルトにリセット

- [E002] PROJECT.md が見つかりません
  修正方法: /gsd:new-project を実行して作成
```

**警告が存在する場合：**
```
## 警告

- [W001] STATE.md がフェーズ5を参照していますが、フェーズ1-3のみ存在します
  修正方法: /gsd:health --repair を実行して再生成

- [W005] フェーズディレクトリ "1-setup" がNN-name形式に従っていません
  修正方法: パターンに合わせてリネーム（例：01-setup）
```

**情報が存在する場合：**
```
## 情報

- [I001] 02-implementation/02-01-PLAN.md に対応するSUMMARY.mdがありません
  備考: 進行中の可能性があります
```

**フッター（修復可能な問題があり--repairが使用されていない場合）：**
```
---
N 件の問題を自動修復できます。実行: /gsd:health --repair
```
</step>

<step name="offer_repair">
**修復可能な問題があり--repairが使用されていない場合：**

ユーザーに修復を実行するか確認する：

```
/gsd:health --repair を実行してN件の問題を自動的に修復しますか？
```

はいの場合、--repairフラグ付きで再実行し、結果を表示する。
</step>

<step name="verify_repairs">
**修復が実行された場合：**

--repairなしでヘルスチェックを再実行し、問題が解決されたことを確認する：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" validate health
```

最終ステータスを報告する。
</step>

</process>

<error_codes>

| Code | Severity | Description | Repairable |
|------|----------|-------------|------------|
| E001 | error | .planning/ ディレクトリが見つかりません | No |
| E002 | error | PROJECT.md が見つかりません | No |
| E003 | error | ROADMAP.md が見つかりません | No |
| E004 | error | STATE.md が見つかりません | Yes |
| E005 | error | config.json パースエラー | Yes |
| W001 | warning | PROJECT.md に必要なセクションがありません | No |
| W002 | warning | STATE.md が無効なフェーズを参照しています | Yes |
| W003 | warning | config.json が見つかりません | Yes |
| W004 | warning | config.json のフィールド値が無効です | No |
| W005 | warning | フェーズディレクトリの命名が不一致です | No |
| W006 | warning | ROADMAPにあるフェーズのディレクトリがありません | No |
| W007 | warning | ディスク上にあるフェーズがROADMAPにありません | No |
| W008 | warning | config.json: workflow.nyquist_validation が未設定（デフォルトは有効ですがエージェントがスキップする場合があります） | Yes |
| W009 | warning | フェーズのRESEARCH.mdにValidation Architectureがありますが、VALIDATION.mdがありません | No |
| I001 | info | SUMMARYのない計画（進行中の可能性） | No |

</error_codes>

<repair_actions>

| Action | Effect | Risk |
|--------|--------|------|
| createConfig | デフォルト値でconfig.jsonを作成 | なし |
| resetConfig | config.jsonを削除して再作成 | カスタム設定が失われる |
| regenerateState | ROADMAPの構造からSTATE.mdを作成 | セッション履歴が失われる |
| addNyquistKey | config.jsonにworkflow.nyquist_validation: trueを追加 | なし — 既存のデフォルトと一致 |

**修復不可（リスクが高い）：**
- PROJECT.md、ROADMAP.md のコンテンツ
- フェーズディレクトリのリネーム
- 孤立した計画のクリーンアップ

</repair_actions>
</output>
