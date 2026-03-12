<trigger>
このワークフローは以下の場合に使用します：
- 既存プロジェクトで新しいセッションを開始する時
- ユーザーが「続き」「次は何」「どこまでやった」「再開」と言った時
- .planning/が既に存在する状態でのプランニング操作
- ユーザーがプロジェクトから離れていた後に戻った時
</trigger>

<purpose>
プロジェクトの完全なコンテキストを即座に復元し、「どこまでやった？」にすぐに完全な回答ができるようにします。
</purpose>

<required_reading>
@~/.claude/get-shit-done/references/continuation-format.md
</required_reading>

<process>

<step name="initialize">
1回の呼び出しですべてのコンテキストを読み込みます：

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init resume)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

JSONを解析: `state_exists`、`roadmap_exists`、`project_exists`、`planning_exists`、`has_interrupted_agent`、`interrupted_agent_id`、`commit_docs`。

**`state_exists`がtrueの場合:** load_stateに進む
**`state_exists`がfalseだが`roadmap_exists`または`project_exists`がtrueの場合:** STATE.mdの再構築を提案
**`planning_exists`がfalseの場合:** 新規プロジェクト - /gsd:new-projectにルーティング
</step>

<step name="load_state">

STATE.mdを読み込んで解析し、次にPROJECT.mdを読み込みます：

```bash
cat .planning/STATE.md
cat .planning/PROJECT.md
```

**STATE.mdから抽出：**

- **プロジェクト参照**: コアバリューと現在のフォーカス
- **現在の位置**: フェーズ X / Y、プラン A / B、ステータス
- **進捗**: ビジュアルプログレスバー
- **最近の決定**: 現在の作業に影響する重要な決定
- **保留中のTodo**: セッション中にキャプチャされたアイデア
- **ブロッカー/懸念事項**: 持ち越された問題
- **セッション継続性**: 中断した場所、再開ファイル

**PROJECT.mdから抽出：**

- **これは何か**: 現在の正確な説明
- **要件**: 検証済み、アクティブ、スコープ外
- **重要な決定**: 結果を含む完全な決定ログ
- **制約**: 実装のハードリミット

</step>

<step name="check_incomplete_work">
注意が必要な未完了作業を探します：

```bash
# Check for continue-here files (mid-plan resumption)
ls .planning/phases/*/.continue-here*.md 2>/dev/null

# Check for plans without summaries (incomplete execution)
for plan in .planning/phases/*/*-PLAN.md; do
  summary="${plan/PLAN/SUMMARY}"
  [ ! -f "$summary" ] && echo "Incomplete: $plan"
done 2>/dev/null

# Check for interrupted agents (use has_interrupted_agent and interrupted_agent_id from init)
if [ "$has_interrupted_agent" = "true" ]; then
  echo "Interrupted agent: $interrupted_agent_id"
fi
```

**.continue-hereファイルが存在する場合：**

- プラン途中の再開ポイントです
- 具体的な再開コンテキストのためにファイルを読み込みます
- フラグ: 「プラン途中のチェックポイントを発見」

**SUMMARYのないPLANが存在する場合：**

- 実行は開始されたが完了していません
- フラグ: 「未完了のプラン実行を発見」

**中断されたエージェントが見つかった場合：**

- サブエージェントが生成されたがセッション終了前に完了しませんでした
- タスクの詳細についてagent-history.jsonを読み込みます
- フラグ: 「中断されたエージェントを発見」
  </step>

<step name="present_status">
ユーザーに完全なプロジェクトステータスを提示します：

```
╔══════════════════════════════════════════════════════════════╗
║  プロジェクトステータス                                         ║
╠══════════════════════════════════════════════════════════════╣
║  構築中: [PROJECT.md "What This Is"からの一行説明]              ║
║                                                               ║
║  フェーズ: [X] / [Y] - [フェーズ名]                            ║
║  プラン:  [A] / [B] - [ステータス]                             ║
║  進捗: [██████░░░░] XX%                                      ║
║                                                               ║
║  最終活動: [日付] - [実施内容]                                  ║
╚══════════════════════════════════════════════════════════════╝

[未完了作業が見つかった場合:]
⚠️  未完了の作業を検出:
    - [.continue-hereファイルまたは未完了のプラン]

[中断されたエージェントが見つかった場合:]
⚠️  中断されたエージェントを検出:
    エージェントID: [id]
    タスク: [agent-history.jsonからのタスク説明]
    中断日時: [timestamp]

    再開方法: Taskツール（エージェントIDでresumeパラメータ）

[保留中のtodoがある場合:]
📋 [N] 件の保留中todo — /gsd:check-todos で確認

[ブロッカーがある場合:]
⚠️  持ち越しの懸念事項:
    - [ブロッカー 1]
    - [ブロッカー 2]

[アラインメントが✓でない場合:]
⚠️  ブリーフアラインメント: [ステータス] - [評価]
```

</step>

<step name="determine_next_action">
プロジェクトの状態に基づいて、最も論理的な次のアクションを決定します：

**中断されたエージェントが存在する場合：**
→ 主要: 中断されたエージェントを再開（Taskツールでresumeパラメータ）
→ オプション: 最初からやり直す（エージェント作業を放棄）

