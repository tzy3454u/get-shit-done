<purpose>
並列デバッグエージェントをオーケストレーションしてUATギャップを調査し、根本原因を特定する。

UATがギャップを発見した後、ギャップごとに1つのデバッグエージェントを生成する。各エージェントはUATからの症状を事前入力された状態で自律的に調査する。根本原因を収集し、UAT.mdのギャップを診断で更新し、実際の診断に基づいてplan-phase --gapsにハンドオフする。

オーケストレーターは軽量に保つ：ギャップをパース、エージェントを生成、結果を収集、UATを更新。
</purpose>

<paths>
DEBUG_DIR=.planning/debug

デバッグファイルは`.planning/debug/`パスを使用する（先頭にドットを持つ隠しディレクトリ）。
</paths>

<core_principle>
**修正を計画する前に診断する。**

UATはWHATが壊れているか（症状）を教えてくれる。デバッグエージェントがWHY（根本原因）を見つける。plan-phase --gapsは推測ではなく実際の原因に基づいて的確な修正を作成する。

診断なし：「コメントが更新されない」→ 修正を推測 → 間違っている可能性
診断あり：「コメントが更新されない」→「useEffectの依存関係が不足」→ 正確な修正
</core_principle>

<process>

<step name="parse_gaps">
**UAT.mdからギャップを抽出：**

「Gaps」セクション（YAML形式）を読む：
```yaml
- truth: "Comment appears immediately after submission"
  status: failed
  reason: "User reported: works but doesn't show until I refresh the page"
  severity: major
  test: 2
  artifacts: []
  missing: []
```

各ギャップについて、完全なコンテキストを取得するために「Tests」セクションから対応するテストも読む。

ギャップリストを構築：
```
gaps = [
  {truth: "Comment appears immediately...", severity: "major", test_num: 2, reason: "..."},
  {truth: "Reply button positioned correctly...", severity: "minor", test_num: 5, reason: "..."},
  ...
]
```
</step>

<step name="report_plan">
**診断計画をユーザーに報告：**

```
## {N}個のギャップを診断中

根本原因を調査するために並列デバッグエージェントを生成中：

| ギャップ（Truth） | 重大度 |
|-------------|----------|
| コメント送信後即座に表示 | major |
| 返信ボタンが正しく配置 | minor |
| 削除でコメントが削除される | blocker |

各エージェントが行うこと：
1. 症状を事前入力したDEBUG-{slug}.mdを作成
2. 自律的に調査（コードを読む、仮説を立てる、テスト）
3. 根本原因を返す

これは並列実行されます - すべてのギャップが同時に調査されます。
```
</step>

<step name="spawn_agents">
**デバッグエージェントを並列で生成：**

各ギャップについて、debug-subagent-promptテンプレートを埋めて生成：

```
Task(
  prompt=filled_debug_subagent_prompt + "\n\n<files_to_read>\n- {phase_dir}/{phase_num}-UAT.md\n- .planning/STATE.md\n</files_to_read>",
  subagent_type="gsd-debugger",
  description="Debug: {truth_short}"
)
```

**すべてのエージェントを1つのメッセージで生成**（並列実行）。

テンプレートのプレースホルダー：
- `{truth}`: 失敗した期待される動作
- `{expected}`: UATテストから
- `{actual}`: reasonフィールドからのユーザー説明の引用
- `{errors}`: UATからのエラーメッセージ（または「None reported」）
- `{reproduction}`: 「Test {test_num} in UAT」
- `{timeline}`: 「Discovered during UAT」
- `{goal}`: `find_root_cause_only`（UATフロー - plan-phase --gapsが修正を処理）
- `{slug}`: truthから生成
</step>

<step name="collect_results">
**エージェントから根本原因を収集：**

各エージェントが以下を返す：
```
## ROOT CAUSE FOUND

**Debug Session:** ${DEBUG_DIR}/{slug}.md

**Root Cause:** {証拠付きの具体的な原因}

**Evidence Summary:**
- {主要な発見1}
- {主要な発見2}
- {主要な発見3}

**Files Involved:**
- {file1}: {何が問題か}
- {file2}: {関連する問題}

**Suggested Fix Direction:** {plan-phase --gaps向けの簡単なヒント}
```

各返却値をパースして抽出：
- root_cause: 診断された原因
- files: 関連するファイル
- debug_path: デバッグセッションファイルへのパス
- suggested_fix: ギャップクロージャープラン向けのヒント

エージェントが`## INVESTIGATION INCONCLUSIVE`を返した場合：
- root_cause: 「調査が不確定 - 手動レビューが必要」
- どの問題が手動対応を必要とするか記載
- エージェントの返却値から残りの可能性を含める
</step>

<step name="update_uat">
**UAT.mdのギャップを診断で更新：**

Gapsセクションの各ギャップについて、artifactsとmissingフィールドを追加：

```yaml
- truth: "Comment appears immediately after submission"
  status: failed
  reason: "User reported: works but doesn't show until I refresh the page"
  severity: major
  test: 2
  root_cause: "useEffect in CommentList.tsx missing commentCount dependency"
  artifacts:
    - path: "src/components/CommentList.tsx"
      issue: "useEffect missing dependency"
  missing:
    - "Add commentCount to useEffect dependency array"
    - "Trigger re-render when new comment added"
  debug_session: .planning/debug/comment-not-refreshing.md
```

フロントマターのステータスを「diagnosed」に更新。

更新されたUAT.mdをコミット：
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs({phase_num}): add root causes from diagnosis" --files ".planning/phases/XX-name/{phase_num}-UAT.md"
```
</step>

<step name="report_results">
**診断結果を報告してハンドオフ：**

表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DIAGNOSIS COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| ギャップ（Truth） | 根本原因 | ファイル |
|-------------|------------|-------|
| コメント即座に表示 | useEffectの依存関係不足 | CommentList.tsx |
| 返信ボタンの位置 | CSSのflex orderが不正 | ReplyButton.tsx |
| 削除でコメント削除 | APIに認証ヘッダーが不足 | api/comments.ts |

デバッグセッション: ${DEBUG_DIR}/

修正プランの作成に進みます...
```

自動計画のためにverify-workオーケストレーターに返す。
手動の次ステップを提案しないこと - verify-workが残りを処理する。
</step>

</process>

<context_efficiency>
エージェントはUATから事前入力された症状で開始する（症状収集なし）。
エージェントは診断のみ — plan-phase --gapsが修正を処理する（修正適用なし）。
</context_efficiency>

<failure_handling>
**エージェントが根本原因を見つけられない場合：**
- ギャップを「手動レビューが必要」とマーク
- 他のギャップで続行
- 不完全な診断を報告

**エージェントがタイムアウトした場合：**
- DEBUG-{slug}.mdの部分的な進捗を確認
- /gsd:debugで再開可能

**すべてのエージェントが失敗した場合：**
- システム的な問題（権限、git等）
- 手動調査のために報告
- 根本原因なしでplan-phase --gapsにフォールバック（精度は低下）
</failure_handling>

<success_criteria>
- [ ] UAT.mdからギャップがパースされた
- [ ] デバッグエージェントが並列で生成された
- [ ] すべてのエージェントから根本原因が収集された
- [ ] UAT.mdのギャップがartifactsとmissingで更新された
- [ ] デバッグセッションが${DEBUG_DIR}/に保存された
- [ ] 自動計画のためにverify-workにハンドオフ
</success_criteria>
</output>