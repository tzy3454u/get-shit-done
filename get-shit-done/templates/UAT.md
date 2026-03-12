# UATテンプレート

`.planning/phases/XX-name/{phase_num}-UAT.md` 用テンプレート — 永続的なUATセッション追跡。

---

## ファイルテンプレート

```markdown
---
status: testing | complete | diagnosed
phase: XX-name
source: [テスト対象のSUMMARY.mdファイルのリスト]
started: [ISOタイムスタンプ]
updated: [ISOタイムスタンプ]
---

## Current Test
<!-- 各テストで上書き - 現在地を表示 -->

number: [N]
name: [テスト名]
expected: |
  [ユーザーが観察すべきこと]
awaiting: user response

## Tests

### 1. [テスト名]
expected: [観察可能な動作 - ユーザーが見るべきもの]
result: [pending]

### 2. [テスト名]
expected: [観察可能な動作]
result: pass

### 3. [テスト名]
expected: [観察可能な動作]
result: issue
reported: "[ユーザーの回答そのまま]"
severity: major

### 4. [テスト名]
expected: [観察可能な動作]
result: skipped
reason: [スキップ理由]

...

## Summary

total: [N]
passed: [N]
issues: [N]
pending: [N]
skipped: [N]

## Gaps

<!-- plan-phase --gaps消費用のYAML形式 -->
- truth: "[テストからの期待される動作]"
  status: failed
  reason: "User reported: [ユーザーの回答そのまま]"
  severity: blocker | major | minor | cosmetic
  test: [N]
  root_cause: ""     # 診断で記入
  artifacts: []      # 診断で記入
  missing: []        # 診断で記入
  debug_session: ""  # 診断で記入
```

---

<section_rules>

**フロントマター:**
- `status`: 上書き - "testing" または "complete"
- `phase`: 不変 - 作成時に設定
- `source`: 不変 - テスト対象のSUMMARYファイル
- `started`: 不変 - 作成時に設定
- `updated`: 上書き - 変更のたびに更新

**Current Test:**
- 各テスト遷移時に完全に上書き
- アクティブなテストと待機中の内容を表示
- 完了時: "[testing complete]"

**Tests:**
- 各テスト: ユーザーの回答時にresultフィールドを上書き
- `result` 値: [pending], pass, issue, skipped
- issueの場合: `reported`（そのまま）と`severity`（推定）を追加
- skippedの場合: 提供されていれば`reason`を追加

**Summary:**
- 各回答後にカウントを上書き
- 追跡: total, passed, issues, pending, skipped

**Gaps:**
- 問題発見時のみ追記（YAML形式）
- 診断後: `root_cause`, `artifacts`, `missing`, `debug_session` を記入
- このセクションは /gsd:plan-phase --gaps に直接供給される

</section_rules>

<diagnosis_lifecycle>

**テスト完了後（status: complete）でギャップが存在する場合:**

1. ユーザーが診断を実行（verify-workからの提案または手動で）
2. diagnose-issuesワークフローが並列デバッグエージェントを起動
3. 各エージェントが1つのギャップを調査し、根本原因を返す
4. UAT.mdのGapsセクションが診断で更新:
   - 各ギャップに `root_cause`, `artifacts`, `missing`, `debug_session` が記入
5. status → "diagnosed"
6. 根本原因付きで /gsd:plan-phase --gaps の準備完了

**診断後:**
```yaml
## Gaps

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
  debug_session: ".planning/debug/comment-not-refreshing.md"
```

</diagnosis_lifecycle>

<lifecycle>

**作成:** /gsd:verify-workが新しいセッションを開始する時
- SUMMARY.mdファイルからテストを抽出
- statusを "testing" に設定
- Current Testをテスト1に設定
- すべてのテストを result: [pending] に設定

**テスト中:**
- Current Testセクションからテストを提示
- ユーザーが合格確認または問題の説明で回答
- テスト結果を更新（pass/issue/skipped）
- Summaryカウントを更新
- issueの場合: Gapsセクションに追記（YAML形式）、重大度を推定
- Current Testを次の保留中テストに移動

**完了時:**
- status → "complete"
- Current Test → "[testing complete]"
- ファイルをコミット
- サマリーと次のステップを提示

**/clear後の再開:**
1. フロントマターを読む → フェーズとステータスを把握
2. Current Testを読む → 現在地を把握
3. 最初の[pending]結果を見つける → そこから継続
4. Summaryでこれまでの進捗を表示

</lifecycle>

<severity_guide>

重大度はユーザーの自然言語から推定され、決して直接尋ねません。

| ユーザーの説明 | 推定 |
|----------------|-------|
| クラッシュ、エラー、例外、完全に失敗、使用不可 | blocker |
| 動かない、何も起こらない、間違った動作、欠落 | major |
| 動くけど...、遅い、おかしい、軽微、小さな問題 | minor |
| 色、フォント、余白、配置、視覚的、見た目がずれている | cosmetic |

デフォルト: **major**（安全なデフォルト、間違っていればユーザーが訂正可能）

</severity_guide>

<good_example>
```markdown
---
status: diagnosed
phase: 04-comments
source: 04-01-SUMMARY.md, 04-02-SUMMARY.md
started: 2025-01-15T10:30:00Z
updated: 2025-01-15T10:45:00Z
---

## Current Test

[testing complete]

## Tests

### 1. 投稿のコメント表示
expected: コメントセクションが展開し、件数とコメントリストを表示
result: pass

### 2. トップレベルコメントの作成
expected: リッチテキストエディターでコメントを送信、作成者情報付きでリストに表示
result: issue
reported: "works but doesn't show until I refresh the page"
severity: major

### 3. コメントへの返信
expected: 返信をクリック、インラインコンポーザーが表示、送信するとネストされた返信が表示
result: pass

### 4. ビジュアルネスト
expected: 3段階以上のスレッドがインデント、左ボーダーを表示、適切な深さで制限
result: pass

### 5. 自分のコメントの削除
expected: 自分のコメントで削除をクリック、削除されるか返信がある場合は[deleted]を表示
result: pass

### 6. コメント数
expected: 投稿に正確な数を表示、コメント追加時にインクリメント
result: pass

## Summary

total: 6
passed: 5
issues: 1
pending: 0
skipped: 0

## Gaps

- truth: "Comment appears immediately after submission in list"
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
  debug_session: ".planning/debug/comment-not-refreshing.md"
```
</good_example>
