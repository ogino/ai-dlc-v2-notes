# 15. リリース差分 2.6.55 → 2.6.123

作成日: 2026-08-29
基準: `awslabs/aidlc-workflows` branch `v2` HEAD `2fbee12f`（取得 2026-08-29）／実装バージョン **2.6.123**
前回: [14-release-impact-2655.md](./14-release-impact-2655.md)（2.6.49 → 2.6.55、HEAD `840ba653`）

CHANGELOG の実エントリ **62 件**（2.6.56 〜 2.6.123）。
コミット **80 件**、変更ファイル **1,774 件**（+474,074 −83,246 行）、期間 **6 日**（上流コミット日 2026-08-23 〜 28）。

**これまでで最大の区間である。** 前回（2.6.49→2.6.55）が 6 版・195 ファイルだったのに対し、
版数で 10 倍、挿入行で 60 倍を超える。

**そのため本章は版番号順ではなく主題別に構成する。** 62 版すべての一覧は [15.10](#1510-全-62-版の一覧) に置く。

> **本章は 13 章・14 章の記述を 5 箇所訂正する。**
> いずれも当時の記述が誤っていたわけではなく、**上流が変わったか、私の射程が広すぎたか**である。
> 訂正は [15.9](#159-過去章の訂正) にまとめた。

---

## 15.1 数値の変化

| 項目 | 2.6.55 | 2.6.123 | 測定方法 |
|---|---:|---:|---|
| フェーズ | 5 | 5 | `stage-graph.json` の `phase` 集計 |
| ステージ | 33 | 33 | `core/aidlc-common/stages/**/*.md` |
| スコープ**数** | 11 | 11 | `core/scopes/*.md` |
| エージェント | 14 | 14 | `core/agents/*.md` |
| センサー**数** | 6 | 6 | `core/sensors/*.md` |
| **TypeScript フック** | **17** | **18** | `core/hooks/*.ts` |
| **`core/tools/*.ts`** | **41** | **51** | 同（※下記） |
| プロトコルモジュール | 8 | 8 | `core/aidlc-common/protocols/*.md` |
| **監査イベント種別** | **86** | **91** | `core/tools/aidlc-audit.ts` の `VALID_EVENT_TYPES` |
| 監査カテゴリ | 22 | 22 | `audit-format.md` の Event Registry 見出し |
| ハーネス | 7 | 7 | `harness/` 直下 |

> **※「CLI tools 51」と書くのは正確でない。**
> 51 は `core/tools/*.ts` の**ファイル数**である。`import.meta.main` を持つ
> **実際に起動可能な CLI は 26 → 32（+6）**、残り **15 → 19 本（+4）はライブラリモジュール**で、
> 利用者が直接叩くものではない（実測: 各ファイルの `import.meta.main` の有無を数えた）。
> 増えた 10 本の内訳も **CLI 6 本 / ライブラリ 4 本**である。
>
> | 新規ファイル | 導入版 | 種別 |
> |---|---|---|
> | `aidlc-plugin-create.ts` / `-validate.ts` / `-build.ts` / `-test.ts` | 2.6.105 | CLI |
> | `aidlc-review-brief.ts` | 2.6.101 | CLI |
> | `aidlc-unit.ts` | 2.6.107 | CLI |
> | `aidlc-plugin-emit.ts` | 2.6.105 | ライブラリ |
> | `aidlc-artifact-resolution.ts` / `-vocabulary.ts` / `aidlc-validity.ts` | 2.6.62 | ライブラリ |
>
> 本ノートは 13 章以来この行を「CLI tools」と呼んできたが、**ここで一度区別を明示しておく**。
> 過去章の値は当時の測定として正しいので変更しない。

### ⚠ 「不変」の指標が 2 つ、誤読されやすい

**スコープ「数」は 11 で不変だが、スコープの中身は動いた。**
**センサー「数」は 6 で不変だが、発火条件と severity の仕組みが変わった。**
どちらも [15.2](#152-bugfix--refactor-がデプロイ段まで走るようになった2670) と
[15.5](#155-センサーに-gate-発火と-blocking-が入った2672) で扱う。

### 変更 1,774 ファイルの内訳

| 区分 | ファイル数 |
|---|---:|
| `dist/` | 1,168 |
| `tests/` | 313 |
| `core/` | 131 |
| `harness/` | 74 |
| `docs/` | 68 |
| `plugins/` | 7 |
| `scripts/` | 5 |
| その他（CI 設定・ルート） | 8 |
| **合計** | **1,774** |

---

## 15.2 bugfix / refactor がデプロイ段まで走るようになった（2.6.70）

**今回いちばん具体的な変更である。** スコープが含むステージ集合そのものが変わった。

| スコープ | 2.6.55 | 2.6.123 |
|---|---:|---:|
| **bugfix** | **7 / 33** | **9 / 33** |
| **refactor** | **8 / 33** | **10 / 33** |
| classic / enterprise / express / feature / infra / mvp / poc / security-patch / workshop | — | **9 スコープすべて不変** |

`scope-grid.json` の差分は **4 行だけ**である。bugfix と refactor の
`deployment-pipeline` と `deployment-execution` が `SKIP` → `EXECUTE` になった。

CHANGELOG:

> Bugfix now runs **9 of 33 stages with 6 approval gates**;
> Refactor runs **10 of 33 stages with 7 approval gates**.

**何が変わるか。** 従来 bugfix / refactor は Build and Test で止まっていた。
今後は `security-patch` と同じデプロイ tail まで進む。
Environment Provisioning と残りの Operation ステージは引き続き SKIP である。

> **⚠ 「ステージ 33・スコープ 11 が不変」と矛盾しない。**
> 動いたのは**各スコープが含むステージ数**であって、ステージの総数でもスコープの総数でもない。
> 本ノートは 13 章以来「bugfix 7 / refactor 8」という数字を載せてきたが、
> [05-scopes-depth-test.md](./05-scopes-depth-test.md) の現行値は 9 / 10 に更新した。
> 10 章・11 章の同じ数字は**当時のスナップショットなのでそのまま**である。

同じコミットは併せて、Deployment Pipeline が条件付きスキップされたときに
Deployment Execution が missing-artifact recovery に落ちず既存のワークスペース構成を使うようにし、
`STAGE_SKIPPED` が「条件付きランタイムスキップ」と「明示ジャンプ」を区別するようにした
（ジャンプで飛び越した producer の欠落出力が "expected absence" として隠れなくなる）。

---

## 15.3 レビュー受領証は「実在する新規レビュー本文」に束縛された（2.6.121）

14 章 14.4 で扱った論点の続きである。**結論の構造は変わらないが、束縛は明確に強くなった。**

### 何が加わったか

ステージ定義に `review_artifact:`（`produces[]` 中の必須 Markdown 1 件）が**必須**になった
（reviewer 宣言のある **13 ステージすべてに付与済み**。実測 13/13）。

`REVIEW_COMPLETED` は今、次を検証する。

- **請求時点に存在しなかった、終端の `## Review` 節が review_artifact に追記されていること**
- その節に Verdict / Reviewer / Iteration が**各 1 個ずつ**であること
- **既存の `## Review` 節があった場合に限り**、request が発行した
  per-request の乱数 **`Review Challenge`** が完全一致すること。
  **初回レビュー（既存節が無い）では challenge は発行されず、書くと逆に拒否される** ——
  `aidlc-log.ts` は `priorAppendixLength > 0` のときだけ `review:<32 hex>` を鋳造し、
  `validateReviewAppendix()` は challenge の無い請求に対して当該行の**不在**を要求する

請求前から存在した節をそのまま残す経路も、1 バイト書き換える経路も、明示的に拒否される。

### ツールが証明するもの・しないもの

| 命題 | 2.6.52 | 2.6.121 |
|---|---|---|
| 受領証の対が存在する | ツール | ツール |
| 配車〜verdict 間に成果物バイトが不変 | ツール | ツール |
| **`## Review` 節が実在し、請求後に新規に書かれた** | **保証なし（散文のみ）** | **ツール（決定的）** |
| **古いレビュー節の使い回し・微修正の拒否** | **保証なし** | **ツール（Challenge）** |
| **レビュアー・サブエージェントが起動したこと** | **保証なし** | **保証なし（不変）** |
| **レビュアーが成果物を読んだこと** | **保証なし** | **保証なし（不変）** |
| verdict 値の正しさ | LLM | LLM |

`core/tools/aidlc-log.ts` の `SUBAGENT_COMPLETED` 参照は**新旧とも 0 件**である。
`.aidlc-reviewer-dispatch.json` も参照しない。

**なぜ Challenge が「起動の証明」にならないか。**
`Review Challenge` は `aidlc-log.ts` 自身が `randomBytes(16)` で生成し、
**request コマンドの stdout で conductor に返す**。散文はそれをレビュアー配車時に渡せと指示するが、
**値を握っているのは conductor である**。したがって Challenge は
**古い節の再利用を防ぐ replay 対策**であって、**conductor による自作を防ぐ仕組みではない**。

> **⚠ 14 章の「2 コマンドで通る」は訂正が要る（→ [15.9](#159-過去章の訂正)）。**
> 14 章 14.4 は「conductor は配車記録と受領証を **2 コマンド**で連続発行でき、
> その間にレビュアーを一度も起動しなくてもツールは通す」と書いた。
> 2.6.121 以降は **「2 コマンド + review_artifact への `## Review` 節 1 回の書き込み」**が正確である。
> **この訂正は 14.4 の結論を弱めない。** 増えたのは書き込みの手間であって、
> レビュアーが動いたことの証明ではない。

---

## 15.4 人間の関与 — 無人ドライバの抑止は opt-in（2.6.108）

13 章・14 章が一貫して扱ってきた「人間ゲートは本当に人間が通しているのか」という論点に、
**具体的な一手**が入った。

`AIDLC_UNATTENDED=1` が設定されたプロセスからの prompt-submit では、
権限を持つ `HUMAN_TURN` 監査イベントを**発行しない**。

上流フックのコメントには実測が残っている。

> Measured: 10 runner-submitted prompts, zero humans,
> and `humanActedSinceGate()` answered true

つまり「無人ランナーが毎サイクル使える `HUMAN_TURN` を鋳造してしまう」という状態が実在した。

### ⚠ ただし opt-in である

```ts
export function humanTurnMintAllowed(): boolean {
  return process.env.AIDLC_UNATTENDED !== "1";
}
```

**宣言しなければ従来どおり mint される。** 上流散文も明記している。

> The declaration is **opt-in** because only the driver knows whether a prompt is unattended;
> without it, prompt-submit events retain interactive behavior.

指定は環境変数のみで、CLI フラグも設定ファイル項目も無い。
宣言するのは「人がいない状態でプロンプトを投げる側のプロセス」（cron / CI / 夜間ランナー）であり、
**フレームワークが無人であることを検出するのではない**。

さらに、ゲート側の判定関数 `humanActedSinceGate` は **2.6.55 と完全に同一**である。

**したがって [13.6](./13-release-impact-2649.md) の
「人間ゲートは presence evidence にすぎず fails open」は維持される。**
ただし「fails open」には **「宣言しない無人ドライバに対しては」** という限定を付けるのが正確になった。
**「無人実行の穴が塞がれた」と書くのは過大である** — 塞ぐ手段が用意されただけで、
使うかどうかは運用側の判断に委ねられている。

### 決定コンテキストの提示（2.6.101）

新規ツール `aidlc-review-brief.ts` が、ゲート提示時に決定的な情報を描画するようになった。
Stage / Review outcome（**生の verdict トークンは出さず平易な文にする**）/
Why now（`first` / `revision` / `stale`）/ findings テーブル / Decision options。
`stale` のときは変更された上流と再確認が要る下流の具体パスまで出る。

指摘の処分（`Accepted risk` / `Rejected: <理由>`）は**ツール所有の監査行**に載り、
次回のレビュアー配車時のブリーフに畳み込まれるので、ID と処分が世代をまたいで生き残る。

> **⚠ ブリーフの実行を強制する仕組みは無い。**
> `aidlc-state.ts` が `aidlc-review-brief.ts` から import しているのは
> **処分の解析・検証ヘルパのみ**で、描画そのものではない。
> 「ゲート提示前にブリーフを実行して出力を見せる」のは**散文契約**である。
> **ツールが決定的なのは「処分の記録」であって「人間に見せたこと」ではない。**

---

## 15.5 センサーに gate 発火と blocking が入った（2.6.72）

**[07-learning-loop-state.md](./07-learning-loop-state.md) の記述が古くなった。**
同章は「現行の受理値は `advisory` のみ。`blocking` は将来の "v0.10.0 ralph driver" 向けに予約」
と書いていたが、**2.6.72 で `blocking` は実装された**。

manifest に 2 つのフィールドが入った。

| フィールド | 値 | 既定 |
|---|---|---|
| `fire_on` | `write` / `gate` | `write` |
| `default_severity` | `advisory` / `blocking` | `advisory` |

`claim-sources` / `required-sections` / `upstream-coverage` の 3 本が `fire_on: gate` に移行し、
承認境界で最終成果物に対して各 1 回走るようになった。
blocking な gate sensor は、findings・評価不能・出力不正・タイムアウト・identity 不一致・
評価中や評価後のバイト変化のすべてで fail closed する。

### ⚠ ただし出荷構成でゲートを塞ぐセンサーは 1 本も無い

実測すると、**出荷 6 センサーはすべて `default_severity: advisory` のまま**である。

| センサー | `default_severity` | `fire_on` |
|---|---|---|
| `claim-sources` | advisory | **gate** |
| `required-sections` | advisory | **gate** |
| `upstream-coverage` | advisory | **gate** |
| `linter` / `traceability` / `type-check` | advisory | （既定 = write） |

つまり `blocking` は「**カスタム/プラグイン sensor が使える能力**」として追加されたのであって、
**出荷したままゲートが塞がるようになったわけではない**。

さらに上流は書き込み発火について明記している。

> `fire_on: write` runs during matching writes and
> **remains advisory in this release, even when the manifest declares `blocking`**

**blocking が効くのは `fire_on: gate` のみである。**

なお blocking gate sensor は人間裏付けの override（decision → answer 受領証 →
`--override-blocking-sensors --user-input`）で上書きできるが、
**autonomous モードでは提示も受理もされない**。

---

## 15.6 「登録されている」ことと「実際に発火する」ことは別 — doctor の 3 層検出

本ノートは 13 章以来「TypeScript フック 17（→18）本」と数を数えてきた。
今回の 3 版は、**その数え方だけでは分からないことがある**と示している。

| 版 | 何を検出するか |
|---|---|
| **2.6.82** | Claude Code が `"disableAllHooks": true` で**フックを全域無効化**している（どのレイヤーが無効化しているかまで報告） |
| **2.6.90** | 「まだ populate されていない」（フックが発火する機会がまだ無い＝正常）と「`STAGE_STARTED` 済みなのに `.last` が 1 つも無い」（seam が死んでいる＝異常）を区別 |
| **2.6.93** | 「一度も heartbeat が無いまま stage が複数進んだ」「heartbeat が最新イベントより 5 分以上古い」「`allowManagedHooksOnly: true` で組織ポリシーが managed hook 以外を遮断」 |

**3 つとも別の失敗モードを見ており、重複ではない。**

> **新しい論点。** 「`settings.json` に登録されている」ことと
> 「Claude Code が実際にそれを発火させる」ことは別の主張である。
> フックの本数は前者しか数えていない。
> インストールが成功しているように見えて、**フックが 1 つも動いていない状態がありえた**
> ——だからこの 3 版が入った。

> **⚠ 「`--doctor` を実行すれば十分」なのは検出の話である。**
> 2.6.93 の Upgrade 文は、`dist/` の再コピーに加えて
> **「`/hooks` で Claude プロジェクトフックを承認し、Claude Code を完全に再起動する」**
> ことまで求めている。組織ポリシー由来の場合は管理者の対応が要る。

---

## 15.7 プラグイン — 作成ツールチェーンと、再インストールで消える合成

### 2.6.105 — オフラインで完結する作成ツールチェーン

`core/tools/` に 5 本が追加され、**7 ハーネスすべての `dist/` に出荷される**。

| ファイル | 役割 | CLI か |
|---|---|---|
| `aidlc-plugin-create.ts` | 決定論的な最小雛形を生成（非空ディレクトリは拒否） | CLI |
| `aidlc-plugin-validate.ts` | オフライン検証（manifest 形状・ステージスキーマ・成果物名前空間・重複プロデューサ・`tools/` へのテスト混入・symlink） | CLI |
| `aidlc-plugin-build.ts` | 検証したうえで 1 ハーネス分のネイティブ射影を出力 | CLI |
| `aidlc-plugin-test.ts` | 使い捨てコピーに合成 → グラフ再コンパイル → 2 回目の合成で冪等性を証明（**実インストールは変更しない**） | CLI |
| **`aidlc-plugin-emit.ts`** | 共有射影エンジン。`scripts/package.ts` と `plugin-build` が同一のエミッタを呼ぶ | **ライブラリ（CLI ではない）** |

> **⚠ 「検証手段が無かった」わけではない。**
> 2.6.17 で 3 層のテストキット（`tests/harness/plugin-kit.ts`）が既に入っていた。
> 変わったのは **それがフレームワークのチェックアウト内にしか無かった**という点である。
> さらに `scripts/package.ts` は `PLUGINS_ROOT = join(REPO_ROOT, "plugins")` で
> **リポジトリ内に vendoring されたプラグインしか発見しなかった**。
> 2.6.105 でその中核が `dist/<harness>/tools/` に出荷され、オフライン完結になった。
> `plugin-kit.ts` は削除されておらず、出荷ツールへ委譲する形で残っている。

### ⚠ 2.6.110 で検出が入った — エンジンを入れ直すとプラグイン合成が静かに消える

**これは既存のプラグイン利用者に直接影響する。**

`dist/<harness>/` を上書きコピーすると、コンパイル済み `stage-graph.json` とコアステージが素の状態に戻る。
**プラグイン名前空間のファイルと寄与サイドカーは残るのに、グラフのエントリと寄与マージだけが消える。**
結果としてプラグインのステージや寄与が**沈黙のうちに欠落する**。

2.6.110 で `/aidlc --doctor` に **`Composed plugin surface`** チェックが新設され、
これを **fail（exit 1）** として検出するようになった（2.6.55 に該当文字列は 0 件）。

**対処は `/aidlc plugin sync`。** 上流は書き分けている。

> Claude, Codex, Cursor, Kiro IDE は plugin compose フック経由で次セッション開始時に自己修復もできる。
> **Kiro CLI は明示的な sync が必要。**

合成は冪等なので重複は生じない。

### 2.6.78 — `plugin sync` の終了コードが変わった（**自動化に影響**）

| 状況 | 旧 | 新 |
|---|---|---|
| プラグイン root が 0 本（正常） | exit 0 | exit 0（維持） |
| root は設定済みだが使える compose フックが 0 本 | **exit 0** | **exit 1** |
| 一部だけ使える | — | 警告を出しつつ exit 0 |

**これまで exit 0 で通っていた誤設定が CI で落ちるようになる。**

### 2.6.65 と 2.6.95 — 重複プロデューサへの 2 段構え

同じハザード（consume される成果物に producer が 2 つ以上あると、
実行時はグラフ読み込み順の先頭を暗黙に選ぶ）に対する、**レイヤの違う 2 つの防御**である。

| | 2.6.65 | 2.6.95 |
|---|---|---|
| どこで | `aidlc-graph compile`（authored `.md` を読む段階） | `/aidlc --doctor`（コンパイル済みグラフ） |
| 何をする | **`throw` してコンパイルを止める** | **`pass: true` 固定の advisory**（exit code を変えない） |
| 対象 | これから作るグラフ | 既存のコンパイル済みグラフ、プラグイン合成後のグラフ |

**2.6.65 は自作ステージやプラグインが同じ成果物名を重複宣言しているとコンパイルが通らなくなる。**
片方をリネームするか consumer を変更する必要がある。

なお `aidlc-artifact-resolution.ts` / `aidlc-artifact-vocabulary.ts` / `aidlc-validity.ts` の 3 本は
**2.6.62 で導入**されたもので、この 2 版とは別である。これらは
「consume される成果物のプロデューサは 1 つ」という規約に**依存する側**であり、
その規約を強制するのが 2.6.65、既存グラフで報告するのが 2.6.95 という順序になる。

---

## 15.8 その他の主題

### team-owned Unit（2.6.107）— swarm とは排他

既存の opt-in `unit-major` walk（**2.6.55 時点で既に存在する**）の上に載る**第 2 の opt-in** である。
Unit をチームが所有し、clone や兄弟 worktree に fan out して、pin 付きの人間ゲートで収束させる。

**自律 swarm の代替ではなく排他である。** 上流散文:

> The autonomous swarm **never fires under unit-major**.

つまり team 所有の Construction 中は swarm 経路（`invoke-swarm` / worktree / `BOLT_*`）が
一切使われない。並列性の担い手が
「1 セッションが Task を N 本投げる自律 swarm」から
「別々のチェックアウトが claim する git-native な所有権」に替わる。

engine 側で 4 条件が強制される（`unit-major` が先にセットされていること /
intent に workspace repo が記録されていないこと / 妥当な Unit DAG があること /
attempt 中は凍結）。**したがって Unit を作らないスコープでは team 所有は使えない。**
fan out 先は同一ワークスペースリポジトリに限られ、**記録済み sibling repo を持つ intent は拒否される**。

新規 CLI `aidlc-unit.ts` がこの全操作を担う
（`adopt` / `claim` / `release` / `participate` / `publish` / `pin` / `gate` / `land` / `merge-status` / `status`）。

### 品質目標が Build and Test の成功条件になった（2.6.74）

`nfr-requirements/` / `nfr-design/` / `code-generation-plan.md` の承認済み `## Testing Contract`
の 3 源から測定可能な目標を棚卸しし、`## Target Verification Matrix` に
`Met` / `Not Met` / `Unverified` で判定する。`Not Met` / `Unverified` は
[13 章の 2.6.20 の 4 段ラダー](./13-release-impact-2649.md)に入る。

**Testing Posture（2.6.16）の置換ではなく、上に乗るものである。**
2.6.16 は「どんなテスト方法論とカバレッジ床で書くか」を決める契約、
2.6.74 は「そこで定めた床を含む全目標を、Build and Test の出口で証拠付きで判定する」層である。

> **⚠ 実体はステージ定義とプロトコル散文の追記のみである。**
> このコミットが触った `core/` は 5 ファイル（`build-and-test.md` +62 行 /
> `code-generation.md` +9 / `stage-protocol.md` +2 / `memory/org.md` +3 / `aidlc-version.ts`）で、
> **新しい TypeScript チェッカーもフックも追加されていない。**
> 「エンジンが閾値を機械検証する」と読んではいけない。

防いでいるのは「テスト設定や build 設定の閾値を下げてステップを通す」という抜け道である。
上流の文言: 「NEVER relax, lower, or disable a defined target,
**including threshold settings in test or build configuration**, to make a step pass;
surface the gap instead.」

### 質問プロトコル — 上流が「top field-feedback gap」と 2 度書いた

実利用者の声を受けた修正が 2 件ある。

- **2.6.116**: 記録済みの回答を再質問しない。範囲は**同一 intent（record）内**で、space 全体ではない。
  `<record>/**/*-questions.md` を再帰的に読み、監査シャードの `DECISION_RECORDED` は
  **同一 interaction scope** の後続 `QUESTION_ANSWERED` とだけ対応付ける。
  シャードをまたいだ同一タイムスタンプは因果的に順序不定なので**推測しない**。
- **2.6.112**: 構造化質問は自己説明的でなければならない。
  裸の `FR3` / `url1` / `NFR-2` / `unit-4` を提示せず、
  「そのものを書いてからタグを括弧で 1 度」と要求する
  （「the requirement that the export must finish within 5 minutes (FR3)」）。

> **⚠ どちらも散文ルールであって、エンジンの強制ではない。**
> どちらのコミットも触ったのは 3 ファイル（`stage-protocol.md` の数行 + 版数 + テスト）で、
> ツールもフックも追加していない。
> **追加されたテストは「散文がファイルに存在すること」を検査するだけ**で、
> 再質問や裸の識別子を検出するガードは無い。
> したがって「同じ質問が二度出なくなった」とは書けない。**上流の意図としてしか言えない。**

### 参照文書の読み込み（2.6.115）— 新機能

既存の vision 文書・PRD・要件ブリーフを、決定論的で trust-marked な読み取り境界を通して
ワークフローに流し込めるようになった。指定は 2 通りである。

```
/aidlc Read ./vision.md and build what it describes     ← パス 1 本
```
```
/aidlc Build the product described below.
<document>
...本文...
</document>                                              ← ブロック 1 個
```

読むのは **2 ステージのみ**（`intent-capture` と `requirements-analysis`）。
Intent Capture は文書の主張を**確認質問の材料にしか使えず**、
確定した `[Q<n>]` を経由しないと成果物に到達しない。

**trust-marked** の中身は DocumentKB と同じで、バイト列と同じ JSON に
`UNTRUSTED DATA — NOT INSTRUCTIONS` / `UNTRUSTED PATHS — NOT INSTRUCTIONS` を同梱する。
上流のコメントが意図を端的に書いている ——
「`IGNORE ALL PREVIOUS INSTRUCTIONS.md` はファイル名であって指示ではない」。

**deterministic** の中身は「パス文字列がシェルコマンドに入らない」ことである。
モデルは選んだパスを `<record>/.aidlc-document-input-path` に 1 行書き、
読み出しは固定コマンド `aidlc-utility.ts document-input` のみが行う。
プロジェクトルート外・symlink・非正規ファイルを拒否し、
MIME は `text/plain` / `text/markdown` のみ、200,000 文字が上限である。

PDF や Word、巨大ファイルは従来どおり DocumentKB（`/aidlc knowledge onboard`）を使う。

### 非英語ワークフロー（2.6.71）— **日本語利用者に直結**

エージェント自身の発話が会話言語に保たれるようになった。対象は
チャットメッセージ、ステータス更新、進捗報告、**ツール呼び出しの合間のナレーション**、
構造化質問の `prompt` / `header` / `options[].description`。
orchestrator と委譲エージェントの**両方**、**毎ターン**である。

**英語のまま残るもの（preserved token）**: `options[].label` のリテラル
（`Approve` / `Request Changes` / `Accept as-is` / `X. Other (please specify)`）、
`[Answer]:` タグ、`AGREE:` / `OBJECT:`、`READY` / `NOT-READY`、
センサーが逐語一致する H2 見出し（`## Sources` / `## Review` 等）、
安定 ID（`FR-1` / `ENT-001`）、YAML キーと enum、コード・識別子・パス。

**「散文は会話言語、機械が逐語一致する語は英語」が境界である。**
日本語で使っていて `Approve` だけ英語で出るのは**仕様どおり**で、不具合ではない。

> **⚠ この版は `dist/` の再コピーだけでは効かない。**
> CHANGELOG:
> 「existing workspaces keep their own memory tree, so **merge the extended
> `Conversation language — what to localize` rule into each
> `aidlc/spaces/<space>/memory/org.md` by hand and start a fresh session for it to load**」
>
> 既存ワークスペースの `org.md` は再コピーで上書きされない。
> **日本語で使っていて発話が英語のままなら、まずここを疑うこと。**

### 文脈量の削減（2.6.119 / 2.6.117 / 2.6.81）

- **2.6.119**: **33 ステージすべて**の `## Learn` と `## Sensors` の共通説明を、
  常時ロードされる `stage-protocol.md` の §13 と**新設 §14「Sensor Imports」**へ集約した。
  ステージ側に残るのは `Imports:` / `Upstream targets:` の 2 行要約と固有メモである。
- **2.6.117**: 「minimal workflows」は**スコープ名ではなく depth = Minimal** が条件で、
  さらに**刈り込み対象は 2 ステージ（intent-capture / requirements-analysis）の
  出荷済み framework knowledge のみ**である。他 31 ステージは Minimal でも従来どおり。
- **2.6.81**: 14 エージェント定義から重複するオーナーシップ・知識読み込み散文を除去した。

> **上流はいずれについても削減量を数値化していない。** 13 章 13.4 でも同じ状況だった。
> 参考として本ノートで測ると、2.6.119 の導入コミット境界で
> ステージ定義 33 本の合計が **310,472 → 246,465 バイト（−20.6%）**、
> 常時ロードの `stage-protocol.md` が **+1,873 バイト**である
> （測り方: `git cat-file -s` の合計を導入コミットの前後で比較）。
>
> **ただし「プロンプトが 20% 減る」とは言えない。**
> 測ったのは**ディスク上のバイト数**であってトークン数ではない。
> 1 ステージあたり −1,939 バイトに対し常時ロード側が +1,873 バイト増えるので、
> **`--single` の 1 ステージ実行ではほぼ相殺する**。効くのは複数ステージを通す通常のワークフローである。

### Windows 対応（2.6.99 / 2.6.92 / 2.6.85 / 2.6.84）

**4 版とも「Windows が新たにサポートされた」のではなく、既存対応の不具合修正である。**
旧版の時点で上流ドキュメントはネイティブ Windows PowerShell 対応を明言している。

- **2.6.92**: パス境界チェックが `node:path` のプラットフォーム依存 API に頼っていたため、
  混在セパレータ・drive-relative（`C:foo`）・UNC がトラバーサル検知をすり抜けうる**実装欠陥**だった。
  全ホストで `node:path/win32` を使う方式に変え、junction 判定も加えた。
  **セキュリティ性のある修正だが、実データでの悪用を確認したものではない。**
- **2.6.99**: 監査・state・usage ledger の read-modify-write を直列化するロックを、
  世代スタンプと `flock` / `LockFileEx` で堅牢化した。
  **14 章の「exactly-one-winner は単一ローカルファイルシステム前提」とは別機構である**
  （あちらは `.aidlc-active-directive.json` の rename による勝者確定）。
  前提を緩めても壊してもいないが、OS 固有プリミティブへの依存はむしろ増えた。
- **2.6.84**: コンパイル済みバイナリの検出が `/$bunfs/` という POSIX 形式のマーカーに依存しており、
  ネイティブ Windows で一致せず source モードにフォールバックしていた。
  **配布形態自体は既存で、今回新設ではない。**

### Kiro（2.6.64 / 2.6.60 / 2.6.88 / 2.6.113 / 2.6.106 / 2.6.56）

> **⚠ アップグレード作業の一覧は [06-harnesses-install.md §6.4](./06-harnesses-install.md#64-アップグレードでハーネスごとにすること262--2655) にまとめた。**
> 全 62 版の Upgrade 文を精査すると、**`dist/` の再コピー以外に作業が要る版が 20 ある**。
> **移行しないと止まるのは 3 つ —— 2.6.121（compose / graph compile 時）・
> 2.6.65（graph compile 時）・2.6.94（改名ヘルパを import するカスタムツールの起動時）である。**
> **2.6.121 と 2.6.65 は「上げた瞬間」ではない。** 既にコンパイル済みのグラフを再コンパイルしないなら動き続ける。
> ただし**プラグイン利用者は上げた時点で踏む**（`dist/` 入れ替えでグラフが素に戻り、
> 復旧の `/aidlc plugin sync` が compose を走らせる）。
> 残り 17 版は「やらないと不便・危険」だが動きはする。

> **⚠ 2.6.64 は手動削除が要る。** 13 章で 2.6.47 について書いたのと**同種の作業**である。
> Kiro IDE 配布物が CLI 用の agent-v1 JSON を同梱しなくなったため、
> 再コピーの後に古いファイルを消す必要がある。
>
> ```bash
> rm -f your-project/.kiro/agents/aidlc.json
> rm -f your-project/.kiro/agents/aidlc-*-agent.json
> rm -f your-project/.kiro/settings/cli.json
> ```
>
> 上流自身が理由を明記している ——
> **"An overlay copy cannot delete retired files."**

- **2.6.60**: Kiro のネイティブ面から Claude 専用キー `disallowedTools` が除去された。
  上流は `docs/reference/05-agent-system.md` の
  「唯一の出荷済み制限は `disallowedTools: Task`」という**一律記述をハーネス別に書き換えた**。
  なお [13.5](./13-release-impact-2649.md) の「無効なキーが出荷され続ける」という一般論は
  **依然として有効**である（`maxTurns: 60` は今回も未修正）。
- **2.6.113 / 2.6.106**: Kiro の質問表示（ルーティング質問の保持、`Other` 選択肢の番号）。
  **CLI と IDE を個別に確認したが、両者は同一の修正を受けており挙動差は無い**
  （CLI 用と IDE 用の個別テスト・個別 Windows 実測ログが両方存在する）。

### 監査イベント +5（86 → 91）、カテゴリは 22 で不変

| イベント | 版 |
|---|---|
| `SWARM_SOURCE_MERGED` | 2.6.69 |
| `UNIT_OWNERSHIP_SET` / `UNIT_GATE_RHYTHM_SET` / `UNIT_MERGED` | 2.6.107 |
| `PLAN_APPROVAL_RECORDED` | 2.6.118 |

**カテゴリ数が不変なのは、新規カテゴリが作られず既存 2 カテゴリの改名で吸収されたからである**
（「Unit Lifecycle Events」→「Unit Configuration and Lifecycle Events」、
「Plan Approval Events」→「Plan Approval Enforcement Events」）。

> **⚠ `PLAN_APPROVAL_RECORDED` は権限ではない。**
> 上流 `audit-format.md` が「this Markdown row is **not authorization evidence**」と明記している。
> 実際の承認権限は保護された非公開の challenge / response 受領証が持つ。
> **この行を「承認済み」の証拠として読むパーサを書いてはいけない。**

---

## 15.9 過去章の訂正

本章の調査で、13 章・14 章の記述が 5 箇所古くなっていることが分かった。
**過去章は日付付きの測定記録なので本文は書き換えない。** ここで訂正を宣言する。

| 章 | 記述 | 訂正 |
|---|---|---|
| **14 章 14.6** | 「2.6.55 でステージ日誌のプローブが禁止に変わった」 | **射程が広すぎた。** 実際に直っていたのは `conductor.md` だけで、Kiro の `SKILL.md`（1 件）と **33 ステージ定義すべて**に旧文言（`create on stage start if absent`）が残っていた。全面解消は **2.6.88** |
| **14 章 14.4** | 「conductor は **2 コマンド**で受領証を発行できる」 | 2.6.121 以降は **「2 コマンド + `## Review` 節 1 回の書き込み」**。結論（レビュアー起動の証明は無い）は不変 |
| **14 章 14.4** | `Source Fingerprint` は「**git ネイティブ**の temporary-index ハッシュ」 | **2.6.122 で Git 非依存の境界付きファイルシステム識別子に置き換わった**（`gitTreeFingerprint` の出現が 4 → 0）。Git の無いワークスペースでも束縛が効く。**進行中の `workspace_requires` レビューは完了前にレビューし直しが要る** |
| **13 章 / 4 章 §4.4** | DocumentKB は「決定的ツールで **LLM 不介在**」 | **ツール自体は今も LLM を呼ばない**が、**2.6.87 の `summarize` verb で LLM が書いた要約とタグを保存できる**ようになった。「DocumentKB に LLM 由来の内容が一切入らない」とは読めない |
| **7 章 §7.4** | センサーの `blocking` は「将来の v0.10.0 向けに**予約**」 | **2.6.72 で実装された**（→ [15.5](#155-センサーに-gate-発火と-blocking-が入った2672)）。ただし出荷 6 センサーは全て `advisory` のまま |

加えて、**上流が用語を再定義したことによる更新**が 1 件ある。

**2.6.86 で Bolt の定義が変わった。**

| | 旧（上流グロッサリ） | 新 |
|---|---|---|
| Bolt とは | **Construction の実行単位**。1 つの Unit について 3.1–3.5 を一巡すること | **スプリント様の Construction 反復**。Delivery Planning(2.9) が意図したグルーピングを記録する。**既定の stage-major ランタイムは `bolt-plan.md` をグルーピングや順序の境界として消費しない** |

`BOLT_STARTED` / `BOLT_COMPLETED` は **swarm / worktree 経路の Unit 単位イベント**として文書化され、
**通常のゲート付き実行では記録されない**。ランタイムのバッチは
`unit-of-work-dependency.md`（2.7）から再計算される。

**挙動は変わっていない**（ドキュメントとプロトコル散文の整合のみ）。
本ノートの Bolt 記述は 2.6.55 時点の上流グロッサリを忠実に写したものなので、
**本ノートが間違っていたのではなく上流の定義が変わった**。
[01-overview.md](./01-overview.md) と [03-phases-and-stages.md](./03-phases-and-stages.md) を更新した。

---

## 15.10 全 62 版の一覧

主題別に扱ったため本文で触れなかった版も含め、全 62 版を版番号順に挙げる。

| 版 | 要旨 | 本章での扱い |
|---|---|---|
| 2.6.123 | Cursor の委譲エージェント帰属とレビュアー状態の保護 | — |
| 2.6.122 | source freshness が Git 非依存に | [15.9](#159-過去章の訂正) |
| 2.6.121 | レビュアー受領証がレビュー本文に束縛 | [15.3](#153-レビュー受領証は実在する新規レビュー本文に束縛された26121) |
| 2.6.120 | focused Reverse Engineering の再スキャンが CodeKB にマージ（従来は置換） | — |
| 2.6.119 | ステージ定義の圧縮（33 ステージ全部） | [15.8](#158-その他の主題) |
| 2.6.118 | zero-Unit Code Generation でも Plan Approval を迂回できない | [15.8](#158-その他の主題) |
| 2.6.117 | 最小ワークフローの文脈削減 | [15.8](#158-その他の主題) |
| 2.6.116 | 記録済みの答えを再質問しない | [15.8](#158-その他の主題) |
| 2.6.115 | 参照文書の読み込み（新機能） | [15.8](#158-その他の主題) |
| 2.6.114 | no-DAG per-Unit レビュー継続性（2.6.76 の制限を解除） | — |
| 2.6.113 | Kiro のルーティング質問の保持 | [15.8](#158-その他の主題) |
| 2.6.112 | 構造化質問の自己説明性 | [15.8](#158-その他の主題) |
| 2.6.111 | プラグイン合成が `tools/` へのテスト混入を排除 | [15.7](#157-プラグイン--作成ツールチェーンと再インストールで消える合成) |
| 2.6.110 | 再インストールで失われたプラグイン合成の検出 | [15.7](#157-プラグイン--作成ツールチェーンと再インストールで消える合成) |
| 2.6.109 | swarm 収束が成果物とソースをスナップショットして適用 | — |
| 2.6.108 | 無人ドライバの人間存在トークン抑止 | [15.4](#154-人間の関与--無人ドライバの抑止は-opt-in26108) |
| 2.6.107 | team-owned Unit（複数チームの並列 Construction） | [15.8](#158-その他の主題) |
| 2.6.106 | Kiro の `Other` 選択肢の番号 | [15.8](#158-その他の主題) |
| 2.6.105 | プラグイン作成ツールチェーン | [15.7](#157-プラグイン--作成ツールチェーンと再インストールで消える合成) |
| 2.6.104 | エラー回復のナレーションを通常進行と同じ語調に | — |
| 2.6.103 | kind-vacuous Unit の summary-confirmation デッドロック解消 | — |
| 2.6.102 | ルールファイルのライフサイクルメタデータ | — |
| 2.6.101 | 人間チェックポイントの決定コンテキスト提示 | [15.4](#154-人間の関与--無人ドライバの抑止は-opt-in26108) |
| 2.6.100 | 知識インデックス復旧が symlink を跨いでも動く | — |
| 2.6.99 | 監査/使用量ロックの世代整合性（Windows） | [15.8](#158-その他の主題) |
| 2.6.98 | 分離 pipeline 実行が自身の受領証チェーンからのみ再開 | — |
| 2.6.97 | Stop フックが背景 Agent/Task をセッション単位で認識 | — |
| 2.6.96 | type-check センサーのキャッシュをプロジェクト記録に | — |
| 2.6.95 | 重複プロデューサの doctor advisory | [15.7](#157-プラグイン--作成ツールチェーンと再インストールで消える合成) |
| 2.6.94 | intent は「作成」であって「birth」ではない（語彙統一） | — |
| 2.6.93 | doctor がフックの未起動・途中停止・managed policy を検出 | [15.6](#156-登録されていることと実際に発火することは別--doctor-の-3-層検出) |
| 2.6.92 | 混在セパレータのパス境界強制 | [15.8](#158-その他の主題) |
| 2.6.91 | CodeKB scope fingerprint の pathspec 修正 | — |
| 2.6.90 | doctor が未 populate と死んだ seam を区別 | [15.6](#156-登録されていることと実際に発火することは別--doctor-の-3-層検出) |
| 2.6.89 | 「文脈が重い気がする」だけの park を禁止 | — |
| 2.6.88 | Kiro が日誌を書き込み専用先として扱う | [15.9](#159-過去章の訂正) |
| 2.6.87 | DocumentKB の要約とタグ | [15.9](#159-過去章の訂正) |
| 2.6.86 | Bolt 用語を出荷ランタイムに合わせ直し | [15.9](#159-過去章の訂正) |
| 2.6.85 | Kiro の出力を UTF-8 プレーンテキスト境界経由に | [15.8](#158-その他の主題) |
| 2.6.84 | コンパイル済み Windows 実行ファイルの検出修正 | [15.8](#158-その他の主題) |
| 2.6.83 | summary confirmation 受領証がワークスペース移動後も有効 | — |
| 2.6.82 | doctor が全域無効化されたフックを検出 | [15.6](#156-登録されていることと実際に発火することは別--doctor-の-3-層検出) |
| 2.6.81 | エージェント定義の重複散文除去 | [15.8](#158-その他の主題) |
| 2.6.80 | セッション単位のワークフロー束縛 | — |
| 2.6.79 | Stop フック回復が継続トークンを payload より前に | — |
| 2.6.78 | `plugin sync` が誤設定を exit 1 に | [15.7](#157-プラグイン--作成ツールチェーンと再インストールで消える合成) |
| 2.6.77 | stage validity が zero-Unit のパスを解決 | — |
| 2.6.76 | review-freeze と summary confirmation のデッドロック解消 | — |
| 2.6.75 | 共有 write フックが相対パスを正規化 | — |
| 2.6.74 | 品質目標が Build and Test の成功条件に | [15.8](#158-その他の主題) |
| 2.6.73 | コストプレビューが実際に走るワークフローを反映 | — |
| 2.6.72 | gate 発火センサーと blocking severity | [15.5](#155-センサーに-gate-発火と-blocking-が入った2672) |
| 2.6.71 | 非英語ワークフローの会話言語 | [15.8](#158-その他の主題) |
| 2.6.70 | bugfix / refactor にデプロイ段を追加 | [15.2](#152-bugfix--refactor-がデプロイ段まで走るようになった2670) |
| 2.6.69 | per-unit の source review 帰属 | — |
| 2.6.68 | summary confirmation 受領証が確認内容に束縛 | — |
| 2.6.65 | コンパイルが重複プロデューサを拒否 | [15.7](#157-プラグイン--作成ツールチェーンと再インストールで消える合成) |
| 2.6.64 | Kiro IDE が IDE ネイティブ面のみ出荷 | [15.8](#158-その他の主題) |
| 2.6.62 | stage-validity 受領証とドリフト検出 | [15.7](#157-プラグイン--作成ツールチェーンと再インストールで消える合成) |
| 2.6.61 | プラグインが doctor を拡張できる | [15.7](#157-プラグイン--作成ツールチェーンと再インストールで消える合成) |
| 2.6.60 | Kiro から `disallowedTools` を除去 | [15.8](#158-その他の主題) |
| 2.6.56 | Kiro CLI の agent-v1 hook matcher | [15.8](#158-その他の主題) |

**欠番**: 2.6.57 / 2.6.58 / 2.6.59 / 2.6.63 / 2.6.66 / 2.6.67（公開 CHANGELOG に現れない）。

---

## 15.11 版番号の同定について — ズレが再発した

14 章 14.8 で、2.6.49 → 2.6.55 の 6 件すべてが subject と CHANGELOG 見出しで一致したことを記録し、
**「これを上流の運用が改善したと読むのは早い」**と書いた。そのとおりになった。

80 コミットを機械照合した結果:

| 分類 | 件数 |
|---|---:|
| 一致 | 20 |
| **★ズレ** | **7** |
| 版表記なし | 53 |

ズレの実例を挙げる。

| コミット | subject の版 | 実際の CHANGELOG 見出し |
|---|---|---|
| `a09e55cc` | 2.6.19 | **2.6.119**（桁落ち） |
| `2dcc35d2` | 2.6.5 | **2.6.89** |
| `42164eee` | 2.5.60 | **2.6.79** |
| `3dc3112c` | 2.6.56 | **2.6.86** |
| `4c666fbc` | 2.6.72 | **2.6.87** |
| `b034b498` | 2.6.75 | **2.6.91** |
| `c30a8211` | 2.6.91 | （CHANGELOG 見出しの追加なし） |

`a09e55cc` は桁が 1 つ落ちており、**subject だけを見ると 100 版ずれる**。
`42164eee` に至ってはマイナー版から違う。

**版の判定は `CHANGELOG.md` の `## [x.y.z]` 見出しのみで行う。**
これは 12 章以来 4 区間にわたって同じ結論である。

なお 14 章で記録した別の罠 —— **`git log -S` に `--all` を付けると未マージブランチのコミットを拾う**
—— も今回再現した。本章の調査中、`Source Fingerprint` の導入版を `--all` 付きで検索すると
subject に `(2.5.6)` と書かれた未マージのコミットが返ったが、
それは `v2` の祖先ではなく、出荷版は 2.6.37 だった。
**`git merge-base --is-ancestor` で出荷ブランチの祖先か確認すること。**

---

## 15.12 本ノートの限界

- 本章は**上流リポジトリの差分の静的読解**に基づく。実機での再現は行っていない。
- 上流のテストスイートは実行していない（調査環境に `bun` が無い）。
- 文脈削減の実測値は **Git オブジェクトのバイト数**であって、
  トークン数でもプロンプト全体の実効サイズでもない。
- 2.6.116 / 2.6.112 / 2.6.74 は散文ルールのみで、
  **その効果（同じ質問が出ない、閾値の緩和が起きない）は検証していない**。
- 2.6.92 の混在セパレータ問題は、**コード構造上ありえた経路**を確認したもので、
  実データでの悪用を確認したものではない。
- 「2.6.123 調査で残った未確認事項」は
  [docs/REMAINING_TASKS.md](./docs/REMAINING_TASKS.md) に列挙した。
