<purpose>
プロジェクトの進捗を確認し、最近の作業と今後の内容を要約し、次のアクションにインテリジェントにルーティングする — 既存のプランの実行か、次のプランの作成のいずれか。作業を続行する前に状況認識を提供する。
</purpose>

<required_reading>
開始前に、呼び出し元プロンプトのexecution_contextで参照されているすべてのファイルを読み込むこと。
</required_reading>

<process>

<step name="init_context">
**進捗コンテキストを読み込む（パスのみ）：**

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init progress)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから抽出：`project_exists`, `roadmap_exists`, `state_exists`, `phases`, `current_phase`, `next_phase`, `milestone_version`, `completed_count`, `phase_count`, `paused_at`, `state_path`, `roadmap_path`, `project_path`, `config_path`。

`project_exists`がfalseの場合（`.planning/`ディレクトリがない）：

```
計画構造が見つかりません。

/gsd:new-projectを実行して新しいプロジェクトを開始すること。
```

終了。

STATE.mdが見つからない場合：`/gsd:new-project`を提案。

**ROADMAP.mdが見つからないがPROJECT.mdが存在する場合：**

マイルストーンが完了してアーカイブされたことを意味する。**Route F**（マイルストーン間）に移動。

ROADMAP.mdとPROJECT.mdの両方が見つからない場合：`/gsd:new-project`を提案。
</step>

<step name="load">
**gsd-toolsからの構造化抽出を使用：**

完全なファイルを読む代わりに、レポートに必要なデータのみを取得するためにターゲットツールを使用：
- `ROADMAP=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)`
- `STATE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state-snapshot)`

これによりオーケストレーターのコンテキスト使用量が最小化される。
</step>

<step name="analyze_roadmap">
**包括的なロードマップ分析を取得（手動パースを置き換え）：**

```bash
ROADMAP=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)
```

構造化JSONで以下が返される：
- ディスクステータス付きの全フェーズ（complete/partial/planned/empty/no_directory）
- フェーズごとの目標と依存関係
- フェーズごとのプランとサマリー数
- 集計統計：総プラン、サマリー、進捗率
- 現在のフェーズと次のフェーズの特定

ROADMAP.mdを手動で読み込み/パースする代わりにこれを使用する。
</step>

<step name="recent">
**最近の作業コンテキストを収集：**

- 最新の2-3個のSUMMARY.mdファイルを見つける
- 効率的なパースに`summary-extract`を使用：
  ```bash
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" summary-extract <path> --fields one_liner
  ```
- これにより「何に取り組んでいたか」が表示される
  </step>

<step name="position">
**initコンテキストとロードマップ分析から現在の位置をパース：**

- `$ROADMAP`から`current_phase`と`next_phase`を使用
- 作業が一時停止されている場合は`paused_at`を記録（`$STATE`から）
- 保留中のtodoを数える：`init todos`または`list-todos`を使用
- アクティブなデバッグセッションを確認：`ls .planning/debug/*.md 2>/dev/null | grep -v resolved | wc -l`
  </step>

<step name="report">
**gsd-toolsからプログレスバーを生成し、リッチなステータスレポートを提示：**

```bash
# フォーマット済みプログレスバーを取得
PROGRESS_BAR=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" progress bar --raw)
```

提示：

```
# [Project Name]

**進捗:** {PROGRESS_BAR}
**プロファイル:** [quality/balanced/budget]

## 最近の作業
- [Phase X, Plan Y]: [達成されたこと - summary-extractからの1行]
- [Phase X, Plan Z]: [達成されたこと - summary-extractからの1行]

## 現在の位置
Phase [N] of [total]: [phase-name]
Plan [M] of [phase-total]: [status]
CONTEXT: [✓ has_contextの場合 | - そうでない場合]

## 主要な決定事項
- [$STATE.decisions[]から抽出]
- [例：state-snapshotからjq -r '.decisions[].decision']

## ブロッカー/懸念事項
- [$STATE.blockers[]から抽出]
- [例：state-snapshotからjq -r '.blockers[].text']

## 保留中のTodo
- [count]個の保留 — /gsd:check-todosで確認

## アクティブなデバッグセッション
- [count]個のアクティブ — /gsd:debugで続行
(このセクションはcount > 0の場合のみ表示)

## 次にやること
[ロードマップ分析からの次のフェーズ/プラン目標]
```

</step>

<step name="route">
**検証済みカウントに基づいて次のアクションを決定する。**

**ステップ1：現在のフェーズのプラン、サマリー、問題を数える**

現在のフェーズディレクトリのファイルをリスト：

```bash
ls -1 .planning/phases/[current-phase-dir]/*-PLAN.md 2>/dev/null | wc -l
ls -1 .planning/phases/[current-phase-dir]/*-SUMMARY.md 2>/dev/null | wc -l
ls -1 .planning/phases/[current-phase-dir]/*-UAT.md 2>/dev/null | wc -l
```

状態：「このフェーズには{X}個のプラン、{Y}個のサマリーがある。」

**ステップ1.5：未対処のUATギャップを確認**

ステータスが「diagnosed」のUAT.mdファイル（修正が必要なギャップあり）を確認。

```bash
# ギャップ付きのdiagnosedされたUATを確認
grep -l "status: diagnosed" .planning/phases/[current-phase-dir]/*-UAT.md 2>/dev/null
```

追跡：
- `uat_with_gaps`: ステータスが「diagnosed」のUAT.mdファイル（ギャップの修正が必要）

**ステップ2：カウントに基づいてルーティング**

