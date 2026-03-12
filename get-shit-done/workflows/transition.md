<required_reading>

**今すぐこれらのファイルを読み込んでください：**

1. `.planning/STATE.md`
2. `.planning/PROJECT.md`
3. `.planning/ROADMAP.md`
4. 現在のフェーズのプランファイル（`*-PLAN.md`）
5. 現在のフェーズのサマリーファイル（`*-SUMMARY.md`）

</required_reading>

<purpose>

現在のフェーズを完了としてマークし、次に進みます。これは進捗追跡とPROJECT.mdの進化が行われる自然なポイントです。

「次のフェーズを計画する」=「現在のフェーズは完了」

</purpose>

<process>

<step name="load_project_state" priority="first">

移行前にプロジェクトの状態を読み込みます：

```bash
cat .planning/STATE.md 2>/dev/null
cat .planning/PROJECT.md 2>/dev/null
```

正しいフェーズを移行していることを確認するため、現在の位置を解析します。
移行後に更新が必要な蓄積されたコンテキストに注意します。

</step>

<step name="verify_completion">

現在のフェーズにすべてのプランサマリーがあるか確認します：

```bash
ls .planning/phases/XX-current/*-PLAN.md 2>/dev/null | sort
ls .planning/phases/XX-current/*-SUMMARY.md 2>/dev/null | sort
```

**検証ロジック：**

- PLANファイルをカウント
- SUMMARYファイルをカウント
- カウントが一致する場合：すべてのプランが完了
- カウントが一致しない場合：未完了

<config-check>

```bash
cat .planning/config.json 2>/dev/null
```

</config-check>

**すべてのプランが完了している場合：**

<if mode="yolo">

```
⚡ 自動承認: フェーズ [X] → フェーズ [X+1] への移行
フェーズ [X] 完了 — すべての [Y] プランが終了しました。

完了マークと進行を実行中...
```

cleanup_handoffステップに直接進みます。

</if>

<if mode="interactive" OR="custom with gates.confirm_transition true">

質問: 「フェーズ [X] 完了 — すべての [Y] プランが終了しました。完了としてマークしてフェーズ [X+1] に進みますか？」

確認を待ってから進行します。

</if>

**プランが未完了の場合：**

**安全ガードレール: always_confirm_destructiveがここに適用されます。**
未完了プランのスキップは破壊的操作です — モードに関係なく常にプロンプトを表示します。

提示：

```
フェーズ [X] に未完了のプランがあります:
- {phase}-01-SUMMARY.md ✓ 完了
- {phase}-02-SUMMARY.md ✗ 欠落
- {phase}-03-SUMMARY.md ✗ 欠落

⚠️ 安全ガードレール: プランのスキップには確認が必要です（破壊的操作）

オプション:
1. 現在のフェーズを続行（残りのプランを実行）
2. それでも完了としてマーク（残りのプランをスキップ）
3. 残りの作業を確認
```

ユーザーの決定を待ちます。

</step>

<step name="cleanup_handoff">

残っているハンドオフを確認します：

```bash
ls .planning/phases/XX-current/.continue-here*.md 2>/dev/null
```

見つかった場合は削除します — フェーズが完了しているため、ハンドオフは古くなっています。

</step>

<step name="update_roadmap_and_state">

**ROADMAP.mdとSTATE.mdの更新をgsd-toolsに委譲します：**

```bash
TRANSITION=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase complete "${current_phase}")
```

CLIが処理する内容：
- フェーズのチェックボックスを本日の日付で`[x]`完了としてマーク
- プランカウントを最終値に更新（例: 「3/3プラン完了」）
- Progressテーブルの更新（Status → Complete、日付を追加）
- STATE.mdを次のフェーズに進める（Current Phase、Status → Ready to plan、Current Plan → Not started）
- これがマイルストーンの最後のフェーズかどうかを検出

結果から抽出: `completed_phase`、`plans_executed`、`next_phase`、`next_phase_name`、`is_last_phase`。

</step>

<step name="archive_prompts">

フェーズ用に生成されたプロンプトがある場合、そのまま残します。
create-meta-promptsの`completed/`サブフォルダパターンがアーカイブを処理します。

</step>

<step name="evolve_project">

完了したフェーズからの学びを反映してPROJECT.mdを進化させます。

**フェーズサマリーを読み込みます：**

```bash
cat .planning/phases/XX-current/*-SUMMARY.md
```

**要件の変更を評価します：**

1. **要件が検証されたか？**
   - このフェーズでアクティブな要件が出荷されたか？
   - フェーズ参照付きでValidatedに移動: `- ✓ [要件] — Phase X`

2. **要件が無効化されたか？**
   - アクティブな要件が不要または誤りと判明したか？
   - 理由付きでOut of Scopeに移動: `- [要件] — [無効化の理由]`

3. **要件が出現したか？**
   - 構築中に新しい要件が発見されたか？
   - Activeに追加: `- [ ] [新しい要件]`

4. **記録すべき決定はあるか？**
   - SUMMARY.mdファイルから決定事項を抽出
   - 結果が分かっていればKey Decisionsテーブルに追加

5. **「What This Is」はまだ正確か？**
   - プロダクトが大きく変わった場合、説明を更新
   - 常に最新で正確に保つ

**PROJECT.mdを更新します：**

インラインで編集します。「Last updated」フッターを更新：

```markdown
---
*Last updated: [date] after Phase [X]*
```

**進化の例：**

変更前：

```markdown
### Active

- [ ] JWT authentication
- [ ] Real-time sync < 500ms
- [ ] Offline mode

### Out of Scope

- OAuth2 — complexity not needed for v1
```

変更後（フェーズ2でJWT認証を出荷し、レート制限が必要と判明）：

```markdown
### Validated

- ✓ JWT authentication — Phase 2

### Active

- [ ] Real-time sync < 500ms
- [ ] Offline mode
- [ ] Rate limiting on sync endpoint

### Out of Scope

- OAuth2 — complexity not needed for v1
```

**ステップ完了条件：**

- [ ] フェーズサマリーから学びが確認された
- [ ] 検証済み要件がActiveから移動された
- [ ] 無効化された要件が理由付きでOut of Scopeに移動された
- [ ] 出現した要件がActiveに追加された
- [ ] 新しい決定事項が理由とともに記録された
- [ ] プロダクトが変更された場合「What This Is」が更新された
- [ ] 「Last updated」フッターがこの移行を反映している

</step>

<step name="update_current_position_after_transition">

**注意:** 基本的な位置更新（Current Phase、Status、Current Plan、Last Activity）は、update_roadmap_and_stateステップの`gsd-tools phase complete`で既に処理されています。

STATE.mdを読み込んで更新が正しいか確認します。プログレスバーの更新が必要な場合は以下を使用：

```bash
PROGRESS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" progress bar --raw)
```

STATE.mdのプログレスバー行を結果で更新します。

**ステップ完了条件：**

- [ ] フェーズ番号が次のフェーズに増加された（phase completeで処理済み）
- [ ] プランステータスが「Not started」にリセットされた（phase completeで処理済み）
- [ ] ステータスが「Ready to plan」を表示している（phase completeで処理済み）
- [ ] プログレスバーが完了プランの合計を反映している

</step>

<step name="update_project_reference">

STATE.mdのProject Referenceセクションを更新します。

```markdown
## Project Reference

See: .planning/PROJECT.md (updated [today])

**Core value:** [PROJECT.mdからの現在のコアバリュー]
**Current focus:** [次のフェーズ名]
```

移行を反映するよう日付と現在のフォーカスを更新します。

</step>

<step name="review_accumulated_context">

STATE.mdのAccumulated Contextセクションを確認・更新します。

**決定事項：**

- このフェーズからの最近の決定事項を記載（最大3-5件）
- 完全なログはPROJECT.mdのKey Decisionsテーブルにある

**ブロッカー/懸念事項：**

- 完了したフェーズからのブロッカーを確認
- このフェーズで対処された場合：リストから削除
- 将来も関連がある場合：「Phase X」プレフィックス付きで保持
- 完了したフェーズのサマリーからの新しい懸念事項を追加

**例：**

変更前：

```markdown
### Blockers/Concerns

- ⚠️ [Phase 1] Database schema not indexed for common queries
- ⚠️ [Phase 2] WebSocket reconnection behavior on flaky networks unknown
```

変更後（フェーズ2でデータベースインデックスが対処された場合）：

```markdown
### Blockers/Concerns

- ⚠️ [Phase 2] WebSocket reconnection behavior on flaky networks unknown
```

**ステップ完了条件：**

- [ ] 最近の決定事項が記載された（完全なログはPROJECT.mdに）
- [ ] 解決済みブロッカーがリストから削除された
- [ ] 未解決のブロッカーがフェーズプレフィックス付きで保持された
- [ ] 完了したフェーズからの新しい懸念事項が追加された

</step>

<step name="update_session_continuity_after_transition">

移行完了を反映するためSTATE.mdのSession Continuityセクションを更新します。

**フォーマット：**

```markdown
Last session: [today]
Stopped at: Phase [X] complete, ready to plan Phase [X+1]
Resume file: None
```

**ステップ完了条件：**

- [ ] 最終セッションのタイムスタンプが現在の日時に更新された
- [ ] Stopped atにフェーズ完了と次のフェーズが記述された
- [ ] ResumeファイルがNoneであることが確認された（移行ではresumeファイルを使用しない）

</step>

<step name="offer_next_phase">

**必須: 次のステップを提示する前にマイルストーンステータスを確認してください。**

**`gsd-tools phase complete`からの移行結果を使用します：**

phase completeの結果の`is_last_phase`フィールドが直接教えてくれます：
- `is_last_phase: false` → さらにフェーズがある → **ルートA**へ
- `is_last_phase: true` → マイルストーン完了 → **ルートB**へ

`next_phase`と`next_phase_name`フィールドが次のフェーズの詳細を提供します。

追加のコンテキストが必要な場合は以下を使用：
```bash
ROADMAP=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)
```

すべてのフェーズの目標、ディスクステータス、完了情報を返します。

---

**ルートA: マイルストーンにさらにフェーズが残っている**

ROADMAP.mdを読み込んで次のフェーズの名前と目標を取得します。

**次のフェーズにCONTEXT.mdがあるか確認：**

```bash
ls .planning/phases/*[X+1]*/*-CONTEXT.md 2>/dev/null
```

**次のフェーズが存在する場合：**

<if mode="yolo">

**CONTEXT.mdが存在する場合：**

```
フェーズ [X] を完了としてマークしました。

次: フェーズ [X+1] — [Name]

⚡ 自動続行: フェーズ [X+1] を詳細に計画
```

スキルを終了しSlashCommand("/gsd:plan-phase [X+1] --auto")を呼び出します

**CONTEXT.mdが存在しない場合：**

```
フェーズ [X] を完了としてマークしました。

次: フェーズ [X+1] — [Name]

⚡ 自動続行: まずフェーズ [X+1] を議論
```

スキルを終了しSlashCommand("/gsd:discuss-phase [X+1] --auto")を呼び出します

</if>

<if mode="interactive" OR="custom with gates.confirm_transition true">

**CONTEXT.mdが存在しない場合：**

```
## ✓ フェーズ [X] 完了

---

## ▶ 次のステップ

**フェーズ [X+1]: [Name]** — [ROADMAP.mdからの目標]

`/gsd:discuss-phase [X+1]` — コンテキストを収集しアプローチを明確化

<sub>先に`/clear` → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション:**
- `/gsd:plan-phase [X+1]` — 議論をスキップし直接計画
- `/gsd:research-phase [X+1]` — 不明点を調査

---
```

**CONTEXT.mdが存在する場合：**

```
## ✓ フェーズ [X] 完了

---

## ▶ 次のステップ

**フェーズ [X+1]: [Name]** — [ROADMAP.mdからの目標]
<sub>✓ コンテキスト収集済み、計画準備完了</sub>

`/gsd:plan-phase [X+1]`

<sub>先に`/clear` → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション:**
- `/gsd:discuss-phase [X+1]` — コンテキストを再確認
- `/gsd:research-phase [X+1]` — 不明点を調査

---
```

</if>

---

**ルートB: マイルストーン完了（すべてのフェーズが完了）**

**自動進行チェーンフラグをクリア** — マイルストーン境界が自然な停止ポイントです：
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false
```

<if mode="yolo">

```
フェーズ {X} を完了としてマークしました。

🎉 マイルストーン {version} が100%完了 — すべての {N} フェーズが終了しました！

⚡ 自動続行: マイルストーンを完了しアーカイブ
```

スキルを終了しSlashCommand("/gsd:complete-milestone {version}")を呼び出します

</if>

<if mode="interactive" OR="custom with gates.confirm_transition true">

```
## ✓ フェーズ {X}: {Phase Name} 完了

🎉 マイルストーン {version} が100%完了 — すべての {N} フェーズが終了しました！

---

## ▶ 次のステップ

**マイルストーン {version} を完了** — アーカイブして次の準備

`/gsd:complete-milestone {version}`

<sub>先に`/clear` → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション:**
- アーカイブ前に成果を確認

---
```

</if>

</step>

</process>

<implicit_tracking>
進捗追跡は暗黙的です：フェーズNの計画はフェーズ1-(N-1)の完了を意味します。個別の進捗ステップはありません — 前進すること自体が進捗です。
</implicit_tracking>

<partial_completion>

ユーザーが先に進みたいがフェーズが完全に完了していない場合：

```
フェーズ [X] に未完了のプランがあります:
- {phase}-02-PLAN.md（未実行）
- {phase}-03-PLAN.md（未実行）

オプション:
1. それでも完了としてマーク（プランは不要だった）
2. 作業を後のフェーズに延期
3. 現在のフェーズに留まり完了させる
```

ユーザーの判断を尊重してください — 作業が重要かどうかは本人が分かっています。

**未完了のプランで完了としてマークする場合：**

- ROADMAPを更新: 「2/3プラン完了」（「3/3」ではなく）
- 移行メッセージにどのプランがスキップされたか記載

</partial_completion>

<success_criteria>

移行は以下の条件を満たした時に完了です：

- [ ] 現在のフェーズのプランサマリーが検証された（すべて存在するか、ユーザーがスキップを選択）
- [ ] 古いハンドオフが削除された
- [ ] ROADMAP.mdが完了ステータスとプランカウントで更新された
- [ ] PROJECT.mdが進化した（要件、決定事項、必要に応じて説明）
- [ ] STATE.mdが更新された（位置、プロジェクト参照、コンテキスト、セッション）
- [ ] Progressテーブルが更新された
- [ ] ユーザーが次のステップを把握した

</success_criteria>
