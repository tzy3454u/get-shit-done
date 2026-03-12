<purpose>
ロードマップの現在のマイルストーンの末尾に新しい整数フェーズを追加する。次のフェーズ番号を自動計算し、フェーズディレクトリを作成し、ロードマップ構造を更新する。
</purpose>

<required_reading>
開始前に、呼び出しプロンプトのexecution_contextで参照されるすべてのファイルを読み取ること。
</required_reading>

<process>

<step name="parse_arguments">
コマンド引数をパース:
- すべての引数がフェーズの説明となる
- 例: `/gsd:add-phase Add authentication` → description = "Add authentication"
- 例: `/gsd:add-phase Fix critical performance issues` → description = "Fix critical performance issues"

引数が指定されていない場合:

```
ERROR: フェーズの説明が必要
使い方: /gsd:add-phase <description>
例: /gsd:add-phase Add authentication system
```

終了。
</step>

<step name="init_context">
フェーズ操作コンテキストを読み込む:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "0")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから`roadmap_exists`を確認。falseの場合:
```
ERROR: ロードマップが見つかりません (.planning/ROADMAP.md)
/gsd:new-projectを実行して初期化すること。
```
終了。
</step>

<step name="add_phase">
**フェーズの追加をgsd-toolsに委任:**

```bash
RESULT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase add "${description}")
```

CLIが処理する内容:
- 既存の最大整数フェーズ番号の検出
- 次のフェーズ番号の計算（max + 1）
- 説明からスラグを生成
- フェーズディレクトリの作成（`.planning/phases/{NN}-{slug}/`）
- Goal、Depends on、Plansセクションを持つフェーズエントリのROADMAP.mdへの挿入

結果から抽出: `phase_number`, `padded`, `name`, `slug`, `directory`。
</step>

<step name="update_project_state">
新しいフェーズを反映するためにSTATE.mdを更新:

1. `.planning/STATE.md`を読み取る
2. "## Accumulated Context" → "### Roadmap Evolution"の下にエントリを追加:
   ```
   - Phase {N} added: {description}
   ```

"Roadmap Evolution"セクションが存在しない場合は作成する。
</step>

<step name="completion">
完了サマリーを提示:

```
フェーズ{N}を現在のマイルストーンに追加しました:
- 説明: {description}
- ディレクトリ: .planning/phases/{phase-num}-{slug}/
- ステータス: 未計画

ロードマップを更新: .planning/ROADMAP.md

---

## ▶ Next Up

**フェーズ {N}: {description}**

`/gsd:plan-phase {N}`

<sub>`/clear` first → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション:**
- `/gsd:add-phase <description>` — 別のフェーズを追加
- ロードマップを確認

---
```
</step>

</process>

<success_criteria>
- [ ] `gsd-tools phase add`が正常に実行された
- [ ] フェーズディレクトリが作成された
- [ ] 新しいフェーズエントリでロードマップが更新された
- [ ] ロードマップの変更メモでSTATE.mdが更新された
- [ ] 次のステップをユーザーに通知した
</success_criteria>