**.continue-hereファイルが存在する場合：**
→ 主要: チェックポイントから再開
→ オプション: 現在のプランを最初からやり直す

**未完了のプラン（SUMMARYのないPLAN）がある場合：**
→ 主要: 未完了のプランを完了する
→ オプション: 放棄して次に進む

**フェーズが進行中で、すべてのプランが完了している場合：**
→ 主要: 次のフェーズに移行
→ オプション: 完了した作業を確認

**フェーズがプラン準備完了の場合：**
→ このフェーズにCONTEXT.mdが存在するか確認：

- CONTEXT.mdが存在しない場合：
  → 主要: フェーズのビジョンを議論（ユーザーがどう動作させたいか）
  → 副次: 直接プランニング（コンテキスト収集をスキップ）
- CONTEXT.mdが存在する場合：
  → 主要: フェーズを計画する
  → オプション: ロードマップを確認

**フェーズが実行準備完了の場合：**
→ 主要: 次のプランを実行
→ オプション: 先にプランを確認
</step>

<step name="offer_options">
プロジェクトの状態に基づいたコンテキストに応じたオプションを提示します：

```
何をしますか？

[状態に基づく主要アクション - 例:]
1. 中断されたエージェントを再開 [中断されたエージェントが見つかった場合]
   または
1. フェーズを実行 (/gsd:execute-phase {phase})
   または
1. フェーズ3のコンテキストを議論 (/gsd:discuss-phase 3) [CONTEXT.mdが存在しない場合]
   または
1. フェーズ3を計画 (/gsd:plan-phase 3) [CONTEXT.mdが存在するか、議論オプションが辞退された場合]

[副次オプション:]
2. 現在のフェーズステータスを確認
3. 保留中のtodoを確認（[N] 件保留中）
4. ブリーフアラインメントを確認
5. その他
```

**注意:** フェーズプランニングを提案する際は、まずCONTEXT.mdの存在を確認してください：

```bash
ls .planning/phases/XX-name/*-CONTEXT.md 2>/dev/null
```

存在しない場合は、プランの前にdiscuss-phaseを提案します。存在する場合は、直接プランを提案します。

ユーザーの選択を待ちます。
</step>

<step name="route_to_workflow">
ユーザーの選択に基づいて、適切なワークフローにルーティングします：

- **プランを実行** → クリア後にユーザーが実行するコマンドを表示：
  ```
  ---

  ## ▶ Next Up

  **{phase}-{plan}: [Plan Name]** — [objective from PLAN.md]

  `/gsd:execute-phase {phase}`

  <sub>`/clear` first → fresh context window</sub>

  ---
  ```
- **フェーズを計画** → クリア後にユーザーが実行するコマンドを表示：
  ```
  ---

  ## ▶ Next Up

  **Phase [N]: [Name]** — [Goal from ROADMAP.md]

  `/gsd:plan-phase [phase-number]`

  <sub>`/clear` first → fresh context window</sub>

  ---

  **Also available:**
  - `/gsd:discuss-phase [N]` — gather context first
  - `/gsd:research-phase [N]` — investigate unknowns

  ---
  ```
- **移行** → ./transition.md
- **todoを確認** → .planning/todos/pending/を読み込み、サマリーを提示
- **アラインメントを確認** → PROJECT.mdを読み込み、現在の状態と比較
- **その他** → 何が必要か確認
</step>

<step name="update_session">
ルーティングされたワークフローに進む前に、セッション継続性を更新します：

STATE.mdを更新：

```markdown
## Session Continuity

Last session: [now]
Stopped at: Session resumed, proceeding to [action]
Resume file: [updated if applicable]
```

セッションが予期せず終了した場合に、次の再開時に状態を把握できるようにします。
</step>

</process>

<reconstruction>
STATE.mdがないが他のアーティファクトが存在する場合：

「STATE.mdがありません。アーティファクトから再構築中...」

1. PROJECT.mdを読み込み → 「What This Is」とCore Valueを抽出
2. ROADMAP.mdを読み込み → フェーズを特定し、現在の位置を見つける
3. \*-SUMMARY.mdファイルをスキャン → 決定事項、懸念事項を抽出
4. .planning/todos/pending/内の保留中todoをカウント
5. .continue-hereファイルを確認 → セッション継続性

STATE.mdを再構築して書き込み、その後通常通り進めます。

以下のケースに対応します：

- STATE.md導入前のプロジェクト
- ファイルが誤って削除された
- 完全な.planning/状態なしでリポジトリをクローン
  </reconstruction>

<quick_resume>
ユーザーが「続き」や「行こう」と言った場合：
- 状態をサイレントに読み込み
- 主要アクションを決定
- オプションを提示せずに即座に実行

「[状態]から続行中... [アクション]」
</quick_resume>

<success_criteria>
再開は以下の条件を満たした時に完了です：

- [ ] STATE.mdが読み込まれた（または再構築された）
- [ ] 未完了の作業が検出されフラグが立てられた
- [ ] 明確なステータスがユーザーに提示された
- [ ] コンテキストに応じた次のアクションが提案された
- [ ] ユーザーがプロジェクトの現状を正確に把握した
- [ ] セッション継続性が更新された
      </success_criteria>
