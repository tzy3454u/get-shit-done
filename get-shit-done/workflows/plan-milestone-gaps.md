<purpose>
`/gsd:audit-milestone`で特定されたギャップを埋めるために必要なすべてのフェーズを作成する。MILESTONE-AUDIT.mdを読み込み、ギャップを論理的なフェーズにグループ化し、ROADMAP.mdにフェーズエントリを作成し、各フェーズの計画を提案する。1つのコマンドで全修正フェーズを作成する — ギャップごとに手動で`/gsd:add-phase`を実行する必要はない。
</purpose>

<required_reading>
開始前に、呼び出しプロンプトのexecution_contextで参照されているすべてのファイルを読み込むこと。
</required_reading>

<process>

## 1. 監査結果の読み込み

```bash
# Find the most recent audit file
ls -t .planning/v*-MILESTONE-AUDIT.md 2>/dev/null | head -1
```

YAMLフロントマターを解析して構造化されたギャップを抽出する：
- `gaps.requirements` — 未達成の要件
- `gaps.integration` — 欠落しているフェーズ間接続
- `gaps.flows` — 壊れたE2Eフロー

監査ファイルが存在しないか、ギャップがない場合はエラー：
```
監査ギャップが見つかりません。先に`/gsd:audit-milestone`を実行すること。
```

## 2. ギャップの優先順位付け

REQUIREMENTS.mdの優先度でギャップをグループ化する：

| 優先度 | アクション |
|----------|--------|
| `must` | フェーズを作成、マイルストーンをブロック |
| `should` | フェーズを作成、推奨 |
| `nice` | ユーザーに確認：含めるか延期するか？ |

インテグレーション/フローのギャップについては、影響を受ける要件から優先度を推測する。

## 3. ギャップをフェーズにグループ化

関連するギャップを論理的なフェーズにまとめる：

**グループ化ルール：**
- 同じ影響フェーズ → 1つの修正フェーズに統合
- 同じサブシステム（auth、API、UI） → 統合
- 依存関係の順序（接続の前にスタブを修正）
- フェーズは集中的に：各2-4タスク

**グループ化の例：**
```
Gap: DASH-01 unsatisfied (Dashboard doesn't fetch)
Gap: Integration Phase 1→3 (Auth not passed to API calls)
Gap: Flow "View dashboard" broken at data fetch

→ Phase 6: "Wire Dashboard to API"
  - Add fetch to Dashboard.tsx
  - Include auth header in fetch
  - Handle response, update state
  - Render user data
```

## 4. フェーズ番号の決定

最も高い既存フェーズを見つける：
```bash
# Get sorted phase list, extract last one
PHASES=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phases list)
HIGHEST=$(printf '%s\n' "$PHASES" | jq -r '.directories[-1]')
```

新しいフェーズはそこから続く：
- フェーズ5が最高の場合、ギャップはフェーズ6、7、8...になる

## 5. ギャップ解消プランの提示

```markdown
## ギャップ解消プラン

**マイルストーン:** {version}
**解消するギャップ:** {N} 要件、{M} インテグレーション、{K} フロー

### 提案フェーズ

**フェーズ {N}: {Name}**
解消対象:
- {REQ-ID}: {description}
- インテグレーション: {from} → {to}
タスク数: {count}

**フェーズ {N+1}: {Name}**
解消対象:
- {REQ-ID}: {description}
- フロー: {flow name}
タスク数: {count}

{nice-to-haveギャップが存在する場合:}

### 延期（nice-to-have）

これらのギャップはオプションです。含めますか？
- {gap description}
- {gap description}

---

これら{X}個のフェーズを作成しますか？（yes / adjust / defer all optional）
```

ユーザーの確認を待つ。

## 6. ROADMAP.mdの更新

現在のマイルストーンに新しいフェーズを追加する：

```markdown
### Phase {N}: {Name}
**Goal:** {derived from gaps being closed}
**Requirements:** {REQ-IDs being satisfied}
**Gap Closure:** Closes gaps from audit

### Phase {N+1}: {Name}
...
```

## 7. REQUIREMENTS.mdトレーサビリティテーブルの更新（必須）

ギャップ解消フェーズに割り当てられた各REQ-IDについて：
- Phaseカラムを新しいギャップ解消フェーズに更新
- Statusを`Pending`にリセット

監査で未達成と判定されたチェック済み要件をリセットする：
- 監査で未達成とされた要件の`[x]` → `[ ]`に変更
- REQUIREMENTS.md上部のカバレッジカウントを更新

```bash
# Verify traceability table reflects gap closure assignments
grep -c "Pending" .planning/REQUIREMENTS.md
```

## 8. フェーズディレクトリの作成

```bash
mkdir -p ".planning/phases/{NN}-{name}"
```

## 9. ロードマップと要件更新のコミット

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(roadmap): add gap closure phases {N}-{M}" --files .planning/ROADMAP.md .planning/REQUIREMENTS.md
```

## 10. 次のステップの提案

```markdown
## ✓ ギャップ解消フェーズ作成完了

**追加されたフェーズ:** {N} - {M}
**対処されたギャップ:** {count} 要件、{count} インテグレーション、{count} フロー

---

## ▶ 次のステップ

**最初のギャップ解消フェーズを計画**

`/gsd:plan-phase {N}`

<sub>先に`/clear` → 新しいコンテキストウィンドウ</sub>

---

**その他のオプション:**
- `/gsd:execute-phase {N}` — プランが既に存在する場合
- `cat .planning/ROADMAP.md` — 更新されたロードマップを確認

---

**すべてのギャップフェーズ完了後:**

`/gsd:audit-milestone` — ギャップが解消されたか再監査
`/gsd:complete-milestone {version}` — 監査合格後にアーカイブ
```

</process>

<gap_to_phase_mapping>

## ギャップからタスクへの変換

**要件ギャップ → タスク：**
```yaml
gap:
  id: DASH-01
  description: "User sees their data"
  reason: "Dashboard exists but doesn't fetch from API"
  missing:
    - "useEffect with fetch to /api/user/data"
    - "State for user data"
    - "Render user data in JSX"

becomes:

phase: "Wire Dashboard Data"
tasks:
  - name: "Add data fetching"
    files: [src/components/Dashboard.tsx]
    action: "Add useEffect that fetches /api/user/data on mount"

  - name: "Add state management"
    files: [src/components/Dashboard.tsx]
    action: "Add useState for userData, loading, error states"

  - name: "Render user data"
    files: [src/components/Dashboard.tsx]
    action: "Replace placeholder with userData.map rendering"
```

**インテグレーションギャップ → タスク：**
```yaml
gap:
  from_phase: 1
  to_phase: 3
  connection: "Auth token → API calls"
  reason: "Dashboard API calls don't include auth header"
  missing:
    - "Auth header in fetch calls"
    - "Token refresh on 401"

becomes:

phase: "Add Auth to Dashboard API Calls"
tasks:
  - name: "Add auth header to fetches"
    files: [src/components/Dashboard.tsx, src/lib/api.ts]
    action: "Include Authorization header with token in all API calls"

  - name: "Handle 401 responses"
    files: [src/lib/api.ts]
    action: "Add interceptor to refresh token or redirect to login on 401"
```

**フローギャップ → タスク：**
```yaml
gap:
  name: "User views dashboard after login"
  broken_at: "Dashboard data load"
  reason: "No fetch call"
  missing:
    - "Fetch user data on mount"
    - "Display loading state"
    - "Render user data"

becomes:

# 通常、要件/インテグレーションギャップと同じフェーズ
# フローギャップは他のギャップタイプと重複することが多い
```

</gap_to_phase_mapping>

<success_criteria>
- [ ] MILESTONE-AUDIT.mdが読み込まれ、ギャップが解析された
- [ ] ギャップが優先順位付けされた（must/should/nice）
- [ ] ギャップが論理的なフェーズにグループ化された
- [ ] ユーザーがフェーズプランを確認した
- [ ] ROADMAP.mdが新しいフェーズで更新された
- [ ] REQUIREMENTS.mdトレーサビリティテーブルがギャップ解消フェーズの割り当てで更新された
- [ ] 未達成要件のチェックボックスがリセットされた（`[x]` → `[ ]`）
- [ ] REQUIREMENTS.mdのカバレッジカウントが更新された
- [ ] フェーズディレクトリが作成された
- [ ] 変更がコミットされた（REQUIREMENTS.mdを含む）
- [ ] ユーザーが次に`/gsd:plan-phase`を実行することを把握した
</success_criteria>
