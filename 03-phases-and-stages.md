# 03. フェーズとステージ（5 × 33）

## 3.1 全体フロー

```
INITIALIZATION (0.1–0.3)  ──auto（ゲートなし）──►  IDEATION (1.1–1.7)
                                                      │ Verification Gate 1（自動検査）
                                                      ▼
                                                 INCEPTION (2.1–2.9)
                                                      │ Verification Gate 2（自動検査）
                                                      ▼
                                              CONSTRUCTION
                                                3.1–3.5 = Unit 単位（→ 3.5.1）
                                                3.6–3.7 = Construction の全 Unit 完了後に 1 回
                                                      │ Verification Gate 3（自動検査）
                                                      ▼
                                                OPERATION (4.1–4.7)
                                                      │
                                      complete ◄──────┴──► 新サイクルへ Ideation 1.1
```

フェーズ境界（Init→Ideation を除く）で **Verification Gate** がトレース可能性を **自動**検査する（問題があれば続行／戻るを人が判断）。

---

## 3.2 Phase 0: Initialization（自動・ゲートなし）

目的: ワークスペースのブートストラップ。  
実行: `aidlc-utility intent-create` 内の決定論処理（LLM 委任なし、1 秒未満）。
※ 2.5.57 で `intent-birth` から改名。**旧名はエラーにならず no-op**。

| # | ステージ | 成果 |
|---|----------|------|
| 0.1 | Workspace Scaffold | Intent レコード dir |
| 0.2 | Workspace Detection | 言語・マニフェスト等の規則ベース検出 |
| 0.3 | State Initialization | `aidlc-state.md` / audit shards |

---

## 3.3 Phase 1: Ideation（イニシアチブ妥当性）

目的: 意図の捕捉、実現可能性、スコープ、チーム、承認。

| # | ステージ | Lead | 条件の目安 |
|---|----------|------|------------|
| 1.1 | Intent Capture & Framing | product | ALWAYS |
| 1.2 | Market Research | product | CONDITIONAL |
| 1.3 | Feasibility & Constraints | architect | CONDITIONAL |
| 1.4 | Scope Definition | product | ALWAYS |
| 1.5 | Team Formation | delivery | CONDITIONAL |
| 1.6 | Rough Mockups | design | CONDITIONAL（UI） |
| 1.7 | Approval & Handoff | delivery | ALWAYS |

**2.5.11 の強化**: Intent Capture の claim はソースタグ必須。未根拠の主張は仮定として表面化し、人間の明示受諾が必要。`claim-sources` センサーが検査。

---

## 3.4 Phase 2: Inception（要件精緻化）

目的: コードベース分析、要件、設計、Unit 分解、契約定義、納品計画。

| # | ステージ | Lead | mode / 特記 |
|---|----------|------|-------------|
| 2.1 | Reverse Engineering | developer → architect | `mode: pipeline`（2-link）。Brownfield のみ |
| 2.2 | Practices Discovery | pipeline-deploy | `mode: subagent`（**hub-and-spoke 形状**）。肯定後に memory へ promote |
| 2.3 | Requirements Analysis | product | ALWAYS（inline） |
| 2.4 | User Stories | product | `mode: mob`（design/developer/quality） |
| 2.5 | Refined Mockups | design | UI（CONDITIONAL） |
| 2.6 | Domain Design | architect | 実行計画による（CONDITIONAL）。`components.md` / `decisions.md`（ADR） |
| 2.7 | Units Generation | architect | ALWAYS（DAG） |
| 2.8 | Contract Design | architect | CONDITIONAL。`contract-summary.md` |
| 2.9 | Delivery Planning | delivery | ALWAYS（`bolt-plan.md` 等） |

**2.6.1 の Inception 再編**: 2.6 は `application-design` から **`domain-design`** へ改名（ステージファイル自体が置き換えられており、旧名はステージとして存在しない）。設計成果物も 5 本（`components` / `component-methods` / `services` / `component-dependency` / `decisions`）から **`components` / `decisions` の 2 本**へ整理された（ほかに共通の `traceability` を産出）。整理された 3 本のうち **`component-methods` は廃止**であって改名ではない点に注意。

**2.8 Contract Design（新設）**: Unit をまたぐ境界と外部公開 API を**先に固定して並行開発を可能にする**ステージ。Construction の設計 3 段と違い **per-unit ではなく、ワークフローにつき 1 回**走り、Units Generation の依存 DAG を使って境界を一括で洗い出す。成果物は `contract-summary.md` 1 本のみで、境界ごとの仕様（OpenAPI / AsyncAPI / 共有スキーマ）をフェンス済みブロックとして同一ファイルに埋め込む。単一 Unit で境界も外部公開 API も無い場合のみスキップされる。この新設に伴い Delivery Planning は 2.8 → **2.9** へ繰り下がった。

---

## 3.5 Phase 3: Construction（構築）

### なぜ反復（Bolt）か

- 旧: ステージ毎 × Unit 毎ゲート → 「子守（babysitting）」
- 一括生成: 一度に巨大 diff → レビュー不能
- **現在: 反復単位**（中間）

### 実行モデル