| 条件 | 意味 | アクション |
|-----------|---------|--------|
| uat_with_gaps > 0 | UATギャップの修正プランが必要 | **Route E**へ |
| summaries < plans | 未実行のプランが存在 | **Route A**へ |
| summaries = plans かつ plans > 0 | フェーズ完了 | ステップ3へ |
| plans = 0 | フェーズ未計画 | **Route B**へ |

---

**Route A: 未実行のプランが存在**

対応するSUMMARY.mdがない最初のPLAN.mdを見つける。
その`<objective>`セクションを読む。

```
---

## ▶ Next Up

**{phase}-{plan}: [Plan Name]** — [PLAN.mdからの目標サマリー]

`/gsd:execute-phase {phase}`

<sub>`/clear` first → 新しいコンテキストウィンドウ</sub>

---
```

---

**Route B: フェーズの計画が必要**

`{phase_num}-CONTEXT.md`がフェーズディレクトリに存在するか確認。

**CONTEXT.mdが存在する場合：**

```
---

## ▶ Next Up

**Phase {N}: {Name}** — {ROADMAP.mdからの目標}
<sub>✓ コンテキスト収集済み、計画準備完了</sub>

`/gsd:plan-phase {phase-number}`

<sub>`/clear` first → 新しいコンテキストウィンドウ</sub>

---
```

**CONTEXT.mdが存在しない場合：**

```
---

## ▶ Next Up

**Phase {N}: {Name}** — {ROADMAP.mdからの目標}

`/gsd:discuss-phase {phase}` — コンテキストを収集しアプローチを明確にする

<sub>`/clear` first → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション：**
- `/gsd:plan-phase {phase}` — ディスカッションをスキップして直接計画
- `/gsd:list-phase-assumptions {phase}` — Claudeの前提を確認

---
```

---

**Route E: UATギャップの修正プランが必要**

ギャップ付きのUAT.mdが存在（diagnosed済みの問題）。ユーザーは修正を計画する必要がある。

```
---

## ⚠ UATギャップ検出

**{phase_num}-UAT.md**に修正が必要な{N}個のギャップがある。

`/gsd:plan-phase {phase} --gaps`

<sub>`/clear` first → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション：**
- `/gsd:execute-phase {phase}` — フェーズプランを実行
- `/gsd:verify-work {phase}` — さらにUATテストを実行

---
```

---

**ステップ3：マイルストーンステータスを確認（フェーズ完了時のみ）**

ROADMAP.mdを読み、以下を特定：
1. 現在のフェーズ番号
2. 現在のマイルストーンセクションのすべてのフェーズ番号

総フェーズ数を数え、最大フェーズ番号を特定。

状態：「現在のフェーズは{X}。マイルストーンには{N}フェーズあり（最大：{Y}）。」

**マイルストーンステータスに基づくルーティング：**

| 条件 | 意味 | アクション |
|-----------|---------|--------|
| 現在のフェーズ < 最大フェーズ | さらにフェーズが残っている | **Route C**へ |
| 現在のフェーズ = 最大フェーズ | マイルストーン完了 | **Route D**へ |

---

**Route C: フェーズ完了、さらにフェーズが残っている**

ROADMAP.mdを読んで次のフェーズの名前と目標を取得。

```
---

## ✓ Phase {Z} Complete

## ▶ Next Up

**Phase {Z+1}: {Name}** — {ROADMAP.mdからの目標}

`/gsd:discuss-phase {Z+1}` — コンテキストを収集しアプローチを明確にする

<sub>`/clear` first → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション：**
- `/gsd:plan-phase {Z+1}` — ディスカッションをスキップして直接計画
- `/gsd:verify-work {Z}` — 続行前にユーザー受け入れテスト

---
```

---

**Route D: マイルストーン完了**

```
---

## 🎉 マイルストーン完了

全{N}フェーズが完了しました！

## ▶ Next Up

**マイルストーンを完了** — アーカイブして次の準備

`/gsd:complete-milestone`

<sub>`/clear` first → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション：**
- `/gsd:verify-work` — マイルストーン完了前にユーザー受け入れテスト

---
```

---

**Route F: マイルストーン間（ROADMAP.mdがないがPROJECT.mdが存在）**

マイルストーンが完了してアーカイブされた。次のマイルストーンサイクルを開始する準備が完了。

MILESTONES.mdを読んで最後に完了したマイルストーンバージョンを見つける。

```
---

## ✓ Milestone v{X.Y} Complete

次のマイルストーンの計画準備が完了しました。

## ▶ Next Up

**次のマイルストーンを開始** — 質問 → リサーチ → 要件 → ロードマップ

`/gsd:new-milestone`

<sub>`/clear` first → 新しいコンテキストウィンドウ</sub>

---
```

</step>

<step name="edge_cases">
**エッジケースの処理：**

- フェーズ完了だが次のフェーズが未計画 → `/gsd:plan-phase [next]`を提案
- すべての作業完了 → マイルストーン完了を提案
- ブロッカーが存在 → 続行の提案前にハイライト
- ハンドオフファイルが存在 → 言及し、`/gsd:resume-work`を提案
  </step>

</process>

<success_criteria>

- [ ] リッチなコンテキストが提供された（最近の作業、決定、問題）
- [ ] ビジュアルな進捗付きで現在の位置が明確
- [ ] 次にやることが明確に説明されている
- [ ] スマートルーティング：プランが存在すれば/gsd:execute-phase、なければ/gsd:plan-phase
- [ ] アクション前にユーザーが確認
- [ ] 適切なgsdコマンドへのシームレスなハンドオフ
      </success_criteria>
