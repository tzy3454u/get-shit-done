# リサーチテンプレート

`.planning/phases/XX-name/{phase_num}-RESEARCH.md` 用テンプレート - 計画前の包括的なエコシステムリサーチ。

**目的:** Claudeがフェーズを適切に実装するために知るべきことを文書化します - 「どのライブラリ」だけでなく「専門家はこれをどう構築するか」。

---

## ファイルテンプレート

```markdown
# Phase [X]: [Name] - Research

**Researched:** [date]
**Domain:** [主要な技術/問題ドメイン]
**Confidence:** [HIGH/MEDIUM/LOW]

<user_constraints>
## ユーザー制約（CONTEXT.mdから）

**重要:** /gsd:discuss-phaseからCONTEXT.mdが存在する場合、ロックされた判断をここにそのままコピーしてください。これらはプランナーによって必ず遵守される必要があります。

### ロックされた判断
[CONTEXT.mdの`## Decisions`セクションからコピー - これらは交渉不可]
- [判断1]
- [判断2]

### Claudeの裁量
[CONTEXT.mdからコピー - リサーチャー/プランナーが選択できる領域]
- [領域1]
- [領域2]

### 先送りのアイデア（スコープ外）
[CONTEXT.mdからコピー - これらをリサーチまたは計画しない]
- [先送り1]
- [先送り2]

**CONTEXT.mdが存在しない場合:** 「ユーザー制約なし - すべての判断はClaudeの裁量」と記載
</user_constraints>

<research_summary>
## サマリー

[2〜3段落のエグゼクティブサマリー]
- 何を調査したか
- 標準的なアプローチは何か
- 主要な推奨事項

**主な推奨事項:** [一行の実行可能なガイダンス]
</research_summary>

<standard_stack>
## 標準スタック

このドメインの確立されたライブラリ/ツール:

### コア
| ライブラリ | バージョン | 目的 | 標準である理由 |
|---------|---------|---------|--------------|
| [名前] | [バージョン] | [何をするか] | [なぜ専門家が使うか] |
| [名前] | [バージョン] | [何をするか] | [なぜ専門家が使うか] |

### サポート
| ライブラリ | バージョン | 目的 | 使用場面 |
|---------|---------|---------|-------------|
| [名前] | [バージョン] | [何をするか] | [ユースケース] |
| [名前] | [バージョン] | [何をするか] | [ユースケース] |

### 検討した代替案
| 標準の代わりに | 使用可能 | トレードオフ |
|------------|-----------|----------|
| [標準] | [代替] | [代替が適切な場合] |

**インストール:**
```bash
npm install [packages]
# or
yarn add [packages]
```
</standard_stack>

<architecture_patterns>
## アーキテクチャパターン

### 推奨プロジェクト構造
```
src/
├── [folder]/        # [目的]
├── [folder]/        # [目的]
└── [folder]/        # [目的]
```

### パターン1: [パターン名]
**概要:** [説明]
**使用場面:** [条件]
**例:**
```typescript
// [Context7/公式ドキュメントからのコード例]
```

### パターン2: [パターン名]
**概要:** [説明]
**使用場面:** [条件]
**例:**
```typescript
// [コード例]
```

### 避けるべきアンチパターン
- **[アンチパターン]:** [なぜ悪いか、代わりに何をすべきか]
- **[アンチパターン]:** [なぜ悪いか、代わりに何をすべきか]
</architecture_patterns>

<dont_hand_roll>
## 自作しない

シンプルに見えるが既存のソリューションがある問題:

| 問題 | 自作しない | 代わりに使う | 理由 |
|---------|-------------|-------------|-----|
| [問題] | [自作するもの] | [ライブラリ] | [エッジケース、複雑性] |
| [問題] | [自作するもの] | [ライブラリ] | [エッジケース、複雑性] |
| [問題] | [自作するもの] | [ライブラリ] | [エッジケース、複雑性] |

