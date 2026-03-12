<purpose>
ゴール逆算分析によるフェーズ目標の達成を検証する。タスクが完了しただけでなく、コードベースがフェーズの約束を実現しているかを確認する。

execute-phase.mdから生成された検証サブエージェントによって実行される。
</purpose>

<core_principle>
**タスク完了 ≠ ゴール達成**

タスク「チャットコンポーネントの作成」は、コンポーネントがプレースホルダーの時点で完了とマークできる。タスクは完了した — しかしゴール「動作するチャットインターフェース」は達成されていない。

ゴール逆算検証:
1. ゴールが達成されたためにTRUEでなければならないことは何か？
2. それらの真実が成り立つために存在しなければならないものは何か？
3. それらの成果物が機能するために接続されていなければならないものは何か？

次に、各レベルを実際のコードベースに対して検証する。
</core_principle>

<required_reading>
@~/.claude/get-shit-done/references/verification-patterns.md
@~/.claude/get-shit-done/templates/verification-report.md
</required_reading>

<process>

<step name="load_context" priority="first">
フェーズ操作コンテキストを読み込む:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから抽出: `phase_dir`, `phase_number`, `phase_name`, `has_plans`, `plan_count`。

次にフェーズの詳細を読み込み、計画/サマリーを一覧:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${phase_number}"
grep -E "^| ${phase_number}" .planning/REQUIREMENTS.md 2>/dev/null
ls "$phase_dir"/*-SUMMARY.md "$phase_dir"/*-PLAN.md 2>/dev/null
```

ROADMAP.mdから**フェーズのゴール**（検証する成果、タスクではない）を抽出し、REQUIREMENTS.mdが存在する場合は**要件**を抽出。
</step>

<step name="establish_must_haves">
**オプションA: PLANフロントマターのmust-haves**

gsd-toolsを使用して各PLANからmust_havesを抽出:

```bash
for plan in "$PHASE_DIR"/*-PLAN.md; do
  MUST_HAVES=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" frontmatter get "$plan" --field must_haves)
  echo "=== $plan ===" && echo "$MUST_HAVES"
done
```

JSONを返す: `{ truths: [...], artifacts: [...], key_links: [...] }`

フェーズレベルの検証のために、すべての計画のmust_havesを集約する。

**オプションB: ROADMAP.mdの成功基準を使用**

フロントマターにmust_havesがない場合（MUST_HAVESがエラーまたは空を返す）、成功基準を確認:

```bash
PHASE_DATA=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${phase_number}" --raw)
```

JSON出力から`success_criteria`配列をパース。空でない場合:
1. 各成功基準を直接**真実**として使用（観察可能でテスト可能な動作として記述済み）
2. **成果物**を導出（各真実に対する具体的なファイルパス）
3. **キーリンク**を導出（スタブが隠れている重要な接続）
4. 進む前にmust-havesを文書化

ROADMAP.mdの成功基準は契約 — 両方が存在する場合、PLANレベルのmust_havesより優先。

**オプションC: フェーズゴールから導出（フォールバック）**

フロントマターにmust_havesがなく、かつROADMAPに成功基準がない場合:
1. ROADMAP.mdのゴールを記述
2. **真実**を導出（3-7の観察可能な動作、各々テスト可能）
3. **成果物**を導出（各真実に対する具体的なファイルパス）
4. **キーリンク**を導出（スタブが隠れている重要な接続）
5. 進む前に導出されたmust-havesを文書化
</step>

<step name="verify_truths">
各観察可能な真実について、コードベースがそれを可能にしているかを判断する。

**ステータス:** ✓ VERIFIED（すべてのサポート成果物が合格）| ✗ FAILED（成果物が欠落/スタブ/未接続）| ? UNCERTAIN（人間の確認が必要）

各真実について: サポート成果物を特定 → 成果物のステータスを確認 → 接続を確認 → 真実のステータスを判定。

**例:** 真実「ユーザーは既存のメッセージを見ることができる」は、Chat.tsx（レンダリング）、/api/chat GET（提供）、Messageモデル（スキーマ）に依存。Chat.tsxがスタブであるか、APIがハードコードされた[]を返す場合 → FAILED。すべてが存在し、実質的で、接続されている場合 → VERIFIED。
</step>

<step name="verify_artifacts">
各PLANのmust_havesに対する成果物検証にgsd-toolsを使用:

```bash
for plan in "$PHASE_DIR"/*-PLAN.md; do
  ARTIFACT_RESULT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" verify artifacts "$plan")
  echo "=== $plan ===" && echo "$ARTIFACT_RESULT"
done
```

JSON結果をパース: `{ all_passed, passed, total, artifacts: [{path, exists, issues, passed}] }`

**結果からの成果物ステータス:**
- `exists=false` → MISSING
- `issues`が空でない → STUB（"Only N lines"や"Missing pattern"のissuesを確認）
- `passed=true` → VERIFIED（レベル1-2が合格）

**レベル3 — 接続確認（レベル1-2に合格した成果物の手動チェック）:**
```bash
grep -r "import.*$artifact_name" src/ --include="*.ts" --include="*.tsx"  # インポート済み
grep -r "$artifact_name" src/ --include="*.ts" --include="*.tsx" | grep -v "import"  # 使用済み
```
WIRED = インポートされ、かつ使用されている。ORPHANED = 存在するがインポート/使用されていない。

| 存在 | 実質的 | 接続 | ステータス |
|--------|-------------|-------|--------|
| ✓ | ✓ | ✓ | ✓ VERIFIED |
| ✓ | ✓ | ✗ | ⚠️ ORPHANED |
| ✓ | ✗ | - | ✗ STUB |
| ✗ | - | - | ✗ MISSING |
</step>

<step name="verify_wiring">
各PLANのmust_havesに対するキーリンク検証にgsd-toolsを使用:

```bash
for plan in "$PHASE_DIR"/*-PLAN.md; do
  LINKS_RESULT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" verify key-links "$plan")
  echo "=== $plan ===" && echo "$LINKS_RESULT"
done
```

JSON結果をパース: `{ all_verified, verified, total, links: [{from, to, via, verified, detail}] }`

**結果からのリンクステータス:**
- `verified=true` → WIRED
- `verified=false`で"not found" → NOT_WIRED
- `verified=false`で"Pattern not found" → PARTIAL

**フォールバックパターン（must_havesにkey_linksがない場合）:**

| パターン | チェック | ステータス |
|---------|-------|--------|
| コンポーネント → API | APIパスへのfetch/axios呼び出し、レスポンスが使用されている（await/.then/setState） | WIRED / PARTIAL（呼び出しはあるが未使用のレスポンス）/ NOT_WIRED |
| API → データベース | モデルに対するPrisma/DBクエリ、結果がres.json()で返される | WIRED / PARTIAL（クエリはあるが返されていない）/ NOT_WIRED |
| フォーム → ハンドラー | 実際の実装を持つonSubmit（fetch/axios/mutate/dispatch）、console.log/空ではない | WIRED / STUB（ログのみ/空）/ NOT_WIRED |
| ステート → レンダー | JSXにuseState変数が現れる（`{stateVar}`または`{stateVar.property}`） | WIRED / NOT_WIRED |

各キーリンクのステータスとエビデンスを記録。
</step>

<step name="verify_requirements">
REQUIREMENTS.mdが存在する場合:
```bash
grep -E "Phase ${PHASE_NUM}" .planning/REQUIREMENTS.md 2>/dev/null
```

各要件について: 説明をパース → サポートする真実/成果物を特定 → ステータス: ✓ SATISFIED / ✗ BLOCKED / ? NEEDS HUMAN。
</step>

<step name="scan_antipatterns">
SUMMARY.mdからこのフェーズで変更されたファイルを抽出し、各々をスキャン:

| パターン | 検索 | 深刻度 |
|---------|--------|----------|
| TODO/FIXME/XXX/HACK | `grep -n -E "TODO\|FIXME\|XXX\|HACK"` | ⚠️ 警告 |
| プレースホルダーコンテンツ | `grep -n -iE "placeholder\|coming soon\|will be here"` | 🛑 ブロッカー |
| 空のリターン | `grep -n -E "return null\|return \{\}\|return \[\]\|=> \{\}"` | ⚠️ 警告 |
| ログのみの関数 | console.logのみを含む関数 | ⚠️ 警告 |

分類: 🛑 ブロッカー（ゴールを妨げる）| ⚠️ 警告（不完全）| ℹ️ 情報（注目すべき点）。
</step>

<step name="identify_human_verification">
**常に人間の確認が必要:** 視覚的な外観、ユーザーフローの完了、リアルタイムの動作（WebSocket/SSE）、外部サービス連携、パフォーマンスの体感、エラーメッセージの明確さ。

**不確実な場合に人間の確認が必要:** grepで追跡できない複雑な接続、動的な状態依存の動作、エッジケース。

各項目のフォーマット: テスト名 → 何をするか → 期待される結果 → プログラムで検証できない理由。
</step>

<step name="determine_status">
**passed:** すべての真実がVERIFIED、すべての成果物がレベル1-3に合格、すべてのキーリンクがWIRED、ブロッカーのアンチパターンなし。

**gaps_found:** いずれかの真実がFAILED、成果物がMISSING/STUB、キーリンクがNOT_WIRED、またはブロッカーが検出。

**human_needed:** すべての自動チェックは合格したが、人間の検証項目が残っている。

**スコア:** `verified_truths / total_truths`
</step>

<step name="generate_fix_plans">
gaps_foundの場合:

1. **関連するギャップをクラスター化:** APIスタブ + コンポーネント未接続 → "フロントエンドとバックエンドの接続"。複数の欠落 → "コア実装の完了"。接続のみ → "既存コンポーネントの接続"。

2. **クラスターごとに計画を生成:** 目的、2-3のタスク（各々のファイル/アクション/検証）、再検証ステップ。焦点を絞る: 計画ごとに単一の関心事。

3. **依存関係で順序付け:** 欠落を修正 → スタブを修正 → 接続を修正 → 検証。
</step>

<step name="create_report">
```bash
REPORT_PATH="$PHASE_DIR/${PHASE_NUM}-VERIFICATION.md"
```

テンプレートのセクションを記入: フロントマター（フェーズ/タイムスタンプ/ステータス/スコア）、ゴール達成、成果物テーブル、接続テーブル、要件カバレッジ、アンチパターン、人間の検証、ギャップサマリー、修正計画（gaps_foundの場合）、メタデータ。

完全なテンプレートは~/.claude/get-shit-done/templates/verification-report.mdを参照。
</step>

<step name="return_to_orchestrator">
ステータス（`passed` | `gaps_found` | `human_needed`）、スコア（N/M must-haves）、レポートパスを返す。

gaps_foundの場合: ギャップリスト + 推奨される修正計画名を記載。
human_neededの場合: 人間のテストが必要な項目を記載。

オーケストレーターのルーティング: `passed` → update_roadmap | `gaps_found` → 修正の作成/実行、再検証 | `human_needed` → ユーザーに提示。
</step>

</process>

<success_criteria>
- [ ] must-havesを確立（フロントマターから、または導出）
- [ ] すべての真実をステータスとエビデンスで検証
- [ ] すべての成果物を3つのレベルすべてで確認
- [ ] すべてのキーリンクを検証
- [ ] 要件カバレッジを評価（該当する場合）
- [ ] アンチパターンをスキャンし分類
- [ ] 人間の検証項目を特定
- [ ] 全体的なステータスを判定
- [ ] 修正計画を生成（gaps_foundの場合）
- [ ] 完全なレポートを含むVERIFICATION.mdを作成
- [ ] オーケストレーターに結果を返す
</success_criteria>

