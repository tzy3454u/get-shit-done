<purpose>
未開始の将来フェーズをプロジェクトロードマップから削除し、そのディレクトリを削除し、後続のすべてのフェーズを再番号付けしてクリーンな連番を維持し、変更をコミットします。gitコミットが削除の履歴記録として機能します。
</purpose>

<required_reading>
開始前に、呼び出しプロンプトのexecution_contextで参照されているすべてのファイルを読み込んでください。
</required_reading>

<process>

<step name="parse_arguments">
コマンド引数を解析します：
- 引数は削除するフェーズ番号（整数または小数）
- 例: `/gsd:remove-phase 17` → phase = 17
- 例: `/gsd:remove-phase 16.1` → phase = 16.1

引数が指定されていない場合：

```
ERROR: フェーズ番号が必要です
使用方法: /gsd:remove-phase <phase-number>
例: /gsd:remove-phase 17
```

終了します。
</step>

<step name="init_context">
フェーズ操作コンテキストを読み込みます：

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${target}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

抽出: `phase_found`、`phase_dir`、`phase_number`、`commit_docs`、`roadmap_exists`。

STATE.mdとROADMAP.mdの内容も読み込んで、現在の位置を解析します。
</step>

<step name="validate_future_phase">
対象が将来のフェーズ（未開始）であることを確認します：

1. STATE.mdから現在のフェーズとターゲットフェーズを比較
2. ターゲットは現在のフェーズ番号より大きい必要がある

ターゲットが現在のフェーズ以下の場合：

```
ERROR: フェーズ {target} を削除できません

将来のフェーズのみ削除可能です:
- 現在のフェーズ: {current}
- フェーズ {target} は現在または完了済みです

現在の作業を中断するには、代わりに /gsd:pause-work を使用してください。
```

終了します。
</step>

<step name="confirm_removal">
削除の概要を提示して確認します：

```
フェーズ {target}: {Name} を削除します

以下の処理が実行されます:
- 削除: .planning/phases/{target}-{slug}/
- 後続のすべてのフェーズを再番号付け
- 更新: ROADMAP.md、STATE.md

続行しますか？ (y/n)
```

確認を待ちます。
</step>

<step name="execute_removal">
**削除操作全体をgsd-toolsに委譲します：**

```bash
RESULT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase remove "${target}")
```

フェーズに実行済みプラン（SUMMARY.mdファイル）がある場合、gsd-toolsはエラーを返します。ユーザーが確認した場合のみ`--force`を使用してください：

```bash
RESULT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase remove "${target}" --force)
```

CLIが処理する内容：
- フェーズディレクトリの削除
- 後続のすべてのディレクトリの再番号付け（競合を避けるため逆順で）
- 再番号付けされたディレクトリ内のすべてのファイルの名前変更（PLAN.md、SUMMARY.mdなど）
- ROADMAP.mdの更新（セクションの削除、すべてのフェーズ参照の再番号付け、依存関係の更新）
- STATE.mdの更新（フェーズカウントの減少）

結果から抽出: `removed`、`directory_deleted`、`renamed_directories`、`renamed_files`、`roadmap_updated`、`state_updated`。
</step>

<step name="commit">
削除をステージングしてコミットします：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "chore: remove phase {target} ({original-phase-name})" --files .planning/
```

コミットメッセージに削除された内容の履歴記録を保存します。
</step>

<step name="completion">
完了の概要を提示します：

```
フェーズ {target} ({original-name}) を削除しました。

変更内容:
- 削除: .planning/phases/{target}-{slug}/
- 再番号付け: {N} ディレクトリ、{M} ファイル
- 更新: ROADMAP.md、STATE.md
- コミット: chore: remove phase {target} ({original-name})

---

## 次のステップ

以下から選択してください:
- `/gsd:progress` — 更新されたロードマップステータスを確認
- 現在のフェーズを継続
- ロードマップを確認

---
```
</step>

</process>

<anti_patterns>

- 完了済みフェーズ（SUMMARY.mdファイルあり）を--forceなしで削除しない
- 現在または過去のフェーズを削除しない
- 手動で再番号付けしない — すべての再番号付けを処理する`gsd-tools phase remove`を使用する
- STATE.mdに「削除されたフェーズ」のメモを追加しない — gitコミットが記録です
- 完了済みフェーズディレクトリを変更しない
</anti_patterns>

<success_criteria>
フェーズ削除は以下の条件を満たした時に完了です：

- [ ] ターゲットフェーズが将来/未開始として検証された
- [ ] `gsd-tools phase remove`が正常に実行された
- [ ] 説明的なメッセージで変更がコミットされた
- [ ] ユーザーに変更が通知された
</success_criteria>