1. **最初のスライス = Walking Skeleton**（常にゲート付き・対話）  
   最小の端到端スライスでアーキを証明
2. **Ladder prompt（1 回だけ）**  
   以降を autonomous にするか、反復ごとにゲートを置くか
3. 残りを実行（依存が揃えば **parallel batch**）
4. **3.6 Build and Test** を全体で 1 回
5. **3.7 CI Pipeline**（条件付き）を 1 回

| # | ステージ | Lead | 実行単位 | 条件 |
|---|----------|------|----------|------|
| 3.1 | Functional Design | architect | Per Unit | **CONDITIONAL**（実行計画による） |
| 3.2 | NFR Requirements | architect | Per Unit | **CONDITIONAL** |
| 3.3 | NFR Design | architect | Per Unit | **CONDITIONAL** |
| 3.4 | Infrastructure Design | aws-platform | Per Unit | **CONDITIONAL** |
| 3.5 | Code Generation | developer | Per Unit | **ALWAYS** |
| 3.6 | Build and Test | quality | 全 Unit 後に **1 回** | ALWAYS |
| 3.7 | CI Pipeline | pipeline-deploy | 全 Unit 後に **1 回** | CONDITIONAL |

失敗時: autonomous でも **halt-and-ask**（retry / skip / abort）。  
反復に入っても 3.1–3.4 が常に全部走るわけではない。

### 3.5.1 「Bolt」の語義は 2.6.86 で上流が再定義した

**本ノートの旧記述が間違っていたのではなく、上流グロッサリの定義が変わった。**
本節はかつて「3.1–3.5 = Bolt 単位」「実行単位 = Per Bolt」と書いていたが、
2.6.86 で Bolt は**実行の単位ではなくなった**。

| | 旧（2.6.55 まで） | 新（2.6.86 以降） |
|---|---|---|
| Bolt | 「**Construction 実行の単位** — 1 Unit に対する 3.1–3.5 の一巡」 | 「**スプリント様の Construction 反復**。Delivery Planning (2.9) が意図したグルーピングを記録する」 |
| `bolt-plan.md` | 実行の境界 | **計画上の記録**。既定の stage-major ランタイムは**グルーピング／順序の境界として消費しない** |
| 実行のバッチ | Bolt から | `unit-of-work-dependency.md`（2.7）から**再計算**される |
| `BOLT_STARTED` / `BOLT_COMPLETED` | Construction 一般 | **swarm / worktree 経路の Unit 単位イベント**。通常のゲート付き実行では**記録されない** |

**挙動は変わっていない。** 2.6.86 が触った 69 ファイルはドキュメントとプロトコル散文の整合のみで、
ツール・フック・ステージグラフの変更はゼロである。
→ [01.8.1](./01-overview.md) / [15-release-impact-26123.md](./15-release-impact-26123.md)

---

## 3.6 Phase 4: Operation（運用）

全 7 ステージが CONDITIONAL。`mvp` / `poc` / `bugfix` / `refactor` 等では Operation が丸ごと省略され得る。  
（`mvp` はさらに Ideation の Market Research / Team Formation / Approval & Handoff もスキップ → 合計 10 SKIP / 23 EXECUTE。詳細は [05](./05-scopes-depth-test.md)。）

| # | ステージ | Lead |
|---|----------|------|
| 4.1 | Deployment Pipeline | pipeline-deploy |
| 4.2 | Environment Provisioning | aws-platform |
| 4.3 | Deployment Execution | pipeline-deploy |
| 4.4 | Observability Setup | operations |
| 4.5 | Incident Response | operations |
| 4.6 | Performance Validation | quality |
| 4.7 | Feedback & Optimization | operations（終端。または Ideation へ帰還） |

---

## 3.7 実行モード集計

公式ガイドのトポロジ集計（**33 ステージ全体**。Init 0.1–0.3 も mode 分類上は inline 側に含まれる）:

| Mode（正式値） | 数 | 備考 |
|----------------|-----|------|
| Inline | 29 | Init の決定論 3 段を含む |
| Subagent | 2 | Practices Discovery（hub-and-spoke 形状）+ Code Generation |
| Pipeline | 1 | Reverse Engineering |
| Mob | 1 | User Stories |
| **合計** | **33** | |

---

## 3.8 アーティファクトの考え方

- 原則として Intent の **record dir** 配下に Markdown 等で蓄積
- **例外: Reverse Engineering（2.1）の 9 成果物**は record dir ではなく **space レベルの codekb ストア** `aidlc/spaces/<space>/codekb/<repo>/` に書かれる。record dir に残るのは同ステージの `memory.md` 日誌のみ（2.5.26 で scaffold とドキュメントが是正された）
  - codekb はリポジトリごとに 1 つの共有ビューで、brownfield の再実行が上書きする（last write wins）。2.5.35 以降はスキャン範囲を記録・比較し、狭いスキャンでの無警告上書きを防ぐ
- ステージ間は `consumes` / `produces` でグラフ化
- 上流引用は `upstream-coverage` センサーが advisory 検査
- Construction の並行は contribution ファイルと worktree / swarm 機構で証拠化
