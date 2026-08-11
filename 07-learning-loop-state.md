# 07. Space / Intent・Rules・Sensors・監査

## 7.1 Space と Intent

| 概念 | 意味 |
|------|------|
| **Intent** | 1 つの作業単位＝1 回のライフサイクル実行 |
| **Space** | チーム世界（memory / knowledge / codekb / intents） |
| **Record dir** | `aidlc/spaces/<space>/intents/<YYMMDD>-<label>/` |

既定 Space は `default`。ソロ／単一チームは Space を意識しなくてよい。

### Intent の誕生

```
/aidlc Build a REST API for inventory management
```

→ 自動 birth。別作業と判定されると **第二 Intent を開始するか**を確認（勝手に並行開始しない）。

```
/aidlc intent                 # 一覧
/aidlc intent <slug>          # 切替（例: export-bug は intent のラベル／スラッグ）
/aidlc space create teamB
/aidlc space switch teamB
```

**Human presence**: 承認ゲートでは、モデルが単独で承認を確定できないよう、人間ターン（`HUMAN_TURN`）が要求される。UserPromptSubmit フックが人間の実プロンプトごとに `HUMAN_TURN` を監査台帳へ追記し、承認処理は「前回のゲート解決以降に `HUMAN_TURN` があること」を検査して無ければ拒否する。**プロンプト契約ではなくエンジンによる機械的強制**であり、監査上も人間の関与が残る。

> 例外が 3 つある。①Construction の autonomous モードは設計上 presence を要求しない ②監査台帳が空の場合は fail-open ③環境変数 `AIDLC_SKIP_HUMAN_PRESENCE_GUARD=1` でガード自体を無効化できる（テスト用スイッチだが配布物に同梱）。
>
> なお presence 検査が働くのは**承認経路**であって、CONDITIONAL ステージのスキップ報告（`report --result skipped`）には presence 検査が無い。スキップ判定の妥当性は導体の判断に委ねられる。

### 任意の workspace manifest（2.5.36 以降）

複数リポジトリを扱うチーム向けに、workspace ルートへ `repos.json`（`{org, repos:[{name, branch?, url?}]}`）を置いて期待するリポジトリ集合を宣言できる。ただし**実行時の正はあくまでディスク上の自動検出**であり、manifest は `aidlc-workspace-sync` が不足リポジトリを clone したり `.gitignore` を整えたりするための補助レイヤーにすぎない（宣言の有無にかかわらず、clone された時点で動く）。

### コミット方針

| コミットする | gitignore |
|--------------|-----------|
| memory / knowledge / codekb / intents.json / state / audit / 成果物 | active-space, active-intent, runtime-graph, セッション一時 |

---

## 7.2 Rules（フィードフォワード）

場所: `aidlc/spaces/<active-space>/memory/`

```
org.md          # フレームワーク＋組織既定
team.md         # チームで肯定した実践
project.md      # このプロジェクト特有
phases/
  ideation.md
  inception.md
  construction.md
  operation.md
```

解決チェーン（**strict-additive**）:

```
org → team → project → phase → (stage: 将来)
```

- 上書きではなく **すべてが文脈に載る**
- ワークフロー開始時に 1 回コンパイル

**配信の決定論化（2.5.33 以降）**: コンパイルが開始時 1 回である点は変わらないが、コンパイル済みのルール束を各ステージへ届ける仕組みが変わった。従来は導体がパスを読みに行くかどうかに依存しており、ルールが一切適用されないままステージが走る事故が観測されていた。現在はエンジンが `run-stage` の前に **`load-steering` ディレクティブ**として `{path, text}` の順序付きコンテンツを配信する。

- ディレクティブは **28 KiB 上限**。超過分は Markdown 見出し → UTF-8 境界で分割され、継続トークン（machine-local 鍵による署名付き）で連結する
- 必須ルールが読めない・不正な UTF-8 の場合はステージ着手前に停止し、修復手順を提示する
- 任意ファイルの欠落は `context_warnings` として報告し、ステージはブロックしない
- **対象はルール（`memory/*.md`）のみ**。ペルソナと知識ファイルは引き続きパス参照（path-loaded）で、28 KiB 制約の対象外

---

## 7.3 Learning Loop

1. ステージ中: エージェントが `memory.md` に  
   Interpretations / Deviations / Tradeoffs / Open questions を記録
2. 承認ゲート前: 決定論ツールが候補を提示 + 「次回向けに追加は？」
3. 人間が保持する項目を選択
4. project.md（または promote で team.md）へ書き込み
5. **現行ワークフローのルール集合は不変**。次回から有効

例: 「transaction = DB ではなく銀行取引」という訂正を project ルール化。

---

## 7.4 Sensors（フィードバック）

| Sensor | 検査内容 |
|--------|----------|
| **claim-sources** | Intent Capture の主張にソースタグがあるか |
| **required-sections** | 必須 H2 見出し |
| **upstream-coverage** | 上流成果物への参照 |
| **linter** | ESLint 等（コード） |
| **type-check** | tsc 等 |

- Write/Edit 時に hook で発火
- 結果は監査行（**現行の受理値は `advisory` のみ**。マニフェストのフィールド名は `severity` ではなく `default_severity`。`blocking` は将来の "v0.10.0 ralph driver" 向けに予約）
- 独自センサーを manifest で追加可能

**堅牢化（2.5.31 / 2.5.32）**

- 2.5.31: センサー dispatcher と linter センサーが、sibling リポジトリのパッケージマネージャが吐く先頭バナー等の stdout ノイズを読み飛ばすようになった。以前は前置ノイズで JSON パースが失敗し、実際の FAILED が `script-error: bad-output` として握りつぶされていた
- 2.5.32: プラグインの `sensors/` マニフェストが命名規約（`sensors/` 直下の `aidlc-<id>.md`）を満たさず発見不能な場合、compose 時に degraded drop として記録され `/aidlc --doctor` に表示される。**ただしこの検証は Plugin 経由のみ**で、`sensors/` へ直接置いたセンサーは依然として命名不一致が黙って無視される

> **fail-open 設計に注意**: センサーの判定で FAILED になるのは「終了コード 0 かつ JSON の `pass === false`」の 1 経路のみ。ツール未インストール（exit 127）、spawn 失敗、異常終了、不正 JSON、`pass` フィールド欠如は**いずれも PASSED として記録される**（`Note:` に理由が残る）。監査証跡上の PASSED は「チェックが通った」ことを必ずしも意味しない。

---

## 7.5 State と Audit

### aidlc-state.md

- スコープ・深度・テスト戦略
- ステージ進捗チェックボックス（6 状態）
- Construction Autonomy Mode
- セッション再開情報

### Audit（82 イベント種別・21 分類）

- `audit/` 配下のシャードをタイムスタンプマージ
- 例: STAGE_*, QUESTION_ANSWERED, REVIEW_*, SENSOR_*, RULE_LEARNED, RECOMPOSED, PLUGIN_SELECTION_CHANGED, ...

### Session resume

- チェックポイントから継続
- redo / jump / start fresh
- Compaction 耐性（recovery breadcrumb 等）

---

## 7.6 Knowledge の 2 層

| 層 | 内容 |
|----|------|
| **Methodology knowledge** | フレームワーク同梱（`core/knowledge/` → dist） |
| **Team knowledge** | ユーザー管理（`aidlc/spaces/.../knowledge/`） |

Rules が「制約」、Knowledge が「情報」という切り分け。
