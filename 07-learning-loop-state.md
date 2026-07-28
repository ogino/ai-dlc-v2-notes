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

**Human presence**: 承認ゲートでは、モデルが単独で承認を確定できないよう、人間ターン（`HUMAN_TURN` 等）が要求される設計。監査上も人間の関与が残る。

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
- 結果は監査行（現状 advisory 中心）
- 独自センサーを manifest で追加可能

---

## 7.5 State と Audit

### aidlc-state.md

- スコープ・深度・テスト戦略
- ステージ進捗チェックボックス（6 状態）
- Construction Autonomy Mode
- セッション再開情報

### Audit（74 イベント種別）

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
