<purpose>
下流のエージェントが必要とする実装上の決定を抽出する。フェーズを分析してグレーエリアを特定し、ユーザーに何をディスカッションするか選択させ、選択された各エリアについて満足するまで深掘りする。

あなたはインタビュアーではなく思考パートナーである。ユーザーはビジョナリーであり、あなたはビルダーである。あなたの仕事はリサーチと計画を導く決定を記録することであり、実装を自分で考え出すことではない。
</purpose>

<downstream_awareness>
**CONTEXT.mdが供給するもの：**

1. **gsd-phase-researcher** — 何をリサーチすべきか知るためにCONTEXT.mdを読む
   - 「ユーザーはカードベースのレイアウトを望む」→ リサーチャーがカードコンポーネントパターンを調査
   - 「無限スクロールに決定」→ リサーチャーが仮想化ライブラリを調査

2. **gsd-planner** — どの決定がロックされているか知るためにCONTEXT.mdを読む
   - 「モバイルでプルトゥリフレッシュ」→ プランナーがタスク仕様に含める
   - 「Claude's Discretion：ローディングスケルトン」→ プランナーがアプローチを決定可能

**あなたの仕事：** 下流エージェントがユーザーに再度聞くことなく行動できるほど明確に決定を記録する。

**あなたの仕事ではないこと：** 実装方法を考える。それはリサーチと計画が記録された決定を使って行う。
</downstream_awareness>

<philosophy>
**ユーザー = 創設者/ビジョナリー。Claude = ビルダー。**

ユーザーが知っていること：
- どのように動作するか想像しているか
- どのような見た目/感触にすべきか
- 何が必須で何があれば良いか
- 特定の動作や参考にしているもの

ユーザーに聞くべきでないこと（そして聞かれるべきでないこと）：
- コードベースパターン（リサーチャーがコードを読む）
- 技術的リスク（リサーチャーがこれらを特定する）
- 実装アプローチ（プランナーがこれを考える）
- 成功指標（作業から推論される）

ビジョンと実装上の選択について質問する。下流エージェント向けに決定を記録する。
</philosophy>

<scope_guardrail>
**重要：スコープクリープなし。**

フェーズの境界はROADMAP.mdからのものであり、固定されている。ディスカッションはスコープ内のものをどのように実装するかを明確にするものであり、新しい機能を追加するかどうかではない。

**許可（曖昧さの明確化）：**
- 「投稿はどのように表示すべきですか？」（レイアウト、密度、表示情報）
- 「空の状態ではどうなりますか？」（機能の範囲内）
- 「プルトゥリフレッシュか手動か？」（動作の選択）

**不許可（スコープクリープ）：**
- 「コメントも追加すべきですか？」（新しい機能）
- 「検索/フィルタリングはどうですか？」（新しい機能）
- 「ブックマークを含めた方がいいかも？」（新しい機能）

**ヒューリスティック：** これはフェーズに既にあるものの実装方法を明確にしているか、それとも独自のフェーズになり得る新しい機能を追加しているか？

**ユーザーがスコープクリープを提案した場合：**
```
「[Feature X]は新しい機能 — 独自のフェーズになる。
ロードマップのバックログに記録しておきますか？

今は[phase domain]に集中しましょう。」
```

「Deferred Ideas」セクションにアイデアを記録する。失わないこと、行動もしないこと。
</scope_guardrail>

<gray_area_identification>
グレーエリアとは**ユーザーが気にする実装上の決定** — 複数の方向に進む可能性があり、結果が変わるもの。

**グレーエリアの特定方法：**

1. ROADMAP.mdから**フェーズの目標を読む**
2. **ドメインを理解する** — どんな種類のものを構築しているか？
   - ユーザーが見るもの → ビジュアルプレゼンテーション、インタラクション、状態が重要
   - ユーザーが呼び出すもの → インターフェース契約、レスポンス、エラーが重要
   - ユーザーが実行するもの → 呼び出し方、出力、動作モードが重要
   - ユーザーが読むもの → 構造、トーン、深さ、フローが重要
   - 整理されるもの → 基準、グループ化、例外処理が重要
