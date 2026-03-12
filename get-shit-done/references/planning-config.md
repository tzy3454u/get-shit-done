<planning_config>

`.planning/`ディレクトリの動作に関する設定オプション。

<config_schema>
```json
"planning": {
  "commit_docs": true,
  "search_gitignored": false
},
"git": {
  "branching_strategy": "none",
  "phase_branch_template": "gsd/phase-{phase}-{slug}",
  "milestone_branch_template": "gsd/{milestone}-{slug}"
}
```

| オプション | デフォルト | 説明 |
|--------|---------|-------------|
| `commit_docs` | `true` | プランニング成果物をgitにコミットするかどうか |
| `search_gitignored` | `false` | 広範なrg検索に`--no-ignore`を追加する |
| `git.branching_strategy` | `"none"` | Gitブランチ戦略: `"none"`, `"phase"`, `"milestone"` |
| `git.phase_branch_template` | `"gsd/phase-{phase}-{slug}"` | phase戦略用のブランチテンプレート |
| `git.milestone_branch_template` | `"gsd/{milestone}-{slug}"` | milestone戦略用のブランチテンプレート |
</config_schema>

<commit_docs_behavior>

**`commit_docs: true`（デフォルト）の場合:**
- プランニングファイルは通常通りコミットされる
- SUMMARY.md, STATE.md, ROADMAP.mdはgitで追跡される
- プランニング決定の完全な履歴が保存される

**`commit_docs: false`の場合:**
- `.planning/`ファイルのすべての`git add`/`git commit`をスキップ
- ユーザーは`.planning/`を`.gitignore`に追加する必要がある
- 用途: OSSへのコントリビューション、クライアントプロジェクト、プランニングのプライベート保持

**gsd-tools.cjsの使用（推奨）:**

```bash
# commit_docs + gitignoreチェックを自動で行うコミット:
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: update state" --files .planning/STATE.md

# state loadでconfigを読み込む（JSONを返す）:
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# commit_docsはJSON出力で利用可能

# またはcommit_docsを含むinitコマンドを使用:
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init execute-phase "1")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# commit_docsはすべてのinitコマンド出力に含まれる
```

**自動検出:** `.planning/`がgitignoreされている場合、config.jsonの設定に関係なく`commit_docs`は自動的に`false`になります。これにより、`.planning/`が`.gitignore`にあるときのgitエラーを防ぎます。

**CLIでのコミット（チェックを自動処理）:**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: update state" --files .planning/STATE.md
```

CLIは`commit_docs`のconfigとgitignoreステータスを内部的にチェックします。手動の条件分岐は不要です。

</commit_docs_behavior>

<search_behavior>

**`search_gitignored: false`（デフォルト）の場合:**
- 標準的なrgの動作（.gitignoreを尊重）
- 直接パス検索は動作する: `rg "pattern" .planning/`でファイルを見つける
- 広範な検索はgitignored対象をスキップ: `rg "pattern"`は`.planning/`をスキップ

**`search_gitignored: true`の場合:**
- `.planning/`を含めるべき広範なrg検索に`--no-ignore`を追加
- リポジトリ全体を検索して`.planning/`のマッチを期待する場合にのみ必要

**注意:** ほとんどのGSD操作は直接ファイル読み取りまたは明示的なパスを使用し、gitignoreのステータスに関係なく動作します。

</search_behavior>

<setup_uncommitted_mode>

コミットしないモードを使用するには:

1. **configを設定:**
   ```json
   "planning": {
     "commit_docs": false,
     "search_gitignored": true
   }
   ```

2. **.gitignoreに追加:**
   ```
   .planning/
   ```

3. **既存の追跡ファイル:** `.planning/`が以前追跡されていた場合:
   ```bash
   git rm -r --cached .planning/
   git commit -m "chore: stop tracking planning docs"
   ```

4. **ブランチマージ:** `branching_strategy: phase`または`milestone`を使用する場合、`complete-milestone`ワークフローは`commit_docs: false`の場合、マージコミット前にステージングから`.planning/`ファイルを自動的に除去します。

</setup_uncommitted_mode>

<branching_strategy_behavior>

**ブランチ戦略:**

| 戦略 | ブランチ作成タイミング | ブランチスコープ | マージポイント |
|----------|---------------------|--------------|-------------|
| `none` | なし | N/A | N/A |
| `phase` | `execute-phase`開始時 | 単一フェーズ | フェーズ後にユーザーがマージ |
| `milestone` | マイルストーンの最初の`execute-phase`時 | マイルストーン全体 | `complete-milestone`時 |

**`git.branching_strategy: "none"`（デフォルト）の場合:**
- すべての作業は現在のブランチにコミット
- 標準的なGSDの動作

**`git.branching_strategy: "phase"`の場合:**
- `execute-phase`が実行前にブランチを作成/切り替え
- ブランチ名は`phase_branch_template`から（例: `gsd/phase-03-authentication`）
- すべてのプランコミットはそのブランチに
- フェーズ完了後にユーザーが手動でブランチをマージ
- `complete-milestone`はすべてのフェーズブランチのマージを提案

**`git.branching_strategy: "milestone"`の場合:**
- マイルストーンの最初の`execute-phase`がマイルストーンブランチを作成
- ブランチ名は`milestone_branch_template`から（例: `gsd/v1.0-mvp`）
- マイルストーン内のすべてのフェーズが同じブランチにコミット
- `complete-milestone`がマイルストーンブランチのmainへのマージを提案

**テンプレート変数:**

| 変数 | 利用可能な場所 | 説明 |
|----------|--------------|-------------|
| `{phase}` | phase_branch_template | ゼロパディングされたフェーズ番号（例: "03"） |
| `{slug}` | 両方 | 小文字、ハイフン区切りの名前 |
| `{milestone}` | milestone_branch_template | マイルストーンバージョン（例: "v1.0"） |

**configの確認:**

`init execute-phase`を使用すると、すべてのconfigがJSONとして返されます:
```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init execute-phase "1")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# JSON出力に含まれる: branching_strategy, phase_branch_template, milestone_branch_template
```

または`state load`でconfig値を取得:
```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# JSONからbranching_strategy, phase_branch_template, milestone_branch_templateを解析
```

**ブランチ作成:**

```bash
# phase戦略の場合
if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  PHASE_SLUG=$(echo "$PHASE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$PHASE_BRANCH_TEMPLATE" | sed "s/{phase}/$PADDED_PHASE/g" | sed "s/{slug}/$PHASE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi

# milestone戦略の場合
if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  MILESTONE_SLUG=$(echo "$MILESTONE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$MILESTONE_BRANCH_TEMPLATE" | sed "s/{milestone}/$MILESTONE_VERSION/g" | sed "s/{slug}/$MILESTONE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi
```

**complete-milestoneでのマージオプション:**

| オプション | Gitコマンド | 結果 |
|--------|-------------|--------|
| スカッシュマージ（推奨） | `git merge --squash` | ブランチごとに1つのクリーンなコミット |
| 履歴付きマージ | `git merge --no-ff` | すべての個別コミットを保存 |
| マージせずに削除 | `git branch -D` | ブランチの作業を破棄 |
| ブランチを保持 | （なし） | 後で手動処理 |

スカッシュマージを推奨 — mainブランチの履歴をクリーンに保ちながら、完全な開発履歴をブランチに保存します（削除するまで）。

**ユースケース:**

| 戦略 | 最適な用途 |
|----------|----------|
| `none` | ソロ開発、シンプルなプロジェクト |
| `phase` | フェーズごとのコードレビュー、細粒度のロールバック、チームコラボレーション |
| `milestone` | リリースブランチ、ステージング環境、バージョンごとのPR |

</branching_strategy_behavior>

</planning_config>