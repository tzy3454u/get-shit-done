# モデルプロファイルの解決

オーケストレーション開始時にモデルプロファイルを一度解決し、すべてのTaskスポーンに使用します。

## 解決パターン

```bash
MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | grep -o '"model_profile"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "balanced")
```

デフォルト: 設定されていないかconfigが見つからない場合は`balanced`。

## ルックアップテーブル

@~/.claude/get-shit-done/references/model-profiles.md

解決されたプロファイルのテーブルでエージェントを検索します。Task呼び出しにmodelパラメータを渡します:

```
Task(
  prompt="...",
  subagent_type="gsd-planner",
  model="{resolved_model}"  # "inherit", "sonnet", or "haiku"
)
```

**注意:** Opusティアのエージェントは`"opus"`ではなく`"inherit"`に解決されます。これにより、エージェントは親セッションのモデルを使用し、特定のopusバージョンをブロックする可能性のある組織ポリシーとの競合を回避します。

## 使い方

1. オーケストレーション開始時に一度解決する
2. プロファイル値を保存する
3. スポーン時に各エージェントのモデルをテーブルから検索する
4. 各Task呼び出しにmodelパラメータを渡す（値: `"inherit"`, `"sonnet"`, `"haiku"`）
