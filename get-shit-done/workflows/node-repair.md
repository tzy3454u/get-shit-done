<purpose>
タスク検証失敗時の自律的リペアオペレーター。タスクが完了基準を満たさない場合にexecute-planから呼び出される。ユーザーにエスカレーションする前に構造化された修正を提案・試行する。
</purpose>

<inputs>
- FAILED_TASK: プランからのタスク番号、名前、完了基準
- ERROR: 検証が出力した内容 — 実際の結果 vs 期待値
- PLAN_CONTEXT: 隣接タスクとフェーズゴール（制約認識のため）
- REPAIR_BUDGET: 残りの最大リペア試行回数（デフォルト: 2）
</inputs>

<repair_directive>
失敗を分析し、リペア戦略を1つだけ選択する：

**RETRY** — アプローチは正しかったが実行に失敗した。具体的な調整を行って再試行する。
- 使用場面: コマンドエラー、不足する依存関係、誤ったパス、環境問題、一時的な障害
- 出力: `RETRY: [再試行前に行う具体的な調整]`

**DECOMPOSE** — タスクが粗すぎる。より小さな検証可能なサブステップに分割する。
- 使用場面: 完了基準が複数の懸念をカバーしている、実装ギャップが構造的
- 出力: `DECOMPOSE: [サブタスク1] | [サブタスク2] | ...`（最大3サブタスク）
- 各サブタスクは単一の検証可能な結果を持つ必要がある

**PRUNE** — 現在の制約下でタスクは実行不可能。正当な理由を付けてスキップする。
- 使用場面: 前提条件が不足しており、ここでは修正不可能、スコープ外、以前の決定と矛盾
- 出力: `PRUNE: [一文の正当化理由]`

**ESCALATE** — リペア予算を使い切った、またはアーキテクチャ上の決定（ルール4）が必要。
- 使用場面: RETRYが異なるアプローチで複数回失敗した、または修正に構造的変更が必要
- 出力: `ESCALATE: [試行内容] | [必要な決定]`
</repair_directive>

<process>

<step name="diagnose">
エラーと完了基準を注意深く読む。以下を確認：
1. 一時的/環境的な問題か？ → RETRY
2. タスクが検証可能なほど広すぎるか？ → DECOMPOSE
3. 前提条件が本当に不足しており、スコープ内で修正不可能か？ → PRUNE
4. このタスクで既にRETRYが試行されたか？REPAIR_BUDGETを確認。0の場合 → ESCALATE
</step>

<step name="execute_retry">
RETRYの場合：
1. ディレクティブに記載された具体的な調整を適用
2. タスクの実装を再実行
3. 検証を再実行
4. 合格した場合 → 通常通り続行、ログ記録 `[Node Repair - RETRY] Task [X]: [行った調整]`
5. 再度失敗した場合 → REPAIR_BUDGETをデクリメントし、更新されたコンテキストでnode-repairを再呼び出し
</step>

<step name="execute_decompose">
DECOMPOSEの場合：
1. 失敗したタスクをサブタスクにインラインで置換（ディスク上のPLAN.mdは変更しない）
2. サブタスクを順次実行、各々に独自の検証を実施
3. すべてのサブタスクが合格した場合 → 元のタスクを成功として扱い、ログ記録 `[Node Repair - DECOMPOSE] Task [X] → [N] sub-tasks`
4. サブタスクが失敗した場合 → そのサブタスクに対してnode-repairを再呼び出し（REPAIR_BUDGETはサブタスクごとに適用）
</step>

<step name="execute_prune">
PRUNEの場合：
1. 正当化理由を付けてタスクをスキップ済みとしてマーク
2. SUMMARY「Issues Encountered」にログ記録: `[Node Repair - PRUNE] Task [X]: [正当化理由]`
3. 次のタスクに進む
</step>

<step name="execute_escalate">
ESCALATEの場合：
1. 完全なリペア履歴を付けてverification_failure_gate経由でユーザーに通知
2. 提示：試行内容（各RETRY/DECOMPOSE試行）、ブロッカーの内容、利用可能なオプション
3. 続行前にユーザーの指示を待つ
</step>

</process>

<logging>
すべてのリペアアクションはSUMMARY.mdの「## Deviations from Plan」に記載する必要がある：

| タイプ | フォーマット |
|------|--------|
| RETRY成功 | `[Node Repair - RETRY] Task X: [調整] — 解決済み` |
| RETRY失敗 → ESCALATE | `[Node Repair - RETRY] Task X: [N]回試行で予算切れ — ユーザーにエスカレーション` |
| DECOMPOSE | `[Node Repair - DECOMPOSE] Task Xを[N]サブタスクに分割 — すべて合格` |
| PRUNE | `[Node Repair - PRUNE] Task Xをスキップ: [正当化理由]` |
</logging>

<constraints>
- REPAIR_BUDGETのデフォルトはタスクごとに2。config.jsonの`workflow.node_repair_budget`で設定可能。
- ディスク上のPLAN.mdは変更しないこと — DECOMPOSEのサブタスクはメモリ内のみ。
- DECOMPOSEのサブタスクは元のタスクより具体的でなければならず、同義語の書き換えではない。
- config.jsonの`workflow.node_repair`が`false`の場合、verification_failure_gateに直接スキップする（ユーザーは元の動作を維持）。
</constraints>