**重要な知見:** [このドメインでカスタムソリューションが劣る理由]
</dont_hand_roll>

<common_pitfalls>
## よくある落とし穴

### 落とし穴1: [名前]
**何がうまくいかないか:** [説明]
**なぜ起こるか:** [根本原因]
**避ける方法:** [予防策]
**警告サイン:** [早期発見方法]

### 落とし穴2: [名前]
**何がうまくいかないか:** [説明]
**なぜ起こるか:** [根本原因]
**避ける方法:** [予防策]
**警告サイン:** [早期発見方法]

### 落とし穴3: [名前]
**何がうまくいかないか:** [説明]
**なぜ起こるか:** [根本原因]
**避ける方法:** [予防策]
**警告サイン:** [早期発見方法]
</common_pitfalls>

<code_examples>
## コード例

公式ソースからの検証済みパターン:

### [一般的な操作1]
```typescript
// Source: [Context7/公式ドキュメントURL]
[code]
```

### [一般的な操作2]
```typescript
// Source: [Context7/公式ドキュメントURL]
[code]
```

### [一般的な操作3]
```typescript
// Source: [Context7/公式ドキュメントURL]
[code]
```
</code_examples>

<sota_updates>
## 最新動向（2024-2025）

最近変わったこと:

| 旧アプローチ | 現在のアプローチ | 変更時期 | 影響 |
|--------------|------------------|--------------|--------|
| [旧] | [新] | [日付/バージョン] | [実装への意味] |

**検討すべき新しいツール/パターン:**
- [ツール/パターン]: [何を可能にするか、いつ使うか]
- [ツール/パターン]: [何を可能にするか、いつ使うか]

**非推奨/古くなったもの:**
- [もの]: [なぜ古くなったか、何に置き換えられたか]
</sota_updates>

<open_questions>
## 未解決の質問

完全に解決できなかったこと:

1. **[質問]**
   - わかっていること: [部分的な情報]
   - 不明なこと: [ギャップ]
   - 推奨: [計画/実行中の対処方法]

2. **[質問]**
   - わかっていること: [部分的な情報]
   - 不明なこと: [ギャップ]
   - 推奨: [対処方法]
</open_questions>

<sources>
## ソース

### プライマリ（HIGH信頼度）
- [Context7 library ID] - [取得したトピック]
- [公式ドキュメントURL] - [確認した内容]

### セカンダリ（MEDIUM信頼度）
- [公式ソースで検証されたWebSearch] - [発見 + 検証]

### ターシャリ（LOW信頼度 - 要検証）
- [WebSearchのみ] - [発見、実装中の検証対象としてマーク]
</sources>

<metadata>
## メタデータ

**リサーチスコープ:**
- コア技術: [何]
- エコシステム: [調査したライブラリ]
- パターン: [調査したパターン]
- 落とし穴: [確認した領域]

**信頼度の内訳:**
- 標準スタック: [HIGH/MEDIUM/LOW] - [理由]
- アーキテクチャ: [HIGH/MEDIUM/LOW] - [理由]
- 落とし穴: [HIGH/MEDIUM/LOW] - [理由]
- コード例: [HIGH/MEDIUM/LOW] - [理由]

**リサーチ日:** [date]
**有効期限:** [見積もり - 安定した技術は30日、動きの速いものは7日]
</metadata>

---

*Phase: XX-name*
*Research completed: [date]*
*Ready for planning: [yes/no]*
```

---

## 良い例

```markdown
# Phase 3: 3D City Driving - Research

**Researched:** 2025-01-20
**Domain:** Three.js 3Dウェブゲームとドライビングメカニクス
**Confidence:** HIGH

<research_summary>
## サマリー

3D都市ドライビングゲーム構築のためにThree.jsエコシステムを調査しました。標準的なアプローチはコンポーネントアーキテクチャにReact Three Fiberを使用したThree.js、物理演算にRapier、一般的なヘルパーにdreiを使用します。

