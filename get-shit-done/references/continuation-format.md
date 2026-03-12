# 継続フォーマット

コマンドやワークフローの完了後に次のステップを提示するための標準フォーマット。

## 基本構造

```
---

## ▶ Next Up

**{identifier}: {name}** — {一行の説明}

`{コピー&ペーストするコマンド}`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- `{代替オプション1}` — 説明
- `{代替オプション2}` — 説明

---
```

## フォーマットルール

1. **常に内容を示す** — 名前 + 説明、コマンドパスだけにしない
2. **ソースからコンテキストを取得する** — フェーズにはROADMAP.md、プランにはPLAN.mdの`<objective>`
3. **コマンドはインラインコードで** — バッククォート、コピー&ペーストしやすく、クリック可能なリンクとして表示
4. **`/clear`の説明** — 常に含める、簡潔だが理由を説明
5. **"Also available"であって"Other options"ではない** — よりアプリらしく聞こえる
6. **視覚的区切り** — `---`を上下に配置して目立たせる

## バリエーション

### 次のプランを実行

```
---

## ▶ Next Up

**02-03: Refresh Token Rotation** — Add /api/auth/refresh with sliding expiry

`/gsd:execute-phase 2`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- Review plan before executing
- `/gsd:list-phase-assumptions 2` — check assumptions

---
```

### フェーズの最後のプランを実行

これが最後のプランであることと、その後に何が来るかを記載する:

```
---

## ▶ Next Up

**02-03: Refresh Token Rotation** — Add /api/auth/refresh with sliding expiry
<sub>Final plan in Phase 2</sub>

`/gsd:execute-phase 2`

<sub>`/clear` first → fresh context window</sub>

---

**After this completes:**
- Phase 2 → Phase 3 transition
- Next: **Phase 3: Core Features** — User dashboard and settings

---
```

### フェーズの計画

```
---

## ▶ Next Up

**Phase 2: Authentication** — JWT login flow with refresh tokens

`/gsd:plan-phase 2`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- `/gsd:discuss-phase 2` — gather context first
- `/gsd:research-phase 2` — investigate unknowns
- Review roadmap

---
```

### フェーズ完了、次の準備完了

次のアクションの前に完了ステータスを表示する:

```
---

## ✓ Phase 2 Complete

3/3 plans executed

## ▶ Next Up

**Phase 3: Core Features** — User dashboard, settings, and data export

`/gsd:plan-phase 3`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- `/gsd:discuss-phase 3` — gather context first
- `/gsd:research-phase 3` — investigate unknowns
- Review what Phase 2 built

---
```

### 複数の同等オプション

明確な主要アクションがない場合:

```
---

## ▶ Next Up

**Phase 3: Core Features** — User dashboard, settings, and data export

**To plan directly:** `/gsd:plan-phase 3`

**To discuss context first:** `/gsd:discuss-phase 3`

**To research unknowns:** `/gsd:research-phase 3`

<sub>`/clear` first → fresh context window</sub>

---
```

### マイルストーン完了

```
---

## 🎉 Milestone v1.0 Complete

All 4 phases shipped

## ▶ Next Up

**Start v1.1** — questioning → research → requirements → roadmap

`/gsd:new-milestone`

<sub>`/clear` first → fresh context window</sub>

---
```

## コンテキストの取得

### フェーズの場合（ROADMAP.mdから）:

```markdown
### Phase 2: Authentication
**Goal**: JWT login flow with refresh tokens
```

抽出: `**Phase 2: Authentication** — JWT login flow with refresh tokens`

### プランの場合（ROADMAP.mdから）:

```markdown
Plans:
- [ ] 02-03: Add refresh token rotation
```

またはPLAN.mdの`<objective>`から:

```xml
<objective>
Add refresh token rotation with sliding expiry window.

Purpose: Extend session lifetime without compromising security.
</objective>
```

抽出: `**02-03: Refresh Token Rotation** — Add /api/auth/refresh with sliding expiry`

## アンチパターン

### してはいけない: コマンドだけ（コンテキストなし）

```
## To Continue

Run `/clear`, then paste:
/gsd:execute-phase 2
```

ユーザーには02-03が何なのかわからない。

### してはいけない: /clearの説明がない

```
`/gsd:plan-phase 3`

Run /clear first.
```

理由を説明していない。ユーザーがスキップするかもしれない。

### してはいけない: "Other options"という表現

```
Other options:
- Review roadmap
```

後付けのように聞こえる。代わりに"Also available:"を使用する。

### してはいけない: コマンドにフェンスコードブロック

```
```
/gsd:plan-phase 3
```
```

テンプレート内のフェンスブロックはネストの曖昧さを生む。代わりにインラインバッククォートを使用する。
