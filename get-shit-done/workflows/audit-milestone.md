<purpose>
フェーズ検証を集約し、クロスフェーズ統合をチェックし、要件カバレッジを評価することで、マイルストーンが完了定義を達成したかを検証する。既存のVERIFICATION.mdファイルを読み取り（フェーズはexecute-phase中に既に検証済み）、技術的負債と延期されたギャップを集約し、クロスフェーズの接続のために統合チェッカーを生成する。
</purpose>

<required_reading>
開始前に、呼び出しプロンプトのexecution_contextで参照されるすべてのファイルを読み取ること。
</required_reading>

<process>

## 0. マイルストーンコンテキストの初期化

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init milestone-op)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから抽出: `milestone_version`, `milestone_name`, `phase_count`, `completed_phases`, `commit_docs`。

統合チェッカーモデルの解決:
```bash
integration_checker_model=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-integration-checker --raw)
```

## 1. マイルストーンスコープの決定

```bash
# マイルストーン内のフェーズを取得（数値順にソート、小数対応）
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phases list
```

- 引数からバージョンをパースするか、ROADMAP.mdから現在のバージョンを検出
- スコープ内のすべてのフェーズディレクトリを特定
- ROADMAP.mdからマイルストーンの完了定義を抽出
- REQUIREMENTS.mdからこのマイルストーンにマッピングされた要件を抽出

## 2. すべてのフェーズ検証の読み取り

各フェーズディレクトリについて、VERIFICATION.mdを読み取る:

```bash
# 各フェーズについて、find-phaseを使用してディレクトリを解決（アーカイブ済みフェーズ対応）
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" find-phase 01 --raw)
# JSONからディレクトリを抽出し、そのディレクトリからVERIFICATION.mdを読み取る
# ROADMAP.mdの各フェーズ番号について繰り返す
```

各VERIFICATION.mdから抽出:
- **ステータス:** passed | gaps_found
- **重大なギャップ:** （ある場合 — ブロッカー）
- **非重大なギャップ:** 技術的負債、延期された項目、警告
- **検出されたアンチパターン:** TODO、スタブ、プレースホルダー
- **要件カバレッジ:** どの要件が満たされた/ブロックされた

フェーズにVERIFICATION.mdがない場合、「未検証フェーズ」としてフラグ — これはブロッカー。

## 3. 統合チェッカーの生成

収集されたフェーズコンテキストで:

REQUIREMENTS.mdのトレーサビリティテーブルから`MILESTONE_REQ_IDS`を抽出 — このマイルストーンのフェーズに割り当てられたすべてのREQ-ID。

```
Task(
  prompt="クロスフェーズ統合とE2Eフローを確認。

Phases: {phase_dirs}
Phase exports: {SUMMARYsから}
API routes: {作成されたルート}

マイルストーン要件:
{MILESTONE_REQ_IDS — 各REQ-IDと説明と割り当てフェーズをリスト}

各統合の発見を該当する要件IDにマッピングすること必須。

クロスフェーズの接続とE2Eユーザーフローを検証。",
  subagent_type="gsd-integration-checker",
  model="{integration_checker_model}"
)
```

## 4. 結果の収集

以下を統合:
- フェーズレベルのギャップと技術的負債（ステップ2から）
- 統合チェッカーのレポート（接続ギャップ、壊れたフロー）

## 5. 要件カバレッジの確認（3ソースクロスリファレンス）

各要件について、3つの独立したソースをクロスリファレンスすること必須:

### 5a. REQUIREMENTS.mdトレーサビリティテーブルのパース

トレーサビリティテーブルからマイルストーンのフェーズにマッピングされたすべてのREQ-IDを抽出:
- 要件ID、説明、割り当てフェーズ、現在のステータス、チェック状態（`[x]` vs `[ ]`）

### 5b. フェーズVERIFICATION.md要件テーブルのパース

各フェーズのVERIFICATION.mdから、展開された要件テーブルを抽出:
- 要件 | ソースプラン | 説明 | ステータス | エビデンス
- 各エントリをREQ-IDに紐づけ

### 5c. SUMMARY.mdフロントマターのクロスチェック抽出

