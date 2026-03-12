<purpose>
GSDエージェントが使用するモデルプロファイルを切り替える。各エージェントが使用するClaudeモデルを制御し、品質とトークン消費のバランスを調整する。
</purpose>

<required_reading>
開始前に、呼び出しプロンプトのexecution_contextで参照されているすべてのファイルを読み込むこと。
</required_reading>

<process>

<step name="validate">
引数を検証する：

```
if $ARGUMENTS.profile not in ["quality", "balanced", "budget"]:
  Error: 無効なプロファイル "$ARGUMENTS.profile"
  有効なプロファイル: quality, balanced, budget
  EXIT
```
</step>

<step name="ensure_and_load_config">
設定が存在することを確認し、現在の状態を読み込む：

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-ensure-section
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

`.planning/config.json`が存在しない場合はデフォルトで作成し、現在の設定を読み込む。
</step>

<step name="update_config">
state loadまたは直接から現在の設定を読み込む：

`model_profile`フィールドを更新：
```json
{
  "model_profile": "$ARGUMENTS.profile"
}
```

更新された設定を`.planning/config.json`に書き戻す。
</step>

<step name="confirm">
選択されたプロファイルのモデルテーブルとともに確認を表示する：

```
✓ モデルプロファイルを設定しました: $ARGUMENTS.profile

エージェントは以下を使用する:

[選択されたプロファイルのgsd-tools.cjs内MODEL_PROFILESからテーブルを表示]

例:
| エージェント | モデル |
|-------|-------|
| gsd-planner | opus |
| gsd-executor | sonnet |
| gsd-verifier | haiku |
| ... | ... |

次に生成されるエージェントから新しいプロファイルが使用される。
```

プロファイル名のマッピング：
- quality: MODEL_PROFILESの"quality"カラムを使用
- balanced: MODEL_PROFILESの"balanced"カラムを使用
- budget: MODEL_PROFILESの"budget"カラムを使用
</step>

</process>

<success_criteria>
- [ ] 引数が検証された
- [ ] 設定ファイルの存在が確認された
- [ ] 設定が新しいmodel_profileで更新された
- [ ] モデルテーブルとともに確認が表示された
</success_criteria>
