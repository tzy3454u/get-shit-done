<overview>
GSDフレームワークのGit統合。
</overview>

<core_principle>

**成果をコミットせよ、プロセスではない。**

gitログは、計画活動の日記ではなく、何がリリースされたかの変更履歴として読めるべきです。
</core_principle>

<commit_points>

| イベント                   | コミット? | 理由                                              |
| ----------------------- | ------- | ------------------------------------------------ |
| BRIEF + ROADMAP作成 | はい     | プロジェクト初期化                           |
| PLAN.md作成         | いいえ      | 中間成果物 - プラン完了時にコミット       |
| RESEARCH.md作成     | いいえ      | 中間成果物                                     |
| DISCOVERY.md作成    | いいえ      | 中間成果物                                     |
| **タスク完了**      | はい     | 作業の原子単位（1タスクにつき1コミット）         |
| **プラン完了**      | はい     | メタデータコミット（SUMMARY + STATE + ROADMAP）     |
| ハンドオフ作成         | はい     | WIP状態の保存                              |

</commit_points>

<git_check>

```bash
[ -d .git ] && echo "GIT_EXISTS" || echo "NO_GIT"
```

NO_GITの場合: サイレントに`git init`を実行する。GSDプロジェクトは常に独自のリポジトリを持つ。
</git_check>

<commit_formats>

<format name="initialization">
## プロジェクト初期化（briefとroadmapを一緒に）

```
docs: initialize [project-name] ([N] phases)

[PROJECT.mdからの一行説明]

Phases:
1. [phase-name]: [goal]
2. [phase-name]: [goal]
3. [phase-name]: [goal]
```

コミット対象:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: initialize [project-name] ([N] phases)" --files .planning/
```

</format>

<format name="task-completion">
## タスク完了（プラン実行中）

各タスクは完了直後に独自のコミットを取得します。

```
{type}({phase}-{plan}): {task-name}

- [主要な変更1]
- [主要な変更2]
- [主要な変更3]
```

**コミットタイプ:**
- `feat` - 新機能/機能性
- `fix` - バグ修正
- `test` - テストのみ（TDD REDフェーズ）
- `refactor` - コードクリーンアップ（TDD REFACTORフェーズ）
- `perf` - パフォーマンス改善
- `chore` - 依存関係、設定、ツール

**例:**

```bash
# 標準タスク
git add src/api/auth.ts src/types/user.ts
git commit -m "feat(08-02): create user registration endpoint

- POST /auth/register validates email and password
- Checks for duplicate users
- Returns JWT token on success
"

# TDDタスク - REDフェーズ
git add src/__tests__/jwt.test.ts
git commit -m "test(07-02): add failing test for JWT generation

- Tests token contains user ID claim
- Tests token expires in 1 hour
- Tests signature verification
"

# TDDタスク - GREENフェーズ
git add src/utils/jwt.ts
git commit -m "feat(07-02): implement JWT generation

- Uses jose library for signing
- Includes user ID and expiry claims
- Signs with HS256 algorithm
"
```

</format>

<format name="plan-completion">
## プラン完了（全タスク完了後）

全タスクがコミットされた後、最後のメタデータコミットでプラン完了を記録します。

```
docs({phase}-{plan}): complete [plan-name] plan

Tasks completed: [N]/[N]
- [Task 1 name]
- [Task 2 name]
- [Task 3 name]

SUMMARY: .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md
```

コミット対象:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs({phase}-{plan}): complete [plan-name] plan" --files .planning/phases/XX-name/{phase}-{plan}-PLAN.md .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md .planning/STATE.md .planning/ROADMAP.md
```

**注意:** コードファイルは含まない - 既にタスクごとにコミット済み。

</format>

<format name="handoff">
## ハンドオフ（WIP）

```
wip: [phase-name] paused at task [X]/[Y]

Current: [task name]
[If blocked:] Blocked: [reason]
```

コミット対象:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "wip: [phase-name] paused at task [X]/[Y]" --files .planning/
```

</format>
</commit_formats>

<example_log>

**旧アプローチ（プラン単位のコミット）:**
```
a7f2d1 feat(checkout): Stripe payments with webhook verification
3e9c4b feat(products): catalog with search, filters, and pagination
8a1b2c feat(auth): JWT with refresh rotation using jose
5c3d7e feat(foundation): Next.js 15 + Prisma + Tailwind scaffold
2f4a8d docs: initialize ecommerce-app (5 phases)
```

**新アプローチ（タスク単位のコミット）:**
```
# Phase 04 - Checkout
1a2b3c docs(04-01): complete checkout flow plan
4d5e6f feat(04-01): add webhook signature verification
7g8h9i feat(04-01): implement payment session creation
0j1k2l feat(04-01): create checkout page component

# Phase 03 - Products
3m4n5o docs(03-02): complete product listing plan
6p7q8r feat(03-02): add pagination controls
9s0t1u feat(03-02): implement search and filters
2v3w4x feat(03-01): create product catalog schema

# Phase 02 - Auth
5y6z7a docs(02-02): complete token refresh plan
8b9c0d feat(02-02): implement refresh token rotation
1e2f3g test(02-02): add failing test for token refresh
4h5i6j docs(02-01): complete JWT setup plan
7k8l9m feat(02-01): add JWT generation and validation
0n1o2p chore(02-01): install jose library

# Phase 01 - Foundation
3q4r5s docs(01-01): complete scaffold plan
6t7u8v feat(01-01): configure Tailwind and globals
9w0x1y feat(01-01): set up Prisma with database
2z3a4b feat(01-01): create Next.js 15 project

# Initialization
5c6d7e docs: initialize ecommerce-app (5 phases)
```

各プランは2-4コミットを生成（タスク + メタデータ）。明確で、細粒度で、bisect可能。

</example_log>

<anti_patterns>

**コミットしないもの（中間成果物）:**
- PLAN.md作成（プラン完了時にコミット）
- RESEARCH.md（中間成果物）
- DISCOVERY.md（中間成果物）
- 軽微な計画の微調整
- 「ロードマップのタイポを修正」

**コミットするもの（成果）:**
- 各タスク完了（feat/fix/test/refactor）
- プラン完了メタデータ（docs）
- プロジェクト初期化（docs）

**重要な原則:** 動作するコードとリリースされた成果をコミットする。計画プロセスではない。

</anti_patterns>

<commit_strategy_rationale>

## なぜタスク単位のコミットか？

**AIのためのコンテキストエンジニアリング:**
- Gitの履歴が将来のClaudeセッションの主要なコンテキストソースになる
- `git log --grep="{phase}-{plan}"`でプランのすべての作業を表示
- `git diff <hash>^..<hash>`でタスクごとの正確な変更を表示
- SUMMARY.mdの解析への依存が減り = 実際の作業により多くのコンテキストを使用可能

**障害回復:**
- タスク1コミット済み、タスク2失敗
- 次のセッションのClaude: タスク1の完了を確認、タスク2を再試行可能
- 最後の成功タスクまで`git reset --hard`可能

**デバッグ:**
- `git bisect`が失敗したプランではなく、正確に失敗したタスクを特定
- `git blame`が行を特定のタスクコンテキストに追跡
- 各コミットは独立して取り消し可能

**可観測性:**
- ソロ開発者 + Claudeのワークフローは細粒度の帰属から恩恵を受ける
- アトミックコミットはgitのベストプラクティス
- コンシューマーが人間ではなくClaudeの場合、「コミットノイズ」は無関係

</commit_strategy_rationale>