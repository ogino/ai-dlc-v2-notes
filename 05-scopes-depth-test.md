# 05. スコープ・深度・テスト戦略

3 軸は独立に調整できる。

| 軸 | 制御対象 |
|----|----------|
| **Scope** | **どのステージを実行するか** |
| **Depth** | **各ステージの成果物の詳細度** |
| **Test strategy** | **テストの量・種類**（Depth と独立） |

---

## 5.1 11 コア・スコープ

| Scope | ステージ数 | 既定 Depth | 用途 |
|-------|-----------|------------|------|
| `enterprise` | 33/33 | Comprehensive | 規制・フル監査・本番級運用 |
| `feature` | 33/33 | Standard | 新機能全般（全ステージ実行）。**2.6.18 で暗黙の既定の座を `classic` に譲った** |
| `classic` | 26/33 | Standard | **2.6.18 追加。暗黙の既定スコープ。** AI-DLC v1 相当のライフサイクル（Ideation 全 7 本をスキップ） |
| `mvp` | 23/33 | Standard | グリーンフィールド MVP（下表のスキップ） |
| `poc` | 8/33 | Minimal | 実現可能性の迅速検証 |
| `bugfix` | 7/33 | Minimal | 特定バグ修正 |
| `refactor` | 8/33 | Minimal | 振る舞いを変えない整理 |
| `infra` | 13/33 | Standard | インフラ・環境・IaC |
| `security-patch` | 10/33 | Minimal | CVE 等の迅速対応 |
| `workshop` | 26/33 | Standard（**Test=Minimal**） | 研修。Ideation 全スキップ |
| `express` | 10/33 | Minimal | **2.6.18 追加。** 要件 → コード → テスト → 条件付きデプロイの最短路。設計パス無し・**reviewer 無効**（→ 5.7） |

（この節を含む 2.6.2 → 2.6.49 の差分全体は [13-release-impact-2649.md](./13-release-impact-2649.md) を参照）

**2.6.18 でスコープが 9 → 11 になった**（出典: `core/scopes/*.md` の件数 / `scope-grid.json` の `scopes` キー数）。追加は `classic` と `express` の 2 つで、**既存 9 スコープの EXECUTE 数はすべて不変**である（`scope-grid.json` と 33 ステージの frontmatter `scopes:` を独立集計した 2 系統で一致）。ステージ 33 / フェーズ 5 / エージェント 14 / センサー 6 / ハーネス 7 も不変で、変わったのは「各ステージがどのスコープに属するか」だけである。

**分母が 33 になった理由**: 2.6.1 で Inception に `contract-design`（2.8）が新設され、全体が 32 → 33 ステージになった。同ステージの `scopes:` は **`enterprise` / `feature` / `mvp` / `workshop` の 4 つのみ**なので、分子が +1 されるのはこの 4 行だけである（`poc` / `bugfix` / `refactor` / `infra` / `security-patch` は分子据え置きで分母のみ 32 → 33）。

### `classic` のスキップ内訳（2.6.18 追加）

**7 SKIP / 26 EXECUTE**: **Ideation の 1.1–1.7 の 7 本すべて**。Inception 以降は全ステージがグリッドに入る。

`core/scopes/aidlc-classic.md` は「AI-DLC v1 had no Ideation phase, so `classic` skips all seven Ideation stages and keeps every stage from Inception onward in the plan」と説明している。無条件（ALWAYS）なのは Initialization 3 本 + Requirements Analysis / Units Generation / Delivery Planning / Code Generation / Build and Test の計 8 本で、残る Inception 設計群と Operation 末尾は CONDITIONAL として文脈から自己選択する。

test strategy は depth から継承して **Standard**（`workshop` と違って Minimal 上書きを持たない）。「26/33」という分子は `workshop` と同じだが、**同じステージ集合ではない**点に注意（`workshop` は Ideation を落として Test=Minimal、`classic` は Ideation を落として Test=Standard）。

### `express` のスキップ内訳（2.6.18 追加）

**23 SKIP / 10 EXECUTE**。グリッドに入るのは Initialization 3 本 + Reverse Engineering / Requirements Analysis / Code Generation / Build and Test / Deployment Pipeline / Deployment Execution / Observability Setup。Ideation・Inception 設計群・Units Generation・Delivery Planning は全部 SKIP。

`core/scopes/aidlc-express.md` は「The swarm path is structurally unreachable because `express` skips Units Generation, so no Unit DAG can exist」と書いており、**swarm 経路は構造的に到達不能**である（無効化フラグではなく、前提となる Unit DAG が生成されないため）。

### 専用スキルランナー（`runner: true`）

`runner: true` を frontmatter に持つ＝ `/aidlc-<scope>` の一語ショートカットが生成されるのは、**`feature` / `mvp` / `bugfix` / `security-patch` / `express` の 5 つ**である（出典: `core/scopes/aidlc-*.md` の frontmatter）。

**`classic` と `workshop` には専用ランナーが無い。** つまり `classic` は暗黙の既定でありながら、明示的に選びたいときは `--scope classic` と書く必要がある（`/aidlc-classic` は存在しない）。

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
| workshop / lab / training | workshop |
| **express / lightweight** | **express**（2.6.18 追加） |
| （キーワード無し） | **`classic`**（resolver の既定。→ 5.2.1） |

`classic` と `feature` は **`keywords: []`** で、**キーワードによる自動検出の対象ではない**（`--scope` で明示指定するか、`classic` は既定として選ばれるかのいずれか）。`workshop` の `lab` / `training` は 2.6.2 時点で既に存在していたキーワードで、上表の旧版で `lab` が漏れていたのを補った（2.6.49 での変更ではない）。

長文 + キーワード埋没時は **compose 提案**へ誘導（誤検出防止）。

### 5.2.1 暗黙の既定スコープ: `feature` → `classic`（2.6.18）

2.6.18 は CHANGELOG が自ら **declared behavior change** と書いた変更で、今回の更新で利用者に最も見えやすい差分である。**resolver レベルと UX レベルを必ず分けて読むこと。**

**(a) resolver レベル（コード実測）**

| 版 | 定義位置 | 値 |
|---|---|---|
| 2.6.2 | `core/tools/aidlc-orchestrate.ts:191`（ローカル定数） | `const DEFAULT_SCOPE = "feature"` |
| 2.6.49 | `core/tools/aidlc-lib.ts:8847`（export） | `export const DEFAULT_SCOPE = "classic"` |

定数がオーケストレータ内のローカル定義から共有ライブラリの export へ移り、値が変わった。CHANGELOG によれば、engine の scope-resolution fallback、`/aidlc-init`、そして `--scope` 無しの低レベル `intent-create`（**従来は `poc` だった**）が、すべて `AWS_AIDLC_DEFAULT_SCOPE` を先に見て、無ければ `classic` に落ちる形に**統一された**。明示 `--scope`・キーワード検出・compose 提案は影響を受けない。

**(b) UX レベル（上流散文）**

対話的に `/aidlc <説明文>` を打った場合、キーワードに当たらなければ **compose 提案が先に提示され、ワークフローが黙って生成されることはない**。上流 `docs/guide/05-scopes-and-depth.md` の原文:

> In the user-facing cold-start flow, rich prose, no keyword hit, or a keyword buried in a long description enters the compose offer before a workflow is created; **it does not silently start Feature.**

**⚠ この 2 つを混同して「黙って `classic` が始まるようになった」と書くのは誤り。** `classic` が静かに使われるのは、compose 提案を経由しない低レベル呼び出し（`/aidlc-init`、`intent-create` 等の明示的フォールバック経路）に限られる。

### 5.2.2 `AWS_AIDLC_DEFAULT_SCOPE` の出荷値（7 ハーネス個別実測）

| ハーネス | 出荷設定ファイル | 2.6.2 | 2.6.49 |
|---|---|---|---|
| Claude Code | `dist/claude/.claude/settings.json` / `harness/claude/settings.json` | `"workshop"` | **`"classic"`** |
| codex | **該当ファイルを出荷していない** | 未設定 | 未設定 |
| copilot | **該当ファイルを出荷していない** | 未設定 | 未設定 |
| cursor | **該当ファイルを出荷していない** | 未設定 | 未設定 |
| kiro（CLI） | **該当ファイルを出荷していない** | 未設定 | 未設定 |
| kiro-ide | **該当ファイルを出荷していない** | 未設定 | 未設定 |
| opencode | **該当ファイルを出荷していない** | 未設定 | 未設定 |

**この非対称性が実効的な意味を左右する。** 環境変数はハードコード既定より優先されるので、Claude Code 利用者にとっては出荷 `settings.json` が**常にハードコード既定を上書きしている**。したがって Claude 利用者から見た実効的な変化は「ハードコード既定が `feature` → `classic` になったこと」ではなく、**「出荷ショートカット値が `workshop` → `classic` に変わったこと」**である。残り 6 ハーネスの利用者は、自分で環境変数を設定しない限り、ハードコード既定（新: `classic`、旧: `feature`）に到達する。

旧来の全ライフサイクル既定に戻したい場合は `AWS_AIDLC_DEFAULT_SCOPE=feature` を設定する（Claude Code なら `.claude/settings.json` の `env` ブロック。上流 CHANGELOG の Upgrade 文がこの手順を明記している）。

**既定スコープ解決（2.5.30 以降）**: プラグイン選択でコアスコープ（`feature` / `poc`）を無効化した「プラグイン専用インストール」では、従来 `Unknown scope` エラーになり得た。scope の frontmatter に `freeform_default: true` を宣言すると、そのスコープが既定候補として指名される。**コアスコープを併用する通常構成では挙動は変わらない**（コアの 11 スコープはいずれも `freeform_default` を宣言していない）。

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
| **Minimal（Nyquist）** | 要件あたり 1 + コンポーネント最低 1 happy-path。**既定は unit だが例外あり（下記）**。約 5–15 |
| **Standard** | コンポーネントあたり 5–8。unit+integration。ピラミッド ~75/20/5 |
| **Comprehensive** | 10–15/コンポーネント。unit+integration+E2E+perf/sec（NFR 依存） |

**Minimal のテスト種別が緩和された**: 従来は「単体テストのみ。integration / E2E / performance / security はスキップ」と読める書き方だったが、上流 `docs/guide/05-scopes-and-depth.md` の現行記述は次のとおり:

> **Unit tests by default.** A bugfix/security-patch targeted regression may use integration or E2E when that is the narrowest level that reproduces the defect; unrelated test volume remains Minimal.

つまり **`bugfix` / `security-patch` の的を絞った回帰テストに限り、その欠陥を再現できる最も狭いレベルとして integration や E2E を使ってよい**。緩和されたのは「種別の許容」だけで、**無関係なテスト量は Minimal のまま**である（総本数の目安 5–15 は変わらない）。

上書き:

```
/aidlc --test-strategy minimal
/aidlc --depth standard --test-strategy minimal
```

`workshop` だけ Depth=Standard でも Test=Minimal が既定（研修ペース維持）。**`classic` は `workshop` と同じ 26/33 だが Test 上書きを持たない**ため、depth から Standard を継承して本番級のテスト水準が効く。`express` は depth=Minimal を継承して Test=Minimal。

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
| Ideation を明示的に飛ばしたい / v1 相当の進行 | `classic`（**`--scope classic` と明示。専用ランナー無し**） |
| 要件からデプロイまでを最短で回したい | `express`（**reviewer 無効 → 5.7**） |

迷ったら `feature`（CONDITIONAL ステージは条件不成立でスキップ）。上流ガイドも「When in doubt, start with `feature` for backward-compatible full-lifecycle coverage; choose `classic` explicitly when you want to skip Ideation」と書いており、**既定が `classic` になった後も「迷ったら feature」の推奨は変わっていない**。

---

## 5.7 `review_cap`（スコープ単位のレビュアー制御）

scope frontmatter の `review_cap` が、そのスコープでレビュアーをどこまで走らせるかを決める。

| 値 | 2.6.2 の宣言スコープ | 2.6.49 の宣言スコープ |
|---|---|---|
| （未宣言） | enterprise / feature / mvp / infra / refactor / security-patch | 同左 |
| `advisory` | bugfix / poc / workshop | bugfix / poc / workshop / **classic** |
| **`none`** | **（存在しない値）** | **express のみ** |

出典: `core/scopes/aidlc-*.md` の frontmatter。

**`review_cap: none` は 2.6.49 時点で初めて登場した値である。** 2.6.2 では未宣言と `advisory` の 2 通りしかなく、**レビュアーを丸ごと無効化するスコープは存在しなかった**。

### ⚠ 静的グリッドの membership と実行時の dispatch は別物

reviewer 宣言（frontmatter の `reviewer:`）を持つステージは**両版とも 13 本で変化なし**。スコープ別の露出数は次のとおり（静的グリッド membership の集計）。

| スコープ | reviewer 宣言付きステージ数 | うち ALWAYS |
|---|---|---|
| enterprise / feature / mvp | 13 | 4 |
| classic（新） | 11 | 3 |
| workshop | 11 | 3 |
| infra | 4 | 1 |
| poc | 3 | 3 |
| refactor / security-patch | 3 | 2 |
| bugfix | 2 | 2 |
| **express（新）** | **2** | **2** |

**`express` は静的グリッド上は reviewer 宣言付きステージを 2 本（Requirements Analysis と Code Generation、いずれも ALWAYS）含むが、`review_cap: none` によって reviewer dispatch 自体が無効化される。** 上流 `core/scopes/aidlc-express.md` も「without a design pass or reviewer dispatch. Reviewers are disabled by `review_cap: none`」と書いている。**片方だけ書くと逆の印象になるので、membership と `review_cap` は必ずセットで読むこと。**

含意として、2.6.2 まではレビュアーを避ける手段が「reviewer 付きステージごとスキップする」しか無かったのに対し、2.6.49 では **`express` を選べばステージを実行したままレビュアーを止められる**経路が 1 本増えた。ただし `express` は 10/33 の最小構成であり、**「フル構成のままレビュアーだけ外す」ことができるようになったわけではない**。

---

## 5.8 読むときの注意（上流ドキュメントの stale とパーサのバグ）

### ⚠ 上流ドキュメント自身に「既定は feature」のままの箇所が残っている

既定スコープの記述について、**上流の中で食い違いが起きている**。

| 出典 | 記述 | 判定 |
|---|---|---|
| `core/tools/aidlc-lib.ts`（`DEFAULT_SCOPE`） | `classic` | **正** |
| `CHANGELOG.md` 2.6.18 | `classic`（declared behavior change） | **正** |
| `docs/guide/13-customization.md` / `docs/guide/12-cli-commands.md` | `classic` | **正** |
| `core/scopes/aidlc-classic.md` | 「`classic` is the implicit default scope」 | **正** |
| `docs/guide/05-scopes-and-depth.md` のキーワード表 fallback 行 | 「`feature` when core is enabled」 | **stale**（同一ファイル内の別記述と矛盾） |
| `docs/reference/03-orchestrator.md` | 「The underlying no-keyword resolver defaults to `feature`」 | **stale** |
| `core/scopes/aidlc-feature.md` 本文 | 「It remains the implicit freeform fallback」 | **stale**（`description` は更新済みなのに本文だけ古い） |

**同一ファイル内での矛盾**が 2 件ある。`docs/guide/05-scopes-and-depth.md` は Adaptive Composer 節で「The underlying resolver's no-keyword default is `classic`」と正しく書きながら、その少し上のキーワード表では fallback を `feature` と書いている。

さらに**出荷ファイル 2 つが直接矛盾している**: `core/scopes/aidlc-classic.md` が「classic is the implicit default scope」と主張する一方で、`core/scopes/aidlc-feature.md` は「It remains the implicit freeform fallback」と書いたままである。

本ノートは、**コード・CHANGELOG・`13-customization.md`・`12-cli-commands.md` の 4 系統が `classic` で一致していること**を根拠に `classic` を正とし、上記 3 箇所を stale と判断した。上流の Issue / PR は未参照なので、これが意図的な緩さか単なる更新漏れかは確認していない。

### ⚠ 2.6.49 の flow-style YAML バグ（カスタムスコープ作成者向け）

2.6.49 は frontmatter の文字列リスト項目で **1 行の flow 記法 `keywords: [a, b]` が黙って空リストとして解析されていた**不具合を修正した。**構文エラーにならないので気づけない**種類のバグで、キーワードを書いたつもりのスコープが自動検出に一切参加しない状態になる。scope の `keywords`、agent の examples、stage のリスト項目が対象。

影響を受けるのは主に**カスタムスコープを自作するチーム**である。**出荷の `classic` / `express` 自体はブロックスタイルで書かれているのでこのバグの被害者ではない**（`classic` の `keywords: []` は空が意図どおり）。対処は `dist/<harness>/` の再コピーで修正済みパーサを入れること。
