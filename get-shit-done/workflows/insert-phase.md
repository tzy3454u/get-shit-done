<purpose>
マイルストーン途中で発見された緊急作業のために、既存の整数フェーズ間に小数フェーズを挿入する。小数番号（72.1、72.2など）を使用して、ロードマップ全体の番号を振り直すことなく、計画済みフェーズの論理的な順序を保持しながら緊急挿入に対応する。
</purpose>

<required_reading>
開始前に、呼び出し元プロンプトのexecution_contextで参照されているすべてのファイルを読み込むこと。
</required_reading>

<process>

<step name="parse_arguments">
コマンド引数をパースする：
- 第1引数：挿入する位置の後の整数フェーズ番号
- 残りの引数：フェーズの説明

例：`/gsd:insert-phase 72 Fix critical auth bug`
-> after = 72
-> description = "Fix critical auth bug"

引数が不足している場合：

```
ERROR: フェーズ番号と説明の両方が必要です
使い方: /gsd:insert-phase <after> <description>
例: /gsd:insert-phase 72 Fix critical auth bug
```

終了。

第1引数が整数であることを検証する。
</step>

<step name="init_context">
フェーズ操作コンテキストを読み込む：

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${after_phase}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

init JSONから `roadmap_exists` を確認する。falseの場合：
```
ERROR: ロードマップが見つかりません (.planning/ROADMAP.md)
```
終了。
</step>

<step name="insert_phase">
**フェーズの挿入をgsd-toolsに委譲する：**

```bash
RESULT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase insert "${after_phase}" "${description}")
```

CLIが処理する内容：
- 対象フェーズがROADMAP.mdに存在するか検証
- 次の小数フェーズ番号を計算（ディスク上の既存の小数を確認）
- 説明からスラグを生成
- フェーズディレクトリを作成 (`.planning/phases/{N.M}-{slug}/`)
- 対象フェーズの後に(INSERTED)マーカー付きでROADMAP.mdにフェーズエントリを挿入

結果から抽出する：`phase_number`、`after_phase`、`name`、`slug`、`directory`。
</step>

<step name="update_project_state">
挿入されたフェーズを反映するためにSTATE.mdを更新する：

1. `.planning/STATE.md` を読み込む
2. "## Accumulated Context" → "### Roadmap Evolution" の下にエントリを追加する：
   ```
   - Phase {decimal_phase} をPhase {after_phase} の後に挿入: {description} (URGENT)
   ```

"Roadmap Evolution" セクションが存在しない場合は作成する。
</step>

<step name="completion">
完了サマリーを表示する：

```
Phase {decimal_phase} をPhase {after_phase} の後に挿入しました：
- 説明: {description}
- ディレクトリ: .planning/phases/{decimal-phase}-{slug}/
- ステータス: 未計画
- マーカー: (INSERTED) - 緊急作業を示す

ロードマップ更新済み: .planning/ROADMAP.md
プロジェクト状態更新済み: .planning/STATE.md

---

## Next Up

**Phase {decimal_phase}: {description}** -- 緊急挿入

`/gsd:plan-phase {decimal_phase}`

<sub>`/clear` first -> fresh context window</sub>

---

**その他のオプション：**
- 挿入の影響を確認: Phase {next_integer} の依存関係がまだ妥当か確認
- ロードマップを確認

---
```
</step>

</process>

<anti_patterns>

- マイルストーン末尾の計画済み作業にはこれを使わない（/gsd:add-phase を使用）
- Phase 1 の前に挿入しない（小数 0.1 は意味がない）
- 既存のフェーズの番号を振り直さない
- 対象フェーズの内容を変更しない
- まだ計画を作成しない（それは /gsd:plan-phase で行う）
- 変更をコミットしない（ユーザーがコミットのタイミングを決める）
</anti_patterns>

<success_criteria>
フェーズ挿入が完了する条件：

- [ ] `gsd-tools phase insert` が正常に実行されている
- [ ] フェーズディレクトリが作成されている
- [ ] ロードマップが新しいフェーズエントリで更新されている（"(INSERTED)" マーカーを含む）
- [ ] STATE.mdがロードマップ進化メモで更新されている
- [ ] ユーザーに次のステップと依存関係への影響が通知されている
</success_criteria>
</output>
