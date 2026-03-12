# デバッグテンプレート

`.planning/debug/[slug].md` 用テンプレート — アクティブなデバッグセッションの追跡。

---

## ファイルテンプレート

```markdown
---
status: gathering | investigating | fixing | verifying | awaiting_human_verify | resolved
trigger: "[ユーザー入力そのまま]"
created: [ISOタイムスタンプ]
updated: [ISOタイムスタンプ]
---

## Current Focus
<!-- 更新のたびに上書き - 常に現在の状態を反映 -->

hypothesis: [テスト中の現在の仮説]
test: [どのようにテストしているか]
expecting: [真/偽の場合に結果が何を意味するか]
next_action: [直近の次のステップ]

## Symptoms
<!-- 収集中に記述、その後不変 -->

expected: [何が起こるべきか]
actual: [実際に何が起こるか]
errors: [エラーメッセージがある場合]
reproduction: [どのようにトリガーするか]
started: [いつ壊れたか / 最初から壊れていたか]

## Eliminated
<!-- 追記のみ - /clear後の再調査を防止 -->

- hypothesis: [誤っていた仮説]
  evidence: [反証したもの]
  timestamp: [排除された時刻]

## Evidence
<!-- 追記のみ - 調査中に発見された事実 -->

- timestamp: [発見時刻]
  checked: [何を調べたか]
  found: [何が観察されたか]
  implication: [これが何を意味するか]

## Resolution
<!-- 理解が深まるにつれて上書き -->

root_cause: [見つかるまで空]
fix: [適用されるまで空]
verification: [検証されるまで空]
files_changed: []
```

---

<section_rules>

**フロントマター（status、trigger、タイムスタンプ）:**
- `status`: 上書き - 現在のフェーズを反映
- `trigger`: 不変 - ユーザー入力そのまま、変更しない
- `created`: 不変 - 一度だけ設定
- `updated`: 上書き - 変更のたびに更新

**Current Focus:**
- 更新のたびに全体を上書き
- 常にClaudeが今何をしているかを反映
- Claudeが/clear後にこれを読めば、正確にどこで再開すべきかがわかる
- フィールド: hypothesis、test、expecting、next_action

**Symptoms:**
- 初期の収集フェーズで記述
- 収集完了後は不変
- 修正しようとしている内容の参照ポイント
- フィールド: expected、actual、errors、reproduction、started

**Eliminated:**
- 追記のみ - エントリを削除しない
- コンテキストリセット後の行き止まりの再調査を防止
- 各エントリ: 仮説、反証した証拠、タイムスタンプ
- /clearの境界を越えた効率性に不可欠

**Evidence:**
- 追記のみ - エントリを削除しない
- 調査中に発見された事実
- 各エントリ: タイムスタンプ、何を調べたか、何を発見したか、意味
- 根本原因の根拠を構築

**Resolution:**
- 理解が深まるにつれて上書き
- 修正が試みられるにつれて複数回更新される可能性あり
- 最終状態は確認された根本原因と検証済みの修正を示す
- フィールド: root_cause、fix、verification、files_changed

</section_rules>

<lifecycle>

**作成:** /gsd:debugが呼ばれたとき即座に
- ユーザー入力からtriggerを含むファイルを作成
- statusを"gathering"に設定
- Current Focus: next_action = "gather symptoms"
- Symptoms: 空、記入予定

**症状収集中:**
- ユーザーが質問に答えるたびにSymptomsセクションを更新
- 各質問でCurrent Focusを更新
- 完了時: status → "investigating"

**調査中:**
- 各仮説でCurrent Focusを上書き
- 各発見でEvidenceに追記
- 仮説が反証されたらEliminatedに追記
- フロントマターのタイムスタンプを更新

**修正中:**
- status → "fixing"
- 確認されたらResolution.root_causeを更新
- 適用されたらResolution.fixを更新
- Resolution.files_changedを更新

**検証中:**
- status → "verifying"
- Resolution.verificationに結果を更新
- 検証失敗時: status → "investigating"、再試行

**自己検証通過後:**
- status -> "awaiting_human_verify"
- チェックポイントで明示的なユーザー確認を要求
- まだファイルをresolvedに移動しない

**解決時:**
- status → "resolved"
- ファイルを.planning/debug/resolved/に移動（ユーザーが修正を確認した後のみ）

</lifecycle>

<resume_behavior>

Claudeが/clear後にこのファイルを読んだ場合:

1. フロントマターを解析 → statusを把握
2. Current Focusを読む → 正確に何をしていたかを把握
3. Eliminatedを読む → 再試行すべきでないものを把握
4. Evidenceを読む → 何が学ばれたかを把握
5. next_actionから継続

このファイルがデバッグの頭脳です。Claudeはどの中断ポイントからでも完璧に再開できるべきです。

</resume_behavior>

<size_constraint>

デバッグファイルは簡潔に保つ:
- Evidenceエントリ: 各1〜2行、事実のみ
- Eliminated: 簡潔に - 仮説 + 失敗した理由
- 散文的な説明なし - 構造化されたデータのみ

エビデンスが非常に多くなった場合（10以上のエントリ）、堂々巡りしていないか検討してください。Eliminatedを確認して同じことを繰り返していないか確認してください。

</size_constraint>
</output>