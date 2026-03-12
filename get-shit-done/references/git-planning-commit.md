# Gitプランニングコミット

gsd-tools CLIを使用してプランニング成果物をコミットします。`commit_docs`設定とgitignoreステータスを自動的にチェックします。

## CLIでのコミット

`.planning/`ファイルには常に`gsd-tools.cjs commit`を使用してください。`commit_docs`とgitignoreのチェックを自動的に処理します:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs({scope}): {description}" --files .planning/STATE.md .planning/ROADMAP.md
```

CLIは`commit_docs`が`false`または`.planning/`がgitignoreされている場合、`skipped`（理由付き）を返します。手動の条件チェックは不要です。

## 前のコミットへのamend

`.planning/`ファイルの変更を前のコミットに統合する場合:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "" --files .planning/codebase/*.md --amend
```

## コミットメッセージパターン

| コマンド | スコープ | 例 |
|---------|-------|---------|
| plan-phase | phase | `docs(phase-03): create authentication plans` |
| execute-phase | phase | `docs(phase-03): complete authentication phase` |
| new-milestone | milestone | `docs: start milestone v1.1` |
| remove-phase | chore | `chore: remove phase 17 (dashboard)` |
| insert-phase | phase | `docs: insert phase 16.1 (critical fix)` |
| add-phase | phase | `docs: add phase 07 (settings page)` |

## スキップする場合

- configで`commit_docs: false`の場合
- `.planning/`がgitignoreされている場合
- コミットする変更がない場合（`git status --porcelain .planning/`で確認）
