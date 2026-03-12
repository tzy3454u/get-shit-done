# デバッグサブエージェントプロンプトテンプレート

gsd-debuggerエージェントを起動するためのテンプレート。エージェントにはすべてのデバッグ専門知識が含まれています — このテンプレートは問題のコンテキストのみを提供します。

---

## テンプレート

```markdown
<objective>
Investigate issue: {issue_id}

**Summary:** {issue_summary}
</objective>

<symptoms>
expected: {expected}
actual: {actual}
errors: {errors}
reproduction: {reproduction}
timeline: {timeline}
</symptoms>

<mode>
symptoms_prefilled: {true_or_false}
goal: {find_root_cause_only | find_and_fix}
</mode>

<debug_file>
Create: .planning/debug/{slug}.md
</debug_file>
```

---

## プレースホルダー

| プレースホルダー | ソース | 例 |
|-------------|--------|---------|
| `{issue_id}` | オーケストレーター割り当て | `auth-screen-dark` |
| `{issue_summary}` | ユーザーの説明 | `Auth screen is too dark` |
| `{expected}` | 症状から | `See logo clearly` |
| `{actual}` | 症状から | `Screen is dark` |
| `{errors}` | 症状から | `None in console` |
| `{reproduction}` | 症状から | `Open /auth page` |
| `{timeline}` | 症状から | `After recent deploy` |
| `{goal}` | オーケストレーターが設定 | `find_and_fix` |
| `{slug}` | 生成される | `auth-screen-dark` |

---

## 使用方法

**From /gsd:debug:**
```python
Task(
  prompt=filled_template,
  subagent_type="gsd-debugger",
  description="Debug {slug}"
)
```

**From diagnose-issues (UAT):**
```python
Task(prompt=template, subagent_type="gsd-debugger", description="Debug UAT-001")
```

---

## 継続

チェックポイントの場合、以下の内容で新しいエージェントを起動します:

```markdown
<objective>
Continue debugging {slug}. Evidence is in the debug file.
</objective>

<prior_state>
Debug file: @.planning/debug/{slug}.md
</prior_state>

<checkpoint_response>
**Type:** {checkpoint_type}
**Response:** {user_response}
</checkpoint_response>

<mode>
goal: {goal}
</mode>
```
</output>