<purpose>
完了したフェーズのNyquist検証ギャップを監査する。不足しているテストを生成する。VALIDATION.mdを更新する。
</purpose>

<required_reading>
@~/.claude/get-shit-done/references/ui-brand.md
</required_reading>

<process>

## 0. 初期化

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

パース: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`。

```bash
AUDITOR_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-nyquist-auditor --raw)
NYQUIST_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config get workflow.nyquist_validation --raw)
```

`NYQUIST_CFG`が`false`の場合: "Nyquist検証は無効です。/gsd:settingsで有効にしてください。"と表示して終了。

バナーを表示: `GSD > VALIDATE PHASE {N}: {name}`

## 1. 入力状態の検出

```bash
VALIDATION_FILE=$(ls "${PHASE_DIR}"/*-VALIDATION.md 2>/dev/null | head -1)
SUMMARY_FILES=$(ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null)
```

- **状態A** (`VALIDATION_FILE`が空でない): 既存を監査
- **状態B** (`VALIDATION_FILE`が空、`SUMMARY_FILES`が空でない): 成果物から再構築
- **状態C** (`SUMMARY_FILES`が空): 終了 — "フェーズ{N}は未実行です。先に/gsd:execute-phase {N}を実行してください。"

## 2. ディスカバリー

### 2a. フェーズ成果物の読み取り

すべてのPLANファイルとSUMMARYファイルを読み取る。抽出: タスクリスト、要件ID、変更された主要ファイル、検証ブロック。

### 2b. 要件-タスクマップの構築

タスクごと: `{ task_id, plan_id, wave, requirement_ids, has_automated_command }`

### 2c. テストインフラの検出

状態A: 既存のVALIDATION.mdのテストインフラテーブルからパース。
状態B: ファイルシステムスキャン:

```bash
find . -name "pytest.ini" -o -name "jest.config.*" -o -name "vitest.config.*" -o -name "pyproject.toml" 2>/dev/null | head -10
find . \( -name "*.test.*" -o -name "*.spec.*" -o -name "test_*" \) -not -path "*/node_modules/*" 2>/dev/null | head -40
```

### 2d. クロスリファレンス

各要件を既存テストとファイル名、インポート、テスト説明で照合する。記録: 要件 → test_file → status。

## 3. ギャップ分析

各要件を分類:

| ステータス | 基準 |
|--------|----------|
| COVERED | テストが存在し、動作を対象とし、グリーンで実行される |
| PARTIAL | テストが存在するが、失敗または不完全 |
| MISSING | テストが見つからない |

構築: `{ task_id, requirement, gap_type, suggested_test_path, suggested_command }`

ギャップなし → ステップ6にスキップし、`nyquist_compliant: true`を設定。

## 4. ギャップ計画の提示

ギャップテーブルとオプションでAskUserQuestionを呼び出す:
1. "すべてのギャップを修正" → ステップ5
2. "スキップ — 手動のみとしてマーク" → Manual-Onlyに追加、ステップ6
3. "キャンセル" → 終了

## 5. gsd-nyquist-auditorの生成

```
Task(
  prompt="Read ~/.claude/agents/gsd-nyquist-auditor.md for instructions.\n\n" +
    "<files_to_read>{PLAN, SUMMARY, impl files, VALIDATION.md}</files_to_read>" +
    "<gaps>{gap list}</gaps>" +
    "<test_infrastructure>{framework, config, commands}</test_infrastructure>" +
    "<constraints>実装ファイルを変更しないこと。デバッグ反復は最大3回。実装のバグはエスカレーション。</constraints>",
  subagent_type="gsd-nyquist-auditor",
  model="{AUDITOR_MODEL}",
  description="Fill validation gaps for Phase {N}"
)
```

戻り値の処理:
- `## GAPS FILLED` → テスト + マップ更新を記録、ステップ6
- `## PARTIAL` → 解決済みを記録、エスカレーション分をmanual-onlyに移動、ステップ6
- `## ESCALATE` → すべてをmanual-onlyに移動、ステップ6

## 6. VALIDATION.mdの生成/更新

**状態B (作成):**
1. `~/.claude/get-shit-done/templates/VALIDATION.md`からテンプレートを読み取る
2. 記入: フロントマター、テストインフラ、タスク別マップ、Manual-Only、サインオフ
3. `${PHASE_DIR}/${PADDED_PHASE}-VALIDATION.md`に書き込む

**状態A (更新):**
1. タスク別マップのステータスを更新、エスカレーション分をManual-Onlyに追加、フロントマターを更新
2. 監査証跡を追記:

```markdown
## Validation Audit {date}
| 指標 | 件数 |
|--------|-------|
| 検出されたギャップ | {N} |
| 解決済み | {M} |
| エスカレーション | {K} |
```

## 7. コミット

```bash
git add {test_files}
git commit -m "test(phase-${PHASE}): add Nyquist validation tests"

node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit-docs "docs(phase-${PHASE}): add/update validation strategy"
```

## 8. 結果 + ルーティング

**準拠:**
```
GSD > PHASE {N} IS NYQUIST-COMPLIANT
すべての要件に自動検証があります。
▶ Next: /gsd:audit-milestone
```

**部分的:**
```
GSD > PHASE {N} VALIDATED (PARTIAL)
{M}件が自動化済み、{K}件が手動のみ。
▶ Retry: /gsd:validate-phase {N}
```

`/clear`リマインダーを表示。

</process>

<success_criteria>
- [ ] Nyquist設定を確認（無効の場合は終了）
- [ ] 入力状態を検出（A/B/C）
- [ ] 状態Cは正常に終了
- [ ] PLAN/SUMMARYファイルを読み取り、要件マップを構築
- [ ] テストインフラを検出
- [ ] ギャップを分類（COVERED/PARTIAL/MISSING）
- [ ] ギャップテーブルによるユーザーゲート
- [ ] 完全なコンテキストでオーディターを生成
- [ ] 3つの戻り値フォーマットすべてを処理
- [ ] VALIDATION.mdを作成または更新
- [ ] テストファイルを個別にコミット
- [ ] ルーティング付きの結果を提示
</success_criteria>
</output>
