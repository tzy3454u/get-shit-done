---
type: prompt
name: gsd:complete-milestone
description: 完了したマイルストーンをアーカイブし、次のバージョンを準備する
argument-hint: <version>
allowed-tools:
  - Read
  - Write
  - Bash
---

<objective>
マイルストーン {{version}} を完了としてマークし、milestones/ にアーカイブし、ROADMAP.md と REQUIREMENTS.md を更新します。

目的: リリース済みバージョンの履歴記録を作成し、マイルストーン成果物（ロードマップ＋要件）をアーカイブし、次のマイルストーンを準備します。
出力: マイルストーンアーカイブ（ロードマップ＋要件）、PROJECT.md の更新、git タグの作成。
</objective>

<execution_context>
**以下のファイルを今すぐ読み込んでください（作業開始前に）:**

- @~/.claude/get-shit-done/workflows/complete-milestone.md （メインワークフロー）
- @~/.claude/get-shit-done/templates/milestone-archive.md （アーカイブテンプレート）
  </execution_context>

<context>
**プロジェクトファイル:**
- `.planning/ROADMAP.md`
- `.planning/REQUIREMENTS.md`
- `.planning/STATE.md`
- `.planning/PROJECT.md`

**ユーザー入力:**

- バージョン: {{version}}（例: "1.0", "1.1", "2.0"）
  </context>

<process>

**complete-milestone.md ワークフローに従う:**

0. **監査の確認:**

   - `.planning/v{{version}}-MILESTONE-AUDIT.md` を確認する
   - 存在しないか古い場合: まず `/gsd:audit-milestone` の実行を推奨
   - 監査ステータスが `gaps_found` の場合: まず `/gsd:plan-milestone-gaps` の実行を推奨
   - 監査ステータスが `passed` の場合: ステップ1に進む

   ```markdown
   ## 事前チェック

   {v{{version}}-MILESTONE-AUDIT.md が存在しない場合:}
   ⚠ マイルストーン監査が見つかりません。まず `/gsd:audit-milestone` を実行して
   要件カバレッジ、フェーズ間統合、E2Eフローを検証してください。

   {監査にギャップがある場合:}
   ⚠ マイルストーン監査でギャップが見つかりました。`/gsd:plan-milestone-gaps` を実行して
   ギャップを埋めるフェーズを作成するか、技術的負債として受け入れて続行してください。

   {監査が合格の場合:}
   ✓ マイルストーン監査に合格しました。完了処理を続行します。
   ```

1. **準備状況の確認:**

   - マイルストーン内のすべてのフェーズに完了した計画（SUMMARY.md が存在）があることを確認
   - マイルストーンのスコープと統計を提示
   - 確認を待つ

2. **統計の収集:**

   - フェーズ数、計画数、タスク数をカウント
   - git の範囲、ファイル変更数、LOC を計算
   - git ログからタイムラインを抽出
   - サマリーを提示し、確認

3. **達成事項の抽出:**

   - マイルストーン範囲内のすべてのフェーズの SUMMARY.md ファイルを読み込む
   - 4〜6個の主要な達成事項を抽出
   - 承認を求める

4. **マイルストーンのアーカイブ:**

   - `.planning/milestones/v{{version}}-ROADMAP.md` を作成
   - ROADMAP.md からフェーズの詳細を完全に抽出
   - milestone-archive.md テンプレートを適用
   - ROADMAP.md を1行のサマリーとリンクに更新

5. **要件のアーカイブ:**

   - `.planning/milestones/v{{version}}-REQUIREMENTS.md` を作成
   - すべての v1 要件を完了としてマーク（チェックボックスをオン）
   - 要件の結果を記録（検証済み、調整済み、取り下げ）
   - `.planning/REQUIREMENTS.md` を削除（次のマイルストーンで新規作成）

6. **PROJECT.md の更新:**

   - リリース済みバージョンの「現在の状態」セクションを追加
   - 「次のマイルストーン目標」セクションを追加
   - 以前のコンテンツを `<details>` にアーカイブ（v1.1以降の場合）

7. **コミットとタグ:**

   - ステージ: MILESTONES.md, PROJECT.md, ROADMAP.md, STATE.md, アーカイブファイル
   - コミット: `chore: archive v{{version}} milestone`
   - タグ: `git tag -a v{{version}} -m "[milestone summary]"`
   - タグのプッシュについて確認

8. **次のステップの提案:**
   - `/gsd:new-milestone` — 次のマイルストーンを開始（質問 → 調査 → 要件 → ロードマップ）

</process>

<success_criteria>

- マイルストーンが `.planning/milestones/v{{version}}-ROADMAP.md` にアーカイブされている
- 要件が `.planning/milestones/v{{version}}-REQUIREMENTS.md` にアーカイブされている
- `.planning/REQUIREMENTS.md` が削除されている（次のマイルストーン用にクリーン）
- ROADMAP.md が1行のエントリに圧縮されている
- PROJECT.md が現在の状態で更新されている
- git タグ v{{version}} が作成されている
- コミットが成功している
- ユーザーが次のステップを把握している（新しい要件の必要性を含む）
  </success_criteria>

<critical_rules>

- **ワークフローを先に読み込む:** 実行前に complete-milestone.md を読み込む
- **完了の確認:** すべてのフェーズに SUMMARY.md ファイルが存在する必要がある
- **ユーザー確認:** 検証ゲートで承認を待つ
- **削除前にアーカイブ:** オリジナルの更新/削除前に必ずアーカイブファイルを作成する
- **1行サマリー:** ROADMAP.md 内の圧縮されたマイルストーンはリンク付きの1行にする
- **コンテキスト効率:** アーカイブにより ROADMAP.md と REQUIREMENTS.md のサイズをマイルストーンごとに一定に保つ
- **新しい要件:** 次のマイルストーンは `/gsd:new-milestone` で開始し、要件定義を含む
  </critical_rules>
