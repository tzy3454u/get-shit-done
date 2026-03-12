<purpose>

完了したマイルストーンから蓄積されたフェーズディレクトリを `.planning/milestones/v{X.Y}-phases/` にアーカイブする。各フェーズがどの完了マイルストーンに属するかを識別し、ドライランのサマリーを表示し、確認後にディレクトリを移動する。

</purpose>

<required_reading>

1. `.planning/MILESTONES.md`
2. `.planning/milestones/` ディレクトリ一覧
3. `.planning/phases/` ディレクトリ一覧

</required_reading>

<process>

<step name="identify_completed_milestones">

`.planning/MILESTONES.md` を読み込み、完了したマイルストーンとそのバージョンを識別する。

```bash
cat .planning/MILESTONES.md
```

各マイルストーンバージョン（例：v1.0、v1.1、v2.0）を抽出する。

既存のマイルストーンアーカイブディレクトリを確認する：

```bash
ls -d .planning/milestones/v*-phases 2>/dev/null
```

まだ `-phases` アーカイブディレクトリを持たないマイルストーンにフィルタする。

すべてのマイルストーンが既にフェーズアーカイブを持っている場合：

```
完了したすべてのマイルストーンのフェーズディレクトリは既にアーカイブ済み。クリーンアップの必要はない。
```

ここで停止する。

</step>

<step name="determine_phase_membership">

`-phases` アーカイブを持たない完了マイルストーンごとに、アーカイブされたROADMAPスナップショットを読み込み、どのフェーズが属するかを判定する：

```bash
cat .planning/milestones/v{X.Y}-ROADMAP.md
```

アーカイブされたロードマップからフェーズ番号と名前を抽出する（例：Phase 1: Foundation、Phase 2: Auth）。

`.planning/phases/` にそれらのフェーズディレクトリがまだ存在するか確認する：

```bash
ls -d .planning/phases/*/ 2>/dev/null
```

フェーズディレクトリをマイルストーンのメンバーシップに照合する。`.planning/phases/` にまだ存在するディレクトリのみを含める。

</step>

<step name="show_dry_run">

各マイルストーンのドライランサマリーを表示する：

```
## クリーンアップサマリー

### v{X.Y} — {マイルストーン名}
以下のフェーズディレクトリがアーカイブされる：
- 01-foundation/
- 02-auth/
- 03-core-features/

移動先: .planning/milestones/v{X.Y}-phases/

### v{X.Z} — {マイルストーン名}
以下のフェーズディレクトリがアーカイブされる：
- 04-security/
- 05-hardening/

移動先: .planning/milestones/v{X.Z}-phases/
```

アーカイブするフェーズディレクトリが残っていない場合（すべて移動済みまたは削除済み）：

```
アーカイブするフェーズディレクトリが見つからない。フェーズは以前に削除またはアーカイブされた可能性がある。
```

ここで停止する。

AskUserQuestion: "アーカイブを実行しますか？" オプション："はい — 一覧のフェーズをアーカイブ" | "キャンセル"

"キャンセル" の場合：停止する。

</step>

<step name="archive_phases">

各マイルストーンについて、フェーズディレクトリを移動する：

```bash
mkdir -p .planning/milestones/v{X.Y}-phases
```

このマイルストーンに属する各フェーズディレクトリについて：

```bash
mv .planning/phases/{dir} .planning/milestones/v{X.Y}-phases/
```

クリーンアップセットのすべてのマイルストーンについて繰り返す。

</step>

<step name="commit">

変更をコミットする：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "chore: archive phase directories from completed milestones" --files .planning/milestones/ .planning/phases/
```

</step>

<step name="report">

```
アーカイブ済み:
{各マイルストーンについて}
- v{X.Y}: {N} 個のフェーズディレクトリ → .planning/milestones/v{X.Y}-phases/

.planning/phases/ をクリーンアップしました。
```

</step>

</process>

<success_criteria>

- [ ] 既存のフェーズアーカイブを持たない完了マイルストーンがすべて識別されている
- [ ] アーカイブされたROADMAPスナップショットからフェーズのメンバーシップが判定されている
- [ ] ドライランサマリーが表示され、ユーザーが確認している
- [ ] フェーズディレクトリが `.planning/milestones/v{X.Y}-phases/` に移動されている
- [ ] 変更がコミットされている

</success_criteria>