重要な発見: 物理演算や衝突検出を自作しないこと。Rapier（@react-three/rapier経由）は車両物理、地形衝突、都市オブジェクトのインタラクションを効率的に処理します。カスタム物理コードはバグやパフォーマンスの問題につながります。

**主な推奨事項:** R3F + Rapier + dreiスタックを使用。dreiの車両コントローラーから始め、Rapierの車両物理を追加し、パフォーマンスのためにインスタンス化メッシュで都市を構築。
</research_summary>

<standard_stack>
## 標準スタック

### コア
| ライブラリ | バージョン | 目的 | 標準である理由 |
|---------|---------|---------|--------------|
| three | 0.160.0 | 3Dレンダリング | ウェブ3Dの標準 |
| @react-three/fiber | 8.15.0 | Three.js用Reactレンダラー | 宣言的3D、より良いDX |
| @react-three/drei | 9.92.0 | ヘルパーと抽象化 | 一般的な問題を解決 |
| @react-three/rapier | 1.2.1 | 物理エンジンバインディング | R3F向け最良の物理 |

### サポート
| ライブラリ | バージョン | 目的 | 使用場面 |
|---------|---------|---------|-------------|
| @react-three/postprocessing | 2.16.0 | ビジュアルエフェクト | Bloom、DOF、モーションブラー |
| leva | 0.9.35 | デバッグUI | パラメータ調整 |
| zustand | 4.4.7 | 状態管理 | ゲーム状態、UI状態 |
| use-sound | 4.0.1 | オーディオ | エンジン音、環境音 |

### 検討した代替案
| 標準の代わりに | 使用可能 | トレードオフ |
|------------|-----------|----------|
| Rapier | Cannon.js | Cannonはシンプルだが車両向けのパフォーマンスが劣る |
| R3F | Vanilla Three | ReactなしならVanilla、ただしR3FのDXがはるかに良い |
| drei | カスタムヘルパー | dreiは実戦テスト済み、再発明しない |

**インストール:**
```bash
npm install three @react-three/fiber @react-three/drei @react-three/rapier zustand
```
</standard_stack>

<architecture_patterns>
## アーキテクチャパターン

### 推奨プロジェクト構造
```
src/
├── components/
│   ├── Vehicle/          # 物理付きプレイヤーカー
│   ├── City/             # 都市生成と建物
│   ├── Road/             # 道路ネットワーク
│   └── Environment/      # 空、照明、霧
├── hooks/
│   ├── useVehicleControls.ts
│   └── useGameState.ts
├── stores/
│   └── gameStore.ts      # Zustand状態
└── utils/
    └── cityGenerator.ts  # 手続き生成ヘルパー
```

### パターン1: Rapier物理による車両
**概要:** カスタム物理ではなく、車両固有の設定を持つRigidBodyを使用
**使用場面:** あらゆる地上車両
**例:**
```typescript
// Source: @react-three/rapier docs
import { RigidBody, useRapier } from '@react-three/rapier'

function Vehicle() {
  const rigidBody = useRef()

  return (
    <RigidBody
      ref={rigidBody}
      type="dynamic"
      colliders="hull"
      mass={1500}
      linearDamping={0.5}
      angularDamping={0.5}
    >
      <mesh>
        <boxGeometry args={[2, 1, 4]} />
        <meshStandardMaterial />
      </mesh>
    </RigidBody>
  )
}
```

### パターン2: 都市向けインスタンス化メッシュ
**概要:** 繰り返しオブジェクト（建物、木、小道具）にInstancedMeshを使用
**使用場面:** 類似オブジェクトが100以上の場合
**例:**
```typescript
// Source: drei docs
import { Instances, Instance } from '@react-three/drei'

function Buildings({ positions }) {
  return (
    <Instances limit={1000}>
      <boxGeometry />
      <meshStandardMaterial />
      {positions.map((pos, i) => (
        <Instance key={i} position={pos} scale={[1, Math.random() * 5 + 1, 1]} />
      ))}
    </Instances>
  )
}
```