各フェーズのSUMMARY.mdから、YAMLフロントマターの`requirements-completed`を抽出:
```bash
for summary in .planning/phases/*-*/*-SUMMARY.md; do
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" summary-extract "$summary" --fields requirements_completed | jq -r '.requirements_completed'
done
```

### 5d. ステータス判定マトリックス

各REQ-IDについて、3つのソースすべてを使用してステータスを判定:

| VERIFICATION.mdステータス | SUMMARYフロントマター | REQUIREMENTS.md | → 最終ステータス |
|------------------------|---------------------|-----------------|----------------|
| passed                 | 記載あり             | `[x]`           | **satisfied**  |
| passed                 | 記載あり             | `[ ]`           | **satisfied** (チェックボックスを更新) |
| passed                 | 記載なし             | いずれか          | **partial** (手動で検証) |
| gaps_found             | いずれか             | いずれか          | **unsatisfied** |
| missing                | 記載あり             | いずれか          | **partial** (検証ギャップ) |
| missing                | 記載なし             | いずれか          | **unsatisfied** |

### 5e. FAILゲートとオーファン検出

**必須:** `unsatisfied`の要件があれば、マイルストーン監査のステータスを`gaps_found`に強制すること。

**オーファン検出:** REQUIREMENTS.mdのトレーサビリティテーブルに存在するがすべてのフェーズVERIFICATION.mdファイルに存在しない要件は、オーファンとしてフラグすること。オーファンの要件は`unsatisfied`として扱う — 割り当てられたがどのフェーズでも検証されなかった。

## 5.5. Nyquistコンプライアンスのディスカバリー

`workflow.nyquist_validation`が明示的に`false`の場合はスキップ（未設定 = 有効）。

```bash
NYQUIST_CONFIG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config get workflow.nyquist_validation --raw 2>/dev/null)
```

`false`の場合: 完全にスキップ。

各フェーズディレクトリについて、`*-VALIDATION.md`を確認。存在する場合、フロントマターをパース（`nyquist_compliant`, `wave_0_complete`）。

フェーズごとに分類:

| ステータス | 条件 |
|--------|-----------|
| COMPLIANT | `nyquist_compliant: true`かつすべてのタスクがグリーン |
| PARTIAL | VALIDATION.mdが存在、`nyquist_compliant: false`またはred/pending |
| MISSING | VALIDATION.mdなし |

監査YAMLに追加: `nyquist: { compliant_phases, partial_phases, missing_phases, overall }`

ディスカバリーのみ — `/gsd:validate-phase`を自動呼び出ししない。

## 6. v{version}-MILESTONE-AUDIT.mdに集約

`.planning/v{version}-v{version}-MILESTONE-AUDIT.md`を以下で作成:

```yaml
---
milestone: {version}
audited: {timestamp}
status: passed | gaps_found | tech_debt
scores:
  requirements: N/M
  phases: N/M
  integration: N/M
  flows: N/M
gaps:  # 重大なブロッカー
  requirements:
    - id: "{REQ-ID}"
      status: "unsatisfied | partial | orphaned"
      phase: "{割り当てフェーズ}"
      claimed_by_plans: ["{この要件を参照する計画ファイル}"]
      completed_by_plans: ["{SUMMARYで完了とマークされた計画ファイル}"]
      verification_status: "passed | gaps_found | missing | orphaned"
      evidence: "{具体的なエビデンスまたはその欠如}"
  integration: [...]
  flows: [...]
tech_debt:  # 非重大、延期
  - phase: 01-auth
    items:
      - "TODO: add rate limiting"
      - "Warning: no password strength validation"
  - phase: 03-dashboard
    items:
      - "Deferred: mobile responsive layout"
---
```

要件、フェーズ、統合、技術的負債のテーブルを含む完全なマークダウンレポートを付加。

**ステータス値:**
- `passed` — すべての要件が満たされ、重大なギャップなし、技術的負債は最小限
- `gaps_found` — 重大なブロッカーが存在
- `tech_debt` — ブロッカーはないが、蓄積された延期項目のレビューが必要

## 7. 結果の提示

ステータスに基づいてルーティング（`<offer_next>`を参照）。

</process>

<offer_next>
このマークダウンを直接出力する（コードブロックとしてではない）。ステータスに基づいてルーティング:

---

**passedの場合:**

## ✓ マイルストーン {version} — 監査合格

**スコア:** {N}/{M}の要件が満たされた
**レポート:** .planning/v{version}-MILESTONE-AUDIT.md

すべての要件がカバーされました。クロスフェーズ統合が検証されました。E2Eフローが完了しました。

───────────────────────────────────────────────────────────────

## ▶ Next Up

**マイルストーンを完了** — アーカイブしてタグ付け

/gsd:complete-milestone {version}

<sub>/clear first → 新しいコンテキストウィンドウ</sub>

───────────────────────────────────────────────────────────────

---

**gaps_foundの場合:**

## ⚠ マイルストーン {version} — ギャップが見つかりました

**スコア:** {N}/{M}の要件が満たされた
**レポート:** .planning/v{version}-MILESTONE-AUDIT.md

### 未達成の要件

{未達成の各要件について:}
- **{REQ-ID}: {description}** (フェーズ {X})
  - {理由}

### クロスフェーズの問題

{各統合ギャップについて:}
- **{from} → {to}:** {問題}

### 壊れたフロー

{各フローギャップについて:}
- **{フロー名}:** {ステップ}で中断

### Nyquistカバレッジ

| フェーズ | VALIDATION.md | コンプライアント | アクション |
|-------|---------------|-----------|--------|
| {phase} | exists/missing | true/false/partial | `/gsd:validate-phase {N}` |

検証が必要なフェーズ: フラグが付いた各フェーズに対して`/gsd:validate-phase {N}`を実行。

───────────────────────────────────────────────────────────────

## ▶ Next Up

**ギャップクロージャーを計画** — マイルストーン完了のためのフェーズを作成

/gsd:plan-milestone-gaps

<sub>/clear first → 新しいコンテキストウィンドウ</sub>

───────────────────────────────────────────────────────────────

**その他のオプション:**
- cat .planning/v{version}-MILESTONE-AUDIT.md — 完全なレポートを確認
- /gsd:complete-milestone {version} — そのまま続行（技術的負債を受け入れ）

───────────────────────────────────────────────────────────────

---

**tech_debtの場合（ブロッカーはないが蓄積された負債あり）:**

## ⚡ マイルストーン {version} — 技術的負債レビュー

**スコア:** {N}/{M}の要件が満たされた
**レポート:** .planning/v{version}-MILESTONE-AUDIT.md

すべての要件が満たされた。重大なブロッカーなし。蓄積された技術的負債のレビューが必要。

### フェーズ別の技術的負債

{負債のある各フェーズについて:}
**フェーズ {X}: {name}**
- {item 1}
- {item 2}

### 合計: {M}フェーズにわたる{N}項目

───────────────────────────────────────────────────────────────

## ▶ オプション

**A. マイルストーンを完了** — 負債を受け入れ、バックログで追跡

/gsd:complete-milestone {version}

**B. クリーンアップフェーズを計画** — 完了前に負債を対処

/gsd:plan-milestone-gaps

<sub>/clear first → 新しいコンテキストウィンドウ</sub>

───────────────────────────────────────────────────────────────
</offer_next>

<success_criteria>
- [ ] マイルストーンスコープを特定
- [ ] すべてのフェーズVERIFICATION.mdファイルを読み取り
- [ ] 各フェーズのSUMMARY.md `requirements-completed`フロントマターを抽出
- [ ] マイルストーンのすべてのREQ-IDについてREQUIREMENTS.mdトレーサビリティテーブルをパース
- [ ] 3ソースクロスリファレンスを完了（VERIFICATION + SUMMARY + トレーサビリティ）
- [ ] オーファン要件を検出（トレーサビリティにあるがすべてのVERIFICATIONに存在しない）
- [ ] 技術的負債と延期されたギャップを集約
- [ ] マイルストーン要件IDで統合チェッカーを生成
- [ ] 構造化された要件ギャップオブジェクトを含むv{version}-MILESTONE-AUDIT.mdを作成
- [ ] FAILゲートを適用 — 未達成の要件があればgaps_foundステータスを強制
- [ ] すべてのマイルストーンフェーズのNyquistコンプライアンスをスキャン（有効な場合）
- [ ] VALIDATION.mdが欠落しているフェーズをvalidate-phaseの提案付きでフラグ
- [ ] アクション可能な次のステップ付きで結果を提示
</success_criteria>