3. **フェーズ固有のグレーエリアを生成** — 一般的なカテゴリではなく、このフェーズの具体的な決定

**一般的なカテゴリラベルは使わない**（UI、UX、Behavior）。具体的なグレーエリアを生成：

```
フェーズ：「ユーザー認証」
→ セッション処理、エラーレスポンス、マルチデバイスポリシー、リカバリーフロー

フェーズ：「写真ライブラリの整理」
→ グループ化基準、重複処理、命名規約、フォルダ構造

フェーズ：「データベースバックアップ用CLI」
→ 出力形式、フラグ設計、進捗報告、エラーリカバリー

フェーズ：「APIドキュメント」
→ 構造/ナビゲーション、コード例の深さ、バージョニングアプローチ、インタラクティブ要素
```

**重要な問い：** ユーザーが意見を述べるべき、結果を変える決定は何か？

**Claudeが処理するもの（聞かないこと）：**
- 技術的な実装の詳細
- アーキテクチャパターン
- パフォーマンス最適化
- スコープ（ロードマップがこれを定義する）
</gray_area_identification>

<process>

**エクスプレスパス利用可能：** PRDまたは受け入れ基準ドキュメントが既にある場合、`/gsd:plan-phase {phase} --prd path/to/prd.md`を使用してこのディスカッションをスキップし、直接計画に進むことができる。

<step name="initialize" priority="first">
引数からフェーズ番号（必須）。

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

JSONをパース：`commit_docs`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_research`, `has_context`, `has_plans`, `has_verification`, `plan_count`, `roadmap_exists`, `planning_exists`。

**`phase_found`がfalseの場合：**
```
フェーズ[X]がロードマップに見つかりません。

利用可能なフェーズを確認するには/gsd:progressを使用すること。
```
ワークフローを終了。

**`phase_found`がtrueの場合：** check_existingに続行。
</step>

<step name="check_existing">
initから`has_context`を使用してCONTEXT.mdが既に存在するか確認。

```bash
ls ${phase_dir}/*-CONTEXT.md 2>/dev/null
```

**存在する場合：**
AskUserQuestionを使用：
- header: "Context"
- question: "フェーズ[X]には既にコンテキストがあります。どうしますか？"
- options:
  - "Update it" — 既存のコンテキストをレビューして修正
  - "View it" — 内容を表示
  - "Skip" — 既存のコンテキストをそのまま使用

"Update"の場合：既存を読み込み、analyze_phaseに続行
"View"の場合：CONTEXT.mdを表示し、その後update/skipを提案
"Skip"の場合：ワークフローを終了

**存在しない場合：**

initから`has_plans`と`plan_count`を確認。**`has_plans`がtrueの場合：**

AskUserQuestionを使用：
- header: "Plans exist"
- question: "フェーズ[X]にはユーザーコンテキストなしで作成された{plan_count}個のプランが既にあります。ここでのあなたの決定は再計画しない限り既存のプランに影響しません。"
- options:
  - "Continue and replan after" — コンテキストを記録し、その後/gsd:plan-phase {X}で再計画
  - "View existing plans" — 決定前にプランを表示
  - "Cancel" — discuss-phaseをスキップ

"Continue and replan after"の場合：analyze_phaseに続行。
"View existing plans"の場合：プランファイルを表示し、その後「Continue」/「Cancel」を提案。
"Cancel"の場合：ワークフローを終了。

**`has_plans`がfalseの場合：** load_prior_contextに続行。
</step>

<step name="load_prior_context">
既に決定された質問を再度聞かず、一貫性を維持するために、プロジェクトレベルと以前のフェーズのコンテキストを読む。

**ステップ1：プロジェクトレベルのファイルを読む**
```bash
# コアプロジェクトファイル
cat .planning/PROJECT.md 2>/dev/null
cat .planning/REQUIREMENTS.md 2>/dev/null
cat .planning/STATE.md 2>/dev/null
```

これらから抽出：
- **PROJECT.md** — ビジョン、原則、譲れないもの、ユーザーの好み
- **REQUIREMENTS.md** — 受け入れ基準、制約、必須 vs あれば良い
- **STATE.md** — 現在の進捗、フラグやセッションメモ

**ステップ2：以前のすべてのCONTEXT.mdファイルを読む**
```bash
# 現在より前のフェーズからすべてのCONTEXT.mdファイルを見つける
find .planning/phases -name "*-CONTEXT.md" 2>/dev/null | sort
```

現在のフェーズより前のフェーズ番号の各CONTEXT.mdについて：
- `<decisions>`セクションを読む — これらはロックされた好み
- `<specifics>`を読む — 特定の参照や「Xのようにしたい」という瞬間
- パターンを記録（例：「ユーザーは一貫してミニマルなUIを好む」、「ユーザーは単一キーショートカットを拒否した」）

**ステップ3：内部`<prior_decisions>`コンテキストを構築**

抽出した情報を構造化：
```
<prior_decisions>
## プロジェクトレベル
- [PROJECT.mdからのキーとなる原則や制約]
- [REQUIREMENTS.mdからのこのフェーズに影響する要件]

## 以前のフェーズから
### Phase N: [Name]
- [現在のフェーズに関連する可能性のある決定]
- [パターンを確立する好み]

### Phase M: [Name]
- [別の関連する決定]
</prior_decisions>
```

**後続ステップでの使用：**
- `analyze_phase`: 以前のフェーズで既に決定されたグレーエリアをスキップ
- `present_gray_areas`: 以前の決定でオプションに注釈を付ける（「Phase 5でXを選択しました」）
- `discuss_areas`: 回答を事前入力するか、矛盾にフラグを立てる（「これはPhase 3と矛盾します — ここでも同じですか、それとも違いますか？」）

**以前のコンテキストが存在しない場合：** なしで続行 — 初期フェーズでは想定される。
</step>

<step name="scout_codebase">
グレーエリアの特定とディスカッションに情報を提供するための、既存コードの軽量スキャン。約10%のコンテキストを使用 — インタラクティブセッションでは許容範囲。

**ステップ1：既存のコードベースマップを確認**
```bash
ls .planning/codebase/*.md 2>/dev/null
```

**コードベースマップが存在する場合：** 最も関連性の高いもの（フェーズタイプに基づいてCONVENTIONS.md、STRUCTURE.md、STACK.md）を読む。抽出：
- 再利用可能なコンポーネント/フック/ユーティリティ
- 確立されたパターン（状態管理、スタイリング、データフェッチ）
- 統合ポイント（新しいコードが接続する場所）

以下のステップ3にスキップ。

**ステップ2：コードベースマップがない場合、ターゲットgrepを実行**

フェーズ目標からキーワードを抽出（例：「feed」→「post」「card」「list」、「auth」→「login」「session」「token」）。

```bash
# フェーズ目標の用語に関連するファイルを検索
grep -rl "{term1}\|{term2}" src/ app/ --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" 2>/dev/null | head -10

# 既存のコンポーネント/フックを検索
ls src/components/ 2>/dev/null
ls src/hooks/ 2>/dev/null
ls src/lib/ src/utils/ 2>/dev/null
```

既存のパターンを理解するために最も関連性の高い3-5ファイルを読む。

**ステップ3：内部codebase_contextを構築**

スキャンから以下を特定：
- **再利用可能なアセット** — このフェーズで使用可能な既存のコンポーネント、フック、ユーティリティ
- **確立されたパターン** — コードベースが状態管理、スタイリング、データフェッチをどのように行っているか
- **統合ポイント** — 新しいコードが既存システムに接続する場所（ルート、ナビゲーション、プロバイダー）
- **クリエイティブオプション** — 既存のアーキテクチャが可能にする、または制約するアプローチ

analyze_phaseとpresent_gray_areasで使用するための内部`<codebase_context>`として保存。これはファイルに書き込まれない — このセッション内でのみ使用。
</step>

<step name="analyze_phase">
ディスカッションに値するグレーエリアを特定するためにフェーズを分析する。**分析を基盤づけるために`prior_decisions`と`codebase_context`の両方を使用する。**

**ROADMAP.mdからフェーズの説明を読み、以下を決定：**

1. **ドメイン境界** — このフェーズはどのような機能を提供するか？明確に述べる。

2. **以前の決定を確認** — グレーエリアを生成する前に、既に決定されたものがないか確認：
   - `<prior_decisions>`を関連する選択についてスキャン（例：「Ctrl+Cのみ、単一キーショートカットなし」）
   - これらは**事前回答済み** — このフェーズで矛盾する必要がない限り再度聞かない
   - プレゼンテーションで使用するために適用される以前の決定を記録

2b. **正規参照アキュムレーターの初期化** — CONTEXT.mdの`<canonical_refs>`リストの構築を開始する。これはこのステップだけでなく、ディスカッション全体を通じて蓄積される。

   **ソース1（今）:** このフェーズのROADMAP.mdから`Canonical refs:`をコピーし、完全な相対パスに展開する。
   **ソース2（今）:** REQUIREMENTS.mdとPROJECT.mdでこのフェーズに参照されている仕様/ADRを確認する。
   **ソース3（scout_codebase）:** 既存コードがドキュメントを参照している場合（例：ADRを引用するコメント）、追加する。
   **ソース4（discuss_areas）:** ディスカッション中にユーザーが「Xを読んで」「Yを確認して」と言ったり、ドキュメント/仕様/ADRを参照した場合 — 即座に追加する。これらはユーザーが下流エージェントに従ってほしいと明示したドキュメントであるため、最も重要な参照であることが多い。

   このリストはCONTEXT.mdで**必須**。各参照には完全な相対パスが必要で、下流エージェントが直接読み込める必要がある。外部ドキュメントが存在しない場合、明示的にその旨を記載する。

3. **カテゴリ別のグレーエリア** — 各関連カテゴリ（UI、UX、動作、空の状態、コンテンツ）について、実装を変える1-2の具体的な曖昧さを特定する。**関連する場合はコードコンテキストで注釈を付ける**（例：「既にCardコンポーネントがある」または「このパターンの既存パターンなし」）。

4. **スキップ評価** — 意味のあるグレーエリアが存在しない場合（純粋なインフラストラクチャ、明確な実装、またはすべて以前のフェーズで既に決定済み）、このフェーズはディスカッションが不要な可能性がある。

**分析を内部的に出力し、その後ユーザーに提示する。**

「Post Feed」フェーズの分析例（コードと以前のコンテキスト付き）：
```
ドメイン：フォロー中のユーザーからの投稿を表示
既存：Cardコンポーネント（src/components/ui/Card.tsx）、useInfiniteQueryフック、Tailwind CSS
以前の決定：「ミニマルなUI優先」（Phase 2）、「ページネーションなし — 常に無限スクロール」（Phase 4）
グレーエリア：
- UI：レイアウトスタイル（カード vs タイムライン vs グリッド）— Cardコンポーネントがshadow/roundedバリアント付きで存在
- UI：情報密度（完全な投稿 vs プレビュー）— 既存の密度パターンなし
- 動作：ローディングパターン — 既に決定済み：無限スクロール（Phase 4）
- 空の状態：投稿がない場合の表示 — EmptyStateコンポーネントがui/に存在
- コンテンツ：表示するメタデータ（時間、著者、リアクション数）
```
</step>

<step name="present_gray_areas">
ドメイン境界、以前の決定、グレーエリアをユーザーに提示する。

**まず、境界と適用される以前の決定を述べる：**
```
Phase [X]: [Name]
ドメイン: [このフェーズが提供するもの — 分析から]

これをどのように実装するか明確にする。
（新しい機能は他のフェーズに属する。）

[以前の決定が適用される場合：]
**以前のフェーズからの引き継ぎ：**
- [ここに適用されるPhase Nからの決定]
- [ここに適用されるPhase Mからの決定]
```

**その後AskUserQuestion（multiSelect: true）を使用：**
- header: "Discuss"
- question: "[phase name]でどのエリアについてディスカッションしたいですか？"
- options: 3-4のフェーズ固有のグレーエリアを生成、各々：
  - "[具体的なエリア]" (label) — 具体的で一般的でない
  - [これがカバーする1-2の質問 + コードコンテキスト注釈] (description)
  - **推奨する選択肢を簡単な理由とともにハイライト**

**以前の決定の注釈：** グレーエリアが以前のフェーズで既に決定されている場合、注釈を付ける：
```
☐ 終了ショートカット — ユーザーはどのように終了すべきか？
  （Phase 5で「Ctrl+Cのみ、単一キーショートカットなし」と決定しました — 見直しますか、維持しますか？）
```

**コードコンテキスト注釈：** スカウトが関連する既存コードを見つけた場合、グレーエリアの説明に注釈を付ける：
```
☐ レイアウトスタイル — カード vs リスト vs タイムライン？
  （shadow/roundedバリアント付きのCardコンポーネントが既にある。再利用するとアプリの一貫性が保たれる。）
```

**両方を組み合わせる場合：** 以前の決定とコードコンテキストの両方が適用される場合：
```
☐ ローディング動作 — 無限スクロールかページネーションか？
  （Phase 4で無限スクロールを選択しました。useInfiniteQueryフックが既に設定済み。）
```

**「スキップ」や「お任せ」のオプションを含めない。** ユーザーはディスカッションするためにこのコマンドを実行した — 本当の選択肢を与える。

**ドメイン別の例（コードコンテキスト付き）：**

「Post Feed」の場合（ビジュアル機能）：
```
☐ レイアウトスタイル — カード vs リスト vs タイムライン？（バリアント付きCardコンポーネントが存在）
☐ ローディング動作 — 無限スクロールかページネーションか？（useInfiniteQueryフックが利用可能）
☐ コンテンツの順序 — 時系列、アルゴリズム、ユーザー選択？
☐ 投稿メタデータ — 投稿ごとの情報は？タイムスタンプ、リアクション、著者？
```

「Database backup CLI」の場合（コマンドラインツール）：
```
☐ 出力形式 — JSON、テーブル、プレーンテキスト？冗長レベルは？
☐ フラグ設計 — ショートフラグ、ロングフラグ、両方？必須 vs 任意？
☐ 進捗報告 — サイレント、プログレスバー、詳細ログ？
☐ エラーリカバリー — 即座に失敗、リトライ、アクションを求める？
```

「Organize photo library」の場合（整理タスク）：
```
☐ グループ化基準 — 日付、場所、顔、イベント？
☐ 重複処理 — ベストを保持、全て保持、毎回確認？
☐ 命名規約 — 元の名前、日付、説明的？
☐ フォルダ構造 — フラット、年別ネスト、カテゴリ別？
```

選択されたエリアでdiscuss_areasに続行。
</step>

<step name="discuss_areas">
選択された各エリアについて、焦点を絞ったディスカッションループを実施する。

**バッチモードサポート:** `$ARGUMENTS`からオプションの`--batch`をパース。
- `--batch`、`--batch=N`、`--batch N`を受け付ける
- 数値が指定されない場合、バッチごとにデフォルト4つの質問
- 明示的なサイズは2-5にクランプし、バッチが回答可能な範囲に収める
- `--batch`がない場合、既存の一問一答フローを維持

**フィロソフィー:** 適応的に、ただしユーザーにペースの選択を委ねる。
- デフォルトモード: 4つの一問一答ターン、その後続行するか次に進むか確認
- `--batch`モード: 2-5の番号付き質問を1つのグループターンで、その後続行するか次に進むか確認

各回答（またはバッチモードでの回答セット）が次の質問や次のバッチを明らかにすべき。

**各エリアについて：**

1. **エリアを告知：**
   ```
   [Area]について話しましょう。
   ```

2. **選択したペースで質問する：**

   **デフォルト（`--batch`なし）: AskUserQuestionで4つの質問**
   - header: "[Area]" (最大12文字 — 必要に応じて省略)
   - question: このエリアの具体的な決定
   - options: 2-3の具体的な選択肢（AskUserQuestionは自動的に「Other」を追加）、推奨する選択肢をハイライトし簡単に理由を説明
   - **関連する場合はオプションにコードコンテキストで注釈を付ける：**
     ```
     「投稿はどのように表示すべきですか？」
     - カード（既存のCardコンポーネントを再利用 — メッセージと一貫）
     - リスト（よりシンプル、新しいパターンになる）
     - タイムライン（新しいTimelineコンポーネントが必要 — まだ存在しない）
     ```
   - 合理的な場合は「You decide」をオプションとして含める — Claude discretionを記録
   - **ライブラリ選択のためのContext7：** グレーエリアがライブラリの選択（例：「マジックリンク」→ next-authドキュメントを照会）やAPIアプローチの決定を含む場合、`mcp__context7__*`ツールを使用して最新のドキュメントを取得しオプションに情報を提供する。すべての質問にContext7を使用しない — ライブラリ固有の知識がオプションを改善する場合のみ。

   **バッチモード（`--batch`）: 1つのプレーンテキストターンで2-5の番号付き質問**
   - 現在のエリアの密接に関連する質問を単一のメッセージにグループ化
   - 各質問を1回の返信で回答できる具体的なものに
   - オプションが有用な場合、各質問にAskUserQuestionを使う代わりに短いインライン選択肢を含める
   - ユーザーが返信後、記録された決定を反映し、未回答の項目を記録し、先に進むために必要な最小限のフォローアップのみ行う
   - バッチ間の適応性を維持：回答セット全体を使って次のバッチを決定するか、エリアが十分に明確かどうかを判断

3. **質問セット後のチェック：**
   - header: "[Area]" (最大12文字)
   - question: "[area]についてさらに質問しますか、次に進みますか？"
   - options: "More questions" / "Next area"

   "More questions"の場合 → `--batch`が有効な場合はさらに2-5の質問バッチ、そうでなければさらに4つの一問一答、その後再度チェック
   "Next area"の場合 → 次の選択されたエリアに進む
   "Other"（フリーテキスト）の場合 → インテントを解釈：継続フレーズ（「もっと話す」「続けて」「はい」「もっと」）は"More questions"にマッピング、進行フレーズ（「完了」「次へ」「スキップ」）は"Next area"にマッピング。曖昧な場合は質問：「[area]についてさらに質問を続けますか、次のエリアに進みますか？」

4. **最初に選択されたすべてのエリアが完了した後：**
   - ディスカッションで記録された内容を要約
   - AskUserQuestion:
     - header: "Done"
     - question: "[エリアのリスト]についてディスカッションしました。どのグレーエリアがまだ不明確ですか？"
     - options: "Explore more gray areas" / "I'm ready for context"
   - "Explore more gray areas"の場合：
     - 学んだことに基づいて2-4の追加グレーエリアを特定
     - これらの新しいエリアでpresent_gray_areasロジックに戻る
     - ループ：新しいエリアをディスカッションし、再度質問
   - "I'm ready for context"の場合：write_contextに進む

**ディスカッション中の正規参照の蓄積:**
ユーザーが回答中にドキュメント、仕様、ADRを参照した場合 — 例：「adr-014を読んで」「MCP仕様を確認して」「browse-spec.mdに従って」— 即座に：
1. 参照されたドキュメントを読む（または存在を確認）
2. 完全な相対パスで正規参照アキュムレーターに追加
3. ドキュメントから学んだことを後続の質問に反映

これらのユーザー参照ドキュメントは、ユーザーが下流エージェントに従ってほしいと明示したドキュメントであるため、ROADMAP.mdの参照よりも重要であることが多い。決して削除しないこと。

**質問の設計：**
- オプションは具体的であるべき、抽象的ではない（「カード」、「Option A」ではない）
- 各回答が次の質問や次のバッチに情報を提供すべき
- ユーザーが「Other」を選んでフリーフォーム入力を提供する場合（例：「説明させて」「他のもの」、またはオープンエンドの回答）、フォローアップを別のAskUserQuestionではなくプレーンテキストで質問する。通常のプロンプトで入力するのを待ち、その入力を反映して確認してから、次の質問のAskUserQuestionや次の番号付きバッチを再開する。

**スコープクリープの処理：**
ユーザーがフェーズドメイン外のものに言及した場合：
```
「[Feature]は新しい機能のようだ — 独自のフェーズに属する。
延期アイデアとして記録しておく。

[current area]に戻ります：[現在の質問に戻る]」
```

延期アイデアを内部的に追跡。
</step>

<step name="write_context">
行われた決定を記録するCONTEXT.mdを作成する。

**フェーズディレクトリを見つけるか作成：**

initからの値を使用：`phase_dir`, `phase_slug`, `padded_phase`。

`phase_dir`がnullの場合（フェーズがロードマップに存在するがディレクトリがない）：
```bash
mkdir -p ".planning/phases/${padded_phase}-${phase_slug}"
```

**ファイルの場所：** `${phase_dir}/${padded_phase}-CONTEXT.md`

**ディスカッションされた内容でコンテンツを構造化：**

```markdown
# Phase [X]: [Name] - Context

**Gathered:** [date]
**Status:** Ready for planning

<domain>
## Phase Boundary

[このフェーズが提供するものの明確な記述 — スコープアンカー]

</domain>

<decisions>
## Implementation Decisions

### [ディスカッションされたカテゴリ1]
- [記録された決定または好み]
- [該当する場合は別の決定]

### [ディスカッションされたカテゴリ2]
- [記録された決定または好み]

### Claude's Discretion
[ユーザーが「お任せ」と言ったエリア — Claudeがここで柔軟に対応できることを記載]

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

[MANDATORY section. Write the FULL accumulated canonical refs list here.
Sources: ROADMAP.md refs + REQUIREMENTS.md refs + user-referenced docs during
discussion + any docs discovered during codebase scout. Group by topic area.
Every entry needs a full relative path — not just a name.]

### [Topic area 1]
- `path/to/adr-or-spec.md` — [What it decides/defines that's relevant]
- `path/to/doc.md` §N — [Specific section reference]

### [Topic area 2]
- `path/to/feature-doc.md` — [What this doc defines]

[If no external specs: "No external specs — requirements fully captured in decisions above"]

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- [コンポーネント/フック/ユーティリティ]: [このフェーズでの使用方法]

### Established Patterns
- [パターン]: [このフェーズをどのように制約/可能にするか]

### Integration Points
- [新しいコードが既存システムに接続する場所]

</code_context>

<specifics>
## Specific Ideas

[ディスカッションからの特定の参照、例、「Xのようにしたい」という瞬間]

[該当なしの場合: "No specific requirements — open to standard approaches"]

</specifics>

<deferred>
## Deferred Ideas

[出てきたが他のフェーズに属するアイデア。失わないこと。]

[該当なしの場合: "None — discussion stayed within phase scope"]

</deferred>

---

*Phase: XX-name*
*Context gathered: [date]*
```

ファイルを書き込む。
</step>

<step name="confirm_creation">
サマリーと次のステップを提示：

```
作成完了: .planning/phases/${PADDED_PHASE}-${SLUG}/${PADDED_PHASE}-CONTEXT.md

## 記録された決定

### [Category]
- [主要な決定]

### [Category]
- [主要な決定]

[延期アイデアがある場合：]
## 後で記録
- [延期アイデア] — 将来のフェーズ

---

## ▶ Next Up

**Phase ${PHASE}: [Name]** — [ROADMAP.mdからの目標]

`/gsd:plan-phase ${PHASE}`

<sub>`/clear` first → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション：**
- `/gsd:plan-phase ${PHASE} --skip-research` — リサーチなしで計画
- 続行前にCONTEXT.mdをレビュー/編集

---
```
</step>

<step name="git_commit">
フェーズコンテキストをコミット（内部的にinitからの`commit_docs`を使用）：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): capture phase context" --files "${phase_dir}/${padded_phase}-CONTEXT.md"
```

確認：「Committed: docs(${padded_phase}): capture phase context」
</step>

<step name="update_state">
STATE.mdをセッション情報で更新：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state record-session \
  --stopped-at "Phase ${PHASE} context gathered" \
  --resume-file "${phase_dir}/${padded_phase}-CONTEXT.md"
```

STATE.mdをコミット：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(state): record phase ${PHASE} context session" --files .planning/STATE.md
```
</step>

<step name="auto_advance">
自動進行トリガーを確認：

1. $ARGUMENTSから`--auto`フラグをパース
2. **チェーンフラグをインテントと同期** — ユーザーが手動で呼び出した場合（`--auto`なし）、以前の中断された`--auto`チェーンからのエフェメラルチェーンフラグをクリアする。これは`workflow.auto_advance`（ユーザーの永続設定）には触れない：
   ```bash
   if [[ ! "$ARGUMENTS" =~ --auto ]]; then
     node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false 2>/dev/null
   fi
   ```
3. チェーンフラグとユーザー設定の両方を読む：
   ```bash
   AUTO_CHAIN=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
   AUTO_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**`--auto`フラグが存在するが`AUTO_CHAIN`がtrueでない場合：** チェーンフラグを設定に永続化（new-projectなしでの直接`--auto`使用を処理）：
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active true
```

**`--auto`フラグが存在する場合 または `AUTO_CHAIN`がtrue または `AUTO_CFG`がtrueの場合：**

バナーを表示：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTO-ADVANCING TO PLAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

コンテキスト記録完了。plan-phaseを起動中...
```

ネストされたTaskセッション（深いエージェントネスティングによるランタイムフリーズを引き起こす — #686参照）を避けるため、Skillツールを使用してplan-phaseを起動：
```
Skill(skill="gsd:plan-phase", args="${PHASE} --auto")
```

これにより自動進行チェーンがフラット化される — discuss、plan、executeがすべて同じネスティングレベルで実行され、深いTaskエージェントを生成しない。

**plan-phaseの戻り値を処理：**
- **PHASE COMPLETE** → フルチェーン成功。表示：
  ```
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   GSD ► PHASE ${PHASE} COMPLETE
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  自動進行パイプライン完了: discuss → plan → execute

  次: /gsd:discuss-phase ${NEXT_PHASE} --auto
  <sub>/clear first → 新しいコンテキストウィンドウ</sub>
  ```
- **PLANNING COMPLETE** → 計画完了、実行が完了していない：
  ```
  自動進行部分完了：計画完了、実行は完了していません。
  続行: /gsd:execute-phase ${PHASE}
  ```
- **PLANNING INCONCLUSIVE / CHECKPOINT** → チェーンを停止：
  ```
  自動進行停止：計画に入力が必要。
  続行: /gsd:plan-phase ${PHASE}
  ```
- **GAPS FOUND** → チェーンを停止：
  ```
  自動進行停止：実行中にギャップが見つかりました。
  続行: /gsd:plan-phase ${PHASE} --gaps
  ```

**`--auto`も設定も有効でない場合：**
`confirm_creation`ステップにルーティング（既存の動作 — 手動の次ステップを表示）。
</step>

</process>

<success_criteria>
- フェーズがロードマップに対して検証された
- 以前のコンテキストが読み込まれた（PROJECT.md、REQUIREMENTS.md、STATE.md、以前のCONTEXT.mdファイル）
- 既に決定された質問が再度聞かれなかった（以前のフェーズから引き継ぎ）
- 再利用可能なアセット、パターン、統合ポイントのためにコードベースがスカウトされた
- コードと以前の決定の注釈付きでインテリジェントな分析を通じてグレーエリアが特定された
- ユーザーがどのエリアをディスカッションするか選択した
- 選択された各エリアがユーザーが満足するまで探索された（コード情報と以前の決定情報に基づくオプション付き）
- スコープクリープが延期アイデアにリダイレクトされた
- CONTEXT.mdが曖昧なビジョンではなく実際の決定を記録している
- CONTEXT.mdに下流エージェントが必要とするすべての仕様/ADR/ドキュメントへの完全ファイルパスを含むcanonical_refsセクションが含まれる（**必須** — 省略しないこと）
- CONTEXT.mdに再利用可能なアセットとパターンのcode_contextセクションが含まれる
- 延期アイデアが将来のフェーズのために保存された
- STATE.mdがセッション情報で更新された
- ユーザーが次のステップを認識している
</success_criteria>