### 避けるべきアンチパターン
- **レンダーループ内でのメッシュ作成:** 一度作成し、トランスフォームのみ更新
- **InstancedMeshを使わない:** 建物に個別メッシュを使うとパフォーマンスが崩壊
- **カスタム物理演算:** Rapierの方が常に優れている
</architecture_patterns>

<dont_hand_roll>
## 自作しない

| 問題 | 自作しない | 代わりに使う | 理由 |
|---------|-------------|-------------|-----|
| 車両物理 | カスタム速度/加速度 | Rapier RigidBody | ホイール摩擦、サスペンション、衝突は複雑 |
| 衝突検出 | すべてをレイキャスト | Rapierコライダー | パフォーマンス、エッジケース、トンネリング |
| カメラ追従 | 手動lerp | drei CameraControlsまたはuseFrameを使ったカスタム | スムーズな補間、境界 |
| 都市生成 | 完全ランダム配置 | バリエーション用ノイズ付きグリッドベース | ランダムは見た目が悪く、グリッドは予測可能 |
| LOD | 手動距離チェック | drei <Detailed> | トランジション、ヒステリシスを処理 |

**重要な知見:** 3Dゲーム開発には40年以上の解決済み問題があります。Rapierは適切な物理シミュレーションを実装しています。dreiは適切な3Dヘルパーを実装しています。これらに逆らうと「ゲームの手触り」の問題に見えるが実際は物理のエッジケースであるバグにつながります。
</dont_hand_roll>

<common_pitfalls>
## よくある落とし穴

### 落とし穴1: 物理のトンネリング
**何がうまくいかないか:** 高速オブジェクトが壁を通過する
**なぜ起こるか:** デフォルトの物理ステップが速度に対して大きすぎる
**避ける方法:** RapierでCCD（連続衝突検出）を使用
**警告サイン:** オブジェクトが建物の外にランダムに出現

### 落とし穴2: ドローコールによるパフォーマンス低下
**何がうまくいかないか:** 多くの建物でゲームがカクつく
**なぜ起こるか:** 各メッシュ = 1ドローコール、数百の建物 = 数百のコール
**避ける方法:** 類似オブジェクトにInstancedMesh、静的ジオメトリをマージ
**警告サイン:** GPUバウンド、シンプルなシーンなのに低FPS

### 落とし穴3: 車両の「浮遊」感
**何がうまくいかないか:** 車が地面に密着していない
**なぜ起こるか:** 適切なホイール/サスペンションシミュレーションが欠如
**避ける方法:** Rapier車両コントローラーを使用するか、質量/ダンピングを慎重に調整
**警告サイン:** 車が不自然に跳ね、コーナーでグリップしない
</common_pitfalls>

<code_examples>
## コード例

### 基本的なR3F + Rapierセットアップ
```typescript
// Source: @react-three/rapier getting started
import { Canvas } from '@react-three/fiber'
import { Physics } from '@react-three/rapier'

function Game() {
  return (
    <Canvas>
      <Physics gravity={[0, -9.81, 0]}>
        <Vehicle />
        <City />
        <Ground />
      </Physics>
    </Canvas>
  )
}
```

### 車両コントロールフック
```typescript
// Source: コミュニティパターン、dreiドキュメントで検証済み
import { useFrame } from '@react-three/fiber'
import { useKeyboardControls } from '@react-three/drei'

function useVehicleControls(rigidBodyRef) {
  const [, getKeys] = useKeyboardControls()

  useFrame(() => {
    const { forward, back, left, right } = getKeys()
    const body = rigidBodyRef.current
    if (!body) return

    const impulse = { x: 0, y: 0, z: 0 }
    if (forward) impulse.z -= 10
    if (back) impulse.z += 5

    body.applyImpulse(impulse, true)

    if (left) body.applyTorqueImpulse({ x: 0, y: 2, z: 0 }, true)
    if (right) body.applyTorqueImpulse({ x: 0, y: -2, z: 0 }, true)
  })
}
```
</code_examples>

