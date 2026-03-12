# プランナーサブエージェントプロンプトテンプレート

gsd-plannerエージェントを起動するためのテンプレート。エージェントにはすべての計画の専門知識が含まれています — このテンプレートは計画コンテキストのみを提供します。

---

## テンプレート

```markdown
<planning_context>

**Phase:** {phase_number}
**Mode:** {standard | gap_closure}

**Project State:**
@.planning/STATE.md

**Roadmap:**
@.planning/ROADMAP.md

**Requirements (if exists):**
@.planning/REQUIREMENTS.md

**Phase Context (if exists):**
@.planning/phases/{phase_dir}/{phase_num}-CONTEXT.md

**Research (if exists):**
@.planning/phases/{phase_dir}/{phase_num}-RESEARCH.md

**Gap Closure (if --gaps mode):**
@.planning/phases/{phase_dir}/{phase_num}-VERIFICATION.md
@.planning/phases/{phase_dir}/{phase_num}-UAT.md

</planning_context>

<downstream_consumer>
出力は/gsd:execute-phaseによって消費される
プランは以下を含む実行可能なプロンプトでなければならない:
- フロントマター（wave、depends_on、files_modified、autonomous）
- XML形式のタスク
- 検証基準
- ゴールからの逆算検証のためのmust_haves
</downstream_consumer>

<quality_gate>
PLANNING COMPLETEを返す前に:
- [ ] PLAN.mdファイルがフェーズディレクトリに作成されている
- [ ] 各プランに有効なフロントマターがある
- [ ] タスクが具体的で実行可能
- [ ] 依存関係が正しく特定されている
- [ ] 並列実行のためのウェーブが割り当てられている
- [ ] フェーズゴールからmust_havesが導出されている
</quality_gate>
```

---

## プレースホルダー

| プレースホルダー | ソース | 例 |
|-------------|--------|---------|
| `{phase_number}` | ロードマップ/引数から | `5`または`2.1` |
| `{phase_dir}` | フェーズディレクトリ名 | `05-user-profiles` |
| `{phase}` | フェーズプレフィックス | `05` |
| `{standard \| gap_closure}` | モードフラグ | `standard` |

---

## 使用方法

**From /gsd:plan-phase（標準モード）:**
```python
Task(
  prompt=filled_template,
  subagent_type="gsd-planner",
  description="Plan Phase {phase}"
)
```

**From /gsd:plan-phase --gaps（ギャップクロージャーモード）:**
```python
Task(
  prompt=filled_template,  # mode: gap_closureで
  subagent_type="gsd-planner",
  description="Plan gaps for Phase {phase}"
)
```

---

## 継続

チェックポイントの場合、以下の内容で新しいエージェントを起動:

```markdown
<objective>
Continue planning for Phase {phase_number}: {phase_name}
</objective>

<prior_state>
Phase directory: @.planning/phases/{phase_dir}/
Existing plans: @.planning/phases/{phase_dir}/*-PLAN.md
</prior_state>

<checkpoint_response>
**Type:** {checkpoint_type}
**Response:** {user_response}
</checkpoint_response>

<mode>
Continue: {standard | gap_closure}
</mode>
```

---

**注意:** 計画手法、タスク分解、依存関係分析、ウェーブ割り当て、TDD検出、ゴールからの逆算導出はgsd-plannerエージェントに組み込まれています。このテンプレートはコンテキストを渡すだけです。
</output>