# 05. スコープ・深度・テスト戦略

3 軸は独立に調整できる。

| 軸 | 制御対象 |
|----|----------|
| **Scope** | **どのステージを実行するか** |
| **Depth** | **各ステージの成果物の詳細度** |
| **Test strategy** | **テストの量・種類**（Depth と独立） |

---

## 5.1 9 コア・スコープ

| Scope | ステージ数 | 既定 Depth | 用途 |
|-------|-----------|------------|------|
| `enterprise` | 33/33 | Comprehensive | 規制・フル監査・本番級運用 |
| `feature` | 33/33 | Standard | 既定。新機能全般 |
| `mvp` | 23/33 | Standard | グリーンフィールド MVP（下表のスキップ） |
| `poc` | 8/33 | Minimal | 実現可能性の迅速検証 |
| `bugfix` | 7/33 | Minimal | 特定バグ修正 |
| `refactor` | 8/33 | Minimal | 振る舞いを変えない整理 |
| `infra` | 13/33 | Standard | インフラ・環境・IaC |
| `security-patch` | 10/33 | Minimal | CVE 等の迅速対応 |
| `workshop` | 26/33 | Standard（**Test=Minimal**） | 研修。Ideation 全スキップ |

**分母が 33 になった理由**: 2.6.1 で Inception に `contract-design`（2.8）が新設され、全体が 32 → 33 ステージになった。同ステージの `scopes:` は **`enterprise` / `feature` / `mvp` / `workshop` の 4 つのみ**なので、分子が +1 されるのはこの 4 行だけである（`poc` / `bugfix` / `refactor` / `infra` / `security-patch` は分子据え置きで分母のみ 32 → 33）。

### `mvp` のスキップ内訳（公式 05-scopes-and-depth）

**10 SKIP / 23 EXECUTE**:

- Operation **全 7**（4.1–4.7）
- Ideation: **Market Research（1.2）**, **Team Formation（1.5）**, **Approval & Handoff（1.7）**

「Operation だけ省略」ではない点に注意。

### ステージ数の体感

- `poc`: 公式 docs の説明どおり、**約 5 承認ゲート**（compiled grid 由来の概算／確認時表示）
- `feature`: **約 29 承認ゲート** + Construction の Unit 展開

起動時のスコープ確認行に **実数**（compiled grid から計算）が表示される。

---

## 5.2 自動検出

```
/aidlc Build a REST API for inventory management
```

キーワード例:

| 語 | Scope |
|----|-------|
| fix / bug / broken | bugfix |
| refactor / clean up | refactor |
| infrastructure / infra | infra |
| security / CVE | security-patch |
| poc / prototype | poc |
| mvp | mvp |
| workshop / training | workshop |
| その他 | feature（core 有効時） |

長文 + キーワード埋没時は **compose 提案**へ誘導（誤検出防止）。

**既定スコープ解決（2.5.30 以降）**: プラグイン選択でコアスコープ（`feature` / `poc`）を無効化した「プラグイン専用インストール」では、従来 `Unknown scope` エラーになり得た。scope の frontmatter に `freeform_default: true` を宣言すると、そのスコープが既定候補として指名される。**コアスコープを併用する通常構成では挙動は変わらない**（コアの 9 スコープはいずれも `freeform_default` を宣言していない）。

---

## 5.3 Adaptive Composer

在庫スコープが合わないとき:

```
/aidlc compose "..."
```

- エントロピー内訳付きの提案
- ステージ毎 EXECUTE/SKIP 理由
- 承認後にカスタム scope が永続化し、以降 `--scope <name>` で再利用可能

**在庫スコープ一致判定の一本化（2.5.64 以降）**: 最終提案が在庫スコープに一致するかの判定は **`aidlc-graph.ts validate-grid` の `nearest_stock`** に一本化された。従来の機械的 ARS 距離は advisory 扱いに降格し、証拠由来の fold を打ち消さない。`validate-grid` はコンパイル済み全ステージについて EXECUTE/SKIP の明示エントリを 1 件ずつ要求するため、**キーの欠落・余剰があるグリッドは在庫一致候補になれない**。また、一致した在庫プランを編集すると、その時点で **custom プランへ転換**して編集内容が永続化される（在庫スコープ側は書き換わらない）。

### ARS 演算の決定論化（2.5.25 以降）

複合スコアの算出は **`aidlc-graph.ts ars` サブコマンド**が担い、モデルは計算を行わない（CHANGELOG の表現は *"a model never does the multiplication"*）。

```
aidlc-graph.ts ars --iae <s> --csu <s> --ve <s> --r <s> --ua <s> \
  [--completed <csv>] [--project-type brownfield|greenfield]
```

出力は JSON で、加重合成値とその帯域ラベル、LOW/MED/HIGH の成分帯域、ステージ別の期待値スクリーン、近傍のストックスコープ、2 つのゲート表を返す。

パラメータは `core/tools/data/ars-priors.json`（schemaVersion 1）に外出しされている。

| 項目 | 値 |
|------|-----|
| 重み | IAE 0.20 / CSU 0.30 / VE 0.25 / R 0.15 / UA 0.10 |
| 成分帯域の境界 | LOW < 0.30 ≤ MED < 0.70 ≤ HIGH |

> **モデル依存が残る部分**: 決定論化されたのは「計算」であって「採点」ではない。5 成分（IAE/CSU/VE/R/UA）を証拠から評価してスコアを付ける工程は、引き続き Composer エージェントの判断による。

---

## 5.4 Depth（3）

| Level | 成果物 |
|-------|--------|
| Minimal | 必須のみ・短い文書 |
| Standard | バランス（多くの機能の既定） |
| Comprehensive | 企業級の網羅・コンプライアンス行列 |

上書き:

```
/aidlc --depth comprehensive
/aidlc --scope bugfix --depth standard
```

ゲート応答でも変更可。

---

## 5.5 Test Strategy（3）

| Level | 方針 |
|-------|------|
| **Minimal（Nyquist）** | 要件あたり 1 + コンポーネント最低 1 happy-path。主に unit。約 5–15 |
| **Standard** | コンポーネントあたり 5–8。unit+integration。ピラミッド ~75/20/5 |
| **Comprehensive** | 10–15/コンポーネント。unit+integration+E2E+perf/sec（NFR 依存） |

上書き:

```
/aidlc --test-strategy minimal
/aidlc --depth standard --test-strategy minimal
```

`workshop` だけ Depth=Standard でも Test=Minimal が既定（研修ペース維持）。

---

## 5.6 選択の目安

| 状況 | 推奨 |
|------|------|
| 本番アプリの新機能 | `feature` |
| ゼロからの製品 | `mvp` or `feature` |
| 方針の素早い検証 | `poc` |
| 既知バグ | `bugfix` |
| 振る舞い不変の整理 | `refactor` |
| AWS 環境 / CDK | `infra` |
| CVE | `security-patch` |
| 規制対応機能 | `enterprise` |
| 研修 | `workshop` |

迷ったら `feature`（CONDITIONAL ステージは条件不成立でスキップ）。