<sota_updates>
## 最新動向（2024-2025）

| 旧アプローチ | 現在のアプローチ | 変更時期 | 影響 |
|--------------|------------------|--------------|--------|
| cannon-es | Rapier | 2023 | Rapierの方が高速で保守が良い |
| vanilla Three.js | React Three Fiber | 2020+ | R3FがReactアプリの標準に |
| 手動InstancedMesh | drei <Instances> | 2022 | シンプルなAPI、更新を処理 |

**検討すべき新しいツール/パターン:**
- **WebGPU:** 来ているがゲーム向けはまだ本番準備ができていない（2025）
- **drei Gltfヘルパー:** ローディング画面用の<useGLTF.preload>

**非推奨/古くなったもの:**
- **cannon.js（オリジナル）:** cannon-esフォークか、より良いRapierを使用
- **物理用の手動レイキャスト:** Rapierコライダーを使うだけ
</sota_updates>

<sources>
## ソース

### プライマリ（HIGH信頼度）
- /pmndrs/react-three-fiber - 入門、フック、パフォーマンス
- /pmndrs/drei - インスタンス、コントロール、ヘルパー
- /dimforge/rapier-js - 物理セットアップ、車両物理

### セカンダリ（MEDIUM信頼度）
- Three.js discourse「都市ドライビングゲーム」スレッド - ドキュメントに対してパターンを検証
- R3F examplesリポジトリ - コードの動作を検証

### ターシャリ（LOW信頼度 - 要検証）
- なし - すべての発見を検証済み
</sources>

<metadata>
## メタデータ

**リサーチスコープ:**
- コア技術: Three.js + React Three Fiber
- エコシステム: Rapier、drei、zustand
- パターン: 車両物理、インスタンシング、都市生成
- 落とし穴: パフォーマンス、物理、手触り

**信頼度の内訳:**
- 標準スタック: HIGH - Context7で検証済み、広く使用
- アーキテクチャ: HIGH - 公式サンプルから
- 落とし穴: HIGH - discourseで文書化、ドキュメントで検証
- コード例: HIGH - Context7/公式ソースから

**リサーチ日:** 2025-01-20
**有効期限:** 2025-02-20（30日 - R3Fエコシステムは安定）
</metadata>

---

*Phase: 03-city-driving*
*Research completed: 2025-01-20*
*Ready for planning: yes*
```

---

## ガイドライン

**作成するタイミング:**
- ニッチ/複雑なドメインのフェーズ計画前
- Claudeのトレーニングデータが古いかまばらな可能性がある場合
- 「どのライブラリ」よりも「専門家はどうやるか」が重要な場合

**構造:**
- セクションマーカーにXMLタグを使用（GSDテンプレートに合わせる）
- 7つのコアセクション: summary、standard_stack、architecture_patterns、dont_hand_roll、common_pitfalls、code_examples、sources
- すべてのセクションが必須（包括的なリサーチを推進）

**コンテンツの品質:**
- 標準スタック: 名前だけでなく具体的なバージョン
- アーキテクチャ: 権威あるソースからの実際のコード例を含める
- 自作しない: どの問題を自分で解決しないかを明示
- 落とし穴: 「これをするな」だけでなく警告サインを含める
- ソース: 信頼度レベルを正直にマーク

**計画との統合:**
- RESEARCH.mdはPLAN.mdの@context参照として読み込まれる
- 標準スタックがライブラリ選択に情報提供
- 自作しないがカスタムソリューションを防止
- 落とし穴が検証基準に情報提供
- コード例がタスクアクション内で参照可能

**作成後:**
- ファイルはフェーズディレクトリに配置: `.planning/phases/XX-name/{phase_num}-RESEARCH.md`
- 計画ワークフロー中に参照
- plan-phaseが存在する場合に自動的に読み込み
