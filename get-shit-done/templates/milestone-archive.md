# マイルストーンアーカイブテンプレート

このテンプレートはcomplete-milestoneワークフローで`.planning/milestones/`にアーカイブファイルを作成するために使用されます。

---

## ファイルテンプレート

# Milestone v{{VERSION}}: {{MILESTONE_NAME}}

**Status:** ✅ SHIPPED {{DATE}}
**Phases:** {{PHASE_START}}-{{PHASE_END}}
**Total Plans:** {{TOTAL_PLANS}}

## 概要

{{MILESTONE_DESCRIPTION}}

## フェーズ

{{PHASES_SECTION}}

[このマイルストーン内の各フェーズについて、以下を含める:]

### Phase {{PHASE_NUM}}: {{PHASE_NAME}}

**Goal**: {{PHASE_GOAL}}
**Depends on**: {{DEPENDS_ON}}
**Plans**: {{PLAN_COUNT}} plans

Plans:

- [x] {{PHASE}}-01: {{PLAN_DESCRIPTION}}
- [x] {{PHASE}}-02: {{PLAN_DESCRIPTION}}
      [... すべてのプラン ...]

**Details:**
{{PHASE_DETAILS_FROM_ROADMAP}}

**小数フェーズの場合、(INSERTED)マーカーを含める:**

### Phase 2.1: Critical Security Patch (INSERTED)

**Goal**: Fix authentication bypass vulnerability
**Depends on**: Phase 2
**Plans**: 1 plan

Plans:

- [x] 02.1-01: Patch auth vulnerability

**Details:**
{{PHASE_DETAILS_FROM_ROADMAP}}

---

## マイルストーンサマリー

**小数フェーズ:**

- Phase 2.1: Critical Security Patch（緊急修正のためPhase 2の後に挿入）
- Phase 5.1: Performance Hotfix（本番問題のためPhase 5の後に挿入）

**主要な判断:**
{{DECISIONS_FROM_PROJECT_STATE}}
[例:]

- Decision: Use ROADMAP.md split（理由: 一定のコンテキストコスト）
- Decision: Decimal phase numbering（理由: 明確な挿入セマンティクス）

**解決された問題:**
{{ISSUES_RESOLVED_DURING_MILESTONE}}
[例:]

- 100以上のフェーズでのコンテキストオーバーフローを修正
- フェーズ挿入の混乱を解決

**先送りされた問題:**
{{ISSUES_DEFERRED_TO_LATER}}
[例:]

- PROJECT-STATE.mdの階層化（判断が300を超えるまで先送り）

**発生した技術的負債:**
{{SHORTCUTS_NEEDING_FUTURE_WORK}}
[例:]

- 一部のワークフローにハードコードされたパスが残っている（Phase 5で修正）

---

_現在のプロジェクト状況は.planning/ROADMAP.mdを参照_

---

## 使用ガイドライン

<guidelines>
**マイルストーンアーカイブを作成するタイミング:**
- マイルストーン内のすべてのフェーズが完了した後（v1.0、v1.1、v2.0など）
- complete-milestoneワークフローによってトリガー
- 次のマイルストーン作業の計画前

**テンプレートの記入方法:**

- {{PLACEHOLDERS}}を実際の値に置き換える
- ROADMAP.mdからフェーズの詳細を抽出
- 小数フェーズを(INSERTED)マーカー付きで文書化
- PROJECT-STATE.mdまたはSUMMARYファイルから主要な判断を含める
- 解決された問題と先送りされた問題をリスト
- 将来の参照のために技術的負債を記録

**アーカイブの保存場所:**

- `.planning/milestones/v{VERSION}-{NAME}.md`に保存
- 例: `.planning/milestones/v1.0-mvp.md`

**アーカイブ後:**

- ROADMAP.mdを更新して完了したマイルストーンを`<details>`タグで折りたたむ
- PROJECT.mdをCurrent Stateセクション付きのブラウンフィールド形式に更新
- 次のマイルストーンでフェーズ番号を継続（01から再開しない）
  </guidelines>
</output>