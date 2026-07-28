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
| `enterprise` | 32/32 | Comprehensive | 規制・フル監査・本番級運用 |
| `feature` | 32/32 | Standard | 既定。新機能全般 |
| `mvp` | 22/32 | Standard | グリーンフィールド MVP（下表のスキップ） |
| `poc` | 8/32 | Minimal | 実現可能性の迅速検証 |
| `bugfix` | 7/32 | Minimal | 特定バグ修正 |
| `refactor` | 8/32 | Minimal | 振る舞いを変えない整理 |
| `infra` | 13/32 | Standard | インフラ・環境・IaC |
| `security-patch` | 10/32 | Minimal | CVE 等の迅速対応 |
| `workshop` | 25/32 | Standard（**Test=Minimal**） | 研修。Ideation 全スキップ |

### `mvp` のスキップ内訳（公式 05-scopes-and-depth）

**10 SKIP / 22 EXECUTE**:

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

---

## 5.3 Adaptive Composer

在庫スコープが合わないとき:

```
/aidlc compose "..."
```

- エントロピー内訳付きの提案
- ステージ毎 EXECUTE/SKIP 理由
- 承認後にカスタム scope が永続化し、以降 `--scope <name>` で再利用可能

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
