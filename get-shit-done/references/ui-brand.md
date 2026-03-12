<ui_patterns>

ユーザー向けGSD出力のビジュアルパターン。オーケストレーターはこのファイルを@参照します。

## ステージバナー

主要なワークフロー遷移に使用します。

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► {STAGE NAME}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**ステージ名（大文字）:**
- `QUESTIONING`
- `RESEARCHING`
- `DEFINING REQUIREMENTS`
- `CREATING ROADMAP`
- `PLANNING PHASE {N}`
- `EXECUTING WAVE {N}`
- `VERIFYING`
- `PHASE {N} COMPLETE ✓`
- `MILESTONE COMPLETE 🎉`

---

## チェックポイントボックス

ユーザーアクションが必要。62文字幅。

```
╔══════════════════════════════════════════════════════════════╗
║  CHECKPOINT: {Type}                                          ║
╚══════════════════════════════════════════════════════════════╝

{Content}

──────────────────────────────────────────────────────────────
→ {ACTION PROMPT}
──────────────────────────────────────────────────────────────
```

**タイプ:**
- `CHECKPOINT: Verification Required` → `→ Type "approved" or describe issues`
- `CHECKPOINT: Decision Required` → `→ Select: option-a / option-b`
- `CHECKPOINT: Action Required` → `→ Type "done" when complete`

---

## ステータスシンボル

```
✓  完了 / パス / 検証済み
✗  失敗 / 不足 / ブロック
◆  進行中
○  保留中
⚡ 自動承認
⚠  警告
🎉 マイルストーン完了（バナーのみ）
```

---

## 進捗表示

**フェーズ/マイルストーンレベル:**
```
Progress: ████████░░ 80%
```

**タスクレベル:**
```
Tasks: 2/4 complete
```

**プランレベル:**
```
Plans: 3/5 complete
```

---

## スポーンインジケーター

```
◆ Spawning researcher...

◆ Spawning 4 researchers in parallel...
  → Stack research
  → Features research
  → Architecture research
  → Pitfalls research

✓ Researcher complete: STACK.md written
```

---

## Next Upブロック

主要な完了の最後に常に表示します。

```
───────────────────────────────────────────────────────────────

## ▶ Next Up

**{Identifier}: {Name}** — {一行の説明}

`{コピー&ペーストするコマンド}`

<sub>`/clear` first → fresh context window</sub>

───────────────────────────────────────────────────────────────

**Also available:**
- `/gsd:alternative-1` — description
- `/gsd:alternative-2` — description

───────────────────────────────────────────────────────────────
```

---

## エラーボックス

```
╔══════════════════════════════════════════════════════════════╗
║  ERROR                                                       ║
╚══════════════════════════════════════════════════════════════╝

{エラーの説明}

**To fix:** {解決手順}
```

---

## テーブル

```
| Phase | Status | Plans | Progress |
|-------|--------|-------|----------|
| 1     | ✓      | 3/3   | 100%     |
| 2     | ◆      | 1/4   | 25%      |
| 3     | ○      | 0/2   | 0%       |
```

---

## アンチパターン

- ボックス/バナーの幅が不統一
- バナースタイルの混在（`===`, `---`, `***`）
- バナーで`GSD ►`プレフィックスを省略
- ランダムな絵文字（`🚀`, `✨`, `💫`）
- 完了後のNext Upブロックの欠落

</ui_patterns>