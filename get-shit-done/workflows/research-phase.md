<purpose>
フェーズの実装方法を調査する。フェーズコンテキストを持つgsd-phase-researcherを生成する。

独立した調査コマンド。ほとんどのワークフローでは、調査を自動的に統合する`/gsd:plan-phase`を使用すること。
</purpose>

<process>

## Step 0: モデルプロファイルの解決

@~/.claude/get-shit-done/references/model-profile-resolution.md

以下のモデルを解決する:
- `gsd-phase-researcher`

## Step 1: フェーズの正規化と検証

@~/.claude/get-shit-done/references/phase-argument-parsing.md

```bash
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
```

`found`がfalseの場合: エラーを表示して終了。

## Step 2: 既存の調査を確認

```bash
ls .planning/phases/${PHASE}-*/RESEARCH.md 2>/dev/null
```

存在する場合: 更新/表示/スキップのオプションを提示する。

## Step 3: フェーズコンテキストの収集

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# 抽出: phase_dir, padded_phase, phase_number, state_path, requirements_path, context_path
```

## Step 4: リサーチャーの生成

```
Task(
  prompt="<objective>
Research implementation approach for Phase {phase}: {name}
</objective>

<files_to_read>
- {context_path} (/gsd:discuss-phaseからのユーザー決定)
- {requirements_path} (プロジェクト要件)
- {state_path} (プロジェクトの決定と履歴)
</files_to_read>

<additional_context>
Phase description: {description}
</additional_context>

<output>
Write to: .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md
</output>
  subagent_type="gsd-phase-researcher",
  model="{researcher_model}"
)
```

## Step 5: 戻り値の処理

- `## RESEARCH COMPLETE` — サマリーを表示し、次のオプションを提示: 計画/深掘り/レビュー/完了
- `## CHECKPOINT REACHED` — ユーザーに提示し、続行を生成
- `## RESEARCH INCONCLUSIVE` — 試行内容を表示し、次のオプションを提示: コンテキスト追加/別モードで試行/手動

</process>

