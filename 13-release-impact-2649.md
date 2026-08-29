# 13. リリース差分 2.6.2 → 2.6.49

作成日: 2026-08-22
基準: `awslabs/aidlc-workflows` branch `v2` HEAD `71d9a9e0`（取得 2026-08-22）／実装バージョン **2.6.49**
前回: [12-release-impact-2602.md](./12-release-impact-2602.md)（2.5.62 → 2.6.2、HEAD `4569754e`）
次回: [14-release-impact-2655.md](./14-release-impact-2655.md)（2.6.49 → 2.6.55）

> **⚠ 本章の一部の記述は [15-release-impact-26123.md](./15-release-impact-26123.md) で訂正されている。**
> 本章は 2026-08-22 時点の測定記録なので本文は当時のまま残してある。
> どの記述が訂正されたかは 15 章を参照すること。

CHANGELOG の実エントリ **24 件**（2.6.8 / 9 / 12 / 13 / 14 / 15 / 16 / 17 / 18 / 20 /
36 / 37 / 38 / 39 / 40 / 41 / 42 / 43 / 44 / 45 / 46 / 47 / 48 / 49）。
コミット **28 件**、変更ファイル **1,206 件**（+140,570 −14,616 行）。

**この差分の性質は前回とちょうど逆である。** 2.6.1 は設計ステージの成果物名を作り直し、
永続化状態のスキーマを上げた「地形の変更」だった。今回は 28 コミット・+140k 行を投じながら、
**`stage-graph.json` が表す地形（slug・成果物・依存関係）を 1 ミリも動かしていない**
（→ [13.2](#132-ステージグラフは-scopes-以外完全に同一)）。**規則の側は広範に動いている。**
増えたのはスコープの選択肢と、ガードレールと、受領証と、ハーネス適合である。

ただし**宣言された挙動変更が 1 件ある**。2.6.18 で暗黙の既定スコープが
`feature` から `classic` に変わった（→ [13.3](#133-既定スコープの変更2618-今回の中心)）。

---

## 13.1 数値の変化

| 項目 | 2.6.2 | 2.6.49 | 測定方法 |
|---|---:|---:|---|
| フェーズ | 5 | 5 | `stage-graph.json` の `phase` 集計 |
| ステージ | 33 | 33 | 同（件数） |
| 実行区分 | ALWAYS 11 / COND 22 | ALWAYS 11 / COND 22 | 同の `execution` 集計。slug も完全一致 |
| mode 集計 | 29 / 2 / 1 / 1 | 29 / 2 / 1 / 1 | 同の `mode` 集計（inline / subagent / pipeline / mob） |
| エージェント | 14 | 14 | `core/agents/*.md` |
| **スコープ** | **9** | **11** | `core/scopes/*.md` および `scope-grid.json` |
| センサー | 6 | 6 | `core/sensors/*.md` |
| TypeScript フック | 17 | 17 | `core/hooks/*.ts` |
| フック登録数（`dist/claude`） | 18 | 18 | `settings.json` の `hooks[].hooks[]` を JSON パースして計上 |
| **CLI tools** | **38** | **41** | `core/tools/*.ts` |
| **監査イベント種別** | **82** | **86** | `core/tools/aidlc-audit.ts` の `VALID_EVENT_TYPES` |
| **監査カテゴリ** | **21** | **22** | `core/knowledge/aidlc-shared/audit-format.md` の見出し |
| **プロトコルモジュール** | **4** | **8** | `core/aidlc-common/protocols/*.md` |
| ハーネス | 7 | 7 | `dist/` 直下 |
| `produces` 異なり値（※） | 121 | 121 | `stage-graph.json` の `produces` を集合化して計数 |

> ※ **[12 章](./12-release-impact-2602.md) は同じ行を「122 → 122」と記録している。矛盾ではなく計数基準が違う。**
> 12 章は当時のアーティファクトレジストリを数え、本章は `stage-graph.json` の `produces` の
> 異なり数を数えている。本章が言えるのは「**この基準で 2 版が集合として完全一致した**」ことだけで、
> 12 章の 122 を訂正するものではない。**2 つの数字を直接比較しないこと。**

動いたのは 5 項目だけである。内訳は次のとおり。

- **スコープ +2**: `classic` と `express`。**既存 9 スコープの EXECUTE 数はすべて不変**
  （bugfix 7 / enterprise 33 / feature 33 / infra 13 / mvp 23 / poc 8 / refactor 8 /
  security-patch 10 / workshop 26）。新規は classic 26 / express 10。
  検証方法は 2 系統: `scope-grid.json` の各スコープの `EXECUTE` 件数と、
  33 ステージの frontmatter `scopes:` を独立に集計したもの。両者が一致した。
- **CLI tools +3**: `aidlc-documentkb-schema.ts` / `aidlc-knowledge.ts` / `aidlc-testing-posture.ts`
- **監査イベント +4**: `DOCUMENT_INDEXED` / `DOCUMENT_UPDATED` / `DOCUMENT_REMOVED` /
  `PIPELINE_LINK_COMPLETED`。**削除されたイベントは 1 件も無い**（差分は追加のみ）。
- **監査カテゴリ +1**: 増えたトップレベルカテゴリは `Documents`（3 イベント）**だけ**である。
  `Interaction Events` も 7 → 8 になっているが、これは**既存カテゴリの中のイベント数**であって
  カテゴリの増加ではない（3 + 1 = 追加イベント 4 件）。
- **プロトコルモジュール +4**: `stage-protocol-construction.md` / `stage-protocol-ensemble.md` /
  `stage-protocol-reviewer.md` / `stage-protocol-swarm.md`

> **監査カテゴリ数は出典によって違う。** 正典レジストリ `core/knowledge/aidlc-shared/audit-format.md` は
> 21 → 22 と数えるが、`docs/reference/12-state-machine.md` は 18 → 19 と数える。
> 上流自身が「the grouping is presentational, the event set is the invariant」と注記しており、
> 矛盾ではなくグルーピングの粒度の違いである。**本ノートは正典レジストリ側の 22 を採る。**

---

## 13.2 ステージグラフは `scopes` 以外**完全に同一**

今回いちばん強い観測はこれである。`stage-graph.json` の各ステージから
`scopes` フィールドだけを除いて 2 版を比較すると、**差分が 1 行も出ない**。

```
diff <(git show 4569754e:<graph> | 各ステージから scopes を除いてソート出力) \
     <(git show 71d9a9e0:<graph> | 同じ処理)
→ 無出力
```

つまり slug・番号・名前・フェーズ・`execution`・`lead_agent`・`support_agents`・`mode`・
**`produces`・`consumes`**・`requires_stage`・`sensors`・`inputs`・`outputs`・`rules_in_context` が
**すべて 2.6.2 と同一**である。変わったのは「各ステージがどのスコープに属するか」だけで、
それも `classic` / `express` のメンバーシップが加わったことによる。

`produces` の異なり数は両版とも **121** で集合として完全一致し、
`docs/reference/16-artifact-vocabulary.md` も `git diff --stat` が無出力（＝ファイル自体が無変更）だった。

**この観測が何を意味するか。** **永続化状態のスキーマ移行**が不要なのは、
各版の Upgrade 文が親切だからではなく、**動かす対象が無かったから**である。
24 版の Upgrade 文のうち、**状態スキーマの移行を求めるものは 1 件も無い**。
2.6.1 のときに必要だった「`skills/aidlc-application-design/` の手動削除」に
相当するステージ再編由来の作業も今回は発生しない。

> **⚠ 「移行不要」＝「再コピーだけで済む」ではない。** ここで言っているのは
> **状態スキーマ**の話に限られる。実際には Codex の trust テーブル再生成、
> Copilot の進行中会話の作り直し、2.6.36 による旧 `selections` ファイルの再生成など、
> **`dist/` の再コピーだけでは終わらないハーネスが 4 つある**
> （codex は必ず追加操作が要る。copilot は進行中ワークフローがある場合、kiro と kiro-ide はプラグイン利用時に要る。
> → [13.8](#138-ハーネス別のアップグレード手順)）。

逆に言えば、**この差分を「地形が変わっていないから小さい」と読むのは誤りである。**
変わったのは地形ではなく、地形の上で何が許され何が拒否されるかの規則であり、
そちらは 13.3 以降のとおり広範に動いている。

---

## 13.3 既定スコープの変更（2.6.18）— 今回の中心

2.6.18 は `feat!` が付いた唯一のコミットで、CHANGELOG 自身が
**declared behavior change** と明記している。

### 何が変わったか

`classic` と `express` の 2 スコープが新設され、**暗黙の既定が `feature` から `classic` になった**。

| | classic | express |
|---|---|---|
| ステージ | 26 / 33 | 10 / 33 |
| depth | Standard | Minimal |
| テスト戦略 | Standard | Minimal |
| `review_cap` | `advisory` | **`none`** |
| キーワード | **なし**（名指しで選ぶ経路は `--scope classic` だけ。ただし暗黙の既定としては別経路で当たる） | `express`, `lightweight` |
| 専用スキルランナー | **なし** | あり |
| スキップ | Ideation 1.1–1.7 の 7 本すべて | Ideation・設計一式・Units Generation・Delivery Planning・CI Pipeline・環境構築・後半 Operation |

`classic` は上流の定義ファイルが「reproduces the AI-DLC v1 experience」と書いているとおり、
**v1 相当のライフサイクルを v2 の上で選べるようにしたもの**である
（v1 との関係は [08-v1-vs-v2.md](./08-v1-vs-v2.md) を参照）。
`express` は要件から実装・テストを経て条件付きデプロイまで一直線に進む最小構成で、
**設計パスもレビュアー配車も持たない**。

### ⚠ 「黙って classic が始まる」わけではない — 2 つのレベルを分けて読む

ここは誤読しやすいので分けて書く。

**resolver レベル（コード実測）**

```
2.6.2  : core/tools/aidlc-orchestrate.ts  const DEFAULT_SCOPE = "feature";   ← ファイル内ローカル定数
2.6.49 : core/tools/aidlc-lib.ts          export const DEFAULT_SCOPE = "classic";  ← 共有 export
```

（本ノートの他の箇所で行番号を添えている場合、それは `4569754e` / `71d9a9e0` 時点のもので、
整形や後続の変更で容易にずれる。**探すときは行番号ではなく `DEFAULT_SCOPE` という識別子で grep すること。**）

定数の置き場所が orchestrate 内のローカル定数から lib の export に移り、値が変わった。
CHANGELOG によれば、エンジンのスコープ解決フォールバック・`/aidlc-init`・
`--scope` 無しの低レベル `intent-create`（**従来は `poc` を返していた**）の 3 経路が、
すべて `AWS_AIDLC_DEFAULT_SCOPE` を先に見て、無ければ `classic` に落ちる形に統一された。

**UX レベル（上流散文）**

対話的に `/aidlc <説明文>` を打った場合はこうならない。上流はこう書いている。

> In the user-facing cold-start flow, rich prose, no keyword hit, or a keyword buried in a
> long description enters the compose offer before a workflow is created;
> **it does not silently start Feature.**

つまり、キーワードに当たらない自由記述を投げると、ワークフローが生まれる前に
**compose offer（適応型コンポーザの提案）が先に提示される**。
`classic` が静かに使われるのは、compose offer を経由しない低レベル呼び出しに限られる。

（この引用が `Feature` を名指ししているのは、**2.6.18 より前の既定が `feature` だったから**である。
2.6.49 の文書に載っている現行の文言だが、「黙って始まらない」という主張の対象が
旧既定の名前で書かれている点に注意。）

**したがって「既定が classic になった」は正しいが、「対話中に黙って classic が始まる」は誤りである。**

### `AWS_AIDLC_DEFAULT_SCOPE` の出荷値 — ハーネス間で非対称

| ハーネス | 出荷設定ファイル | 2.6.2 | 2.6.49 |
|---|---|---|---|
| Claude Code | `dist/claude/.claude/settings.json`（および `harness/claude/settings.json`） | `"workshop"` | **`"classic"`** |
| codex / copilot / cursor / kiro / kiro-ide / opencode | **出荷していない** | 未設定 | 未設定 |

**Claude 利用者にとっての実効的な変化は、ハードコード既定の変更ではなく出荷設定値の変更である。**
出荷 `settings.json` の `env` ブロックが常にハードコード既定を上書きするため、
Claude では `DEFAULT_SCOPE` に到達しない。旧版で出荷値が `workshop` だったこと
（＝ワークショップ向けの Minimal テスト戦略が既定で効いていたこと）のほうが、
むしろ特殊だったと言える。

他 6 ハーネスの利用者はこの環境変数を出荷設定として持たない。
ただし**「だから常に `classic` で始まる」わけではない**。前述のとおり、対話的な
`/aidlc <説明文>` のコールドスタートは compose offer に入るのでハードコード既定に到達しない。
到達するのは `/aidlc-init` や `--scope` 無しの `intent-create` といった
**低レベル・フォールバック経路**であり、そこでは環境変数を自分で設定しない限り `classic` になる。

**旧挙動に戻すには** `AWS_AIDLC_DEFAULT_SCOPE=feature` を設定する。
Claude では `.claude/settings.json` の `env` ブロックを書き換える。

### `review_cap: none` — reviewer を丸ごと止められるスコープが初めて登場した

| 値 | 2.6.2 のスコープ | 2.6.49 のスコープ |
|---|---|---|
| 未宣言 | enterprise / feature / mvp / infra / refactor / security-patch | 同左 |
| `advisory` | bugfix / poc / workshop | bugfix / poc / workshop / **classic** |
| **`none`** | **（存在しない）** | **express** |

2.6.2 の時点で `review_cap` という仕組み自体は存在したが、値は未宣言と `advisory` の 2 通りだけで、
**reviewer を完全に無効化するスコープは無かった**。`none` は **2.6.18 で導入された**値である
（`git log -S'review_cap: none' -- core/scopes/` が返す唯一のコミット `fbb1460c` は
`express` を新設したコミットそのもので、CHANGELOG 見出しは `## [2.6.18]`）。

> **⚠ 静的グリッドの membership と実行時の挙動は別物。**
> `express` はグリッド上、reviewer 宣言を持つステージを 2 本（Requirements Analysis と
> Code Generation、いずれも ALWAYS）含む。にもかかわらず reviewer は走らない。
> `review_cap: none` が dispatch そのものを無効化するからである。
> 「express は reviewer ステージを 2 本持つ」とだけ書くと逆の印象になるので、
> 必ず `review_cap` と併せて読むこと。

なお reviewer 宣言を持つステージは**両版とも 13 本で不変**（slug も同一）である。

### ⚠ 上流ドキュメント自身が矛盾している

今回、上流の記述に**更新漏れ（stale）が 3 箇所**残っているのを確認した。
本ノートの読者が上流ガイドを直接読んだときに混乱しうるので明記しておく。

| 箇所 | 何と書いてあるか | 問題 |
|---|---|---|
| `docs/guide/05-scopes-and-depth.md` のキーワード表 fallback 行 | fallback は `feature` | **同一ファイルの数行後**が「resolver の no-keyword 既定は `classic`」と書いており自己矛盾 |
| `docs/reference/03-orchestrator.md` の no-keyword resolver の記述 | 既定は `feature` | **同一ファイルの別の箇所**が「classic — The implicit default」と書いており自己矛盾 |
| `core/scopes/aidlc-feature.md` の本文 | 「It remains the implicit freeform fallback」 | `description` は「Full lifecycle for new features」に更新済みなのに本文だけ古い。`aidlc-classic.md` の「classic is the implicit default scope」と**出荷ファイル 2 つが互いに矛盾** |

コード（`DEFAULT_SCOPE = "classic"`）・CHANGELOG・`docs/guide/13-customization.md`・
`docs/guide/12-cli-commands.md` は `classic` で一致しているため、**`classic` が正**、
上記 3 箇所が stale と判断した。

これは [12.9](./12-release-impact-2602.md) で書いた「上流ドキュメントの読み方の注意」の続きである。
前回は `docs/rfcs/` が仕様書ではないという話だったが、今回は**正規のガイドとリファレンス、
さらにランタイムが読む出荷ファイルにまで更新漏れがある**という話になる。
**上流の散文を 1 箇所だけ読んで結論を出さないこと。**

---

## 13.4 プロトコルの条件付きモジュール化（2.6.18）

ステージプロトコルが 4 ファイルから 8 ファイルに分割された。

| モジュール | 何を持つか | いつ読まれるか |
|---|---|---|
| `stage-protocol.md` | 基本 | 常時 |
| `stage-protocol-governance.md` | ガバナンス | 常時 |
| `stage-protocol-recovery.md` | 回復 | 常時 |
| `stage-definition.md` | ステージ定義の書式 | 常時 |
| **`stage-protocol-reviewer.md`** | reviewer 配車・受領証・読み取り範囲・終端順序・NOT-READY ループ | `protocol_modules` に `reviewer` がある、または directive が reviewer を持つとき |
| **`stage-protocol-ensemble.md`** | ensemble トポロジ・subagent 戻り値・contribution files・異議のトリアージ | `protocol_modules` に `ensemble` がある、または topology / support agents があるとき |
| **`stage-protocol-construction.md`** | Bolt ゲート・Build&Test 失敗ループバック・per-unit iteration・受領証・wave | セッション最初の Construction directive、および全 `invoke-swarm` 時 |
| **`stage-protocol-swarm.md`** | ハーネス固有の自律 fan-out・収束・finalize・reviewer 境界 | 全 `invoke-swarm` 時 |

狙いは、reviewer や swarm を使わない通常のステージ実行で読み込む固定コンテキストを減らすことである。

> **削減量は上流が数値化していない。** 上流の記述は
> 「reduces fixed context during normal stage execution」だけで、
> 割合・トークン数・KB のいずれも書かれていない。本ノートも数値を書かない。

> **モジュール数は数え方が 2 通りある。** 実ファイル数は 4 → 8 だが、
> `docs/reference/04-stage-protocol.md` は `stage-definition.md` を数に含めず
> 「three files → seven files」と書く。矛盾ではなくカウント基準の違いである。

---

## 13.5 レビュアーの 60 ターン上限（2.6.8）— ハーネスで強制力がまるで違う

`maxTurns: 60` が `aidlc-architecture-reviewer-agent` と `aidlc-product-lead-agent` に付いた。
**`core/agents/*.md` の 14 エージェント定義のうち、この 2 つだけ**である
（13.1 の表で「エージェント 14」と数えているのと同じ母集合。
ハーネスへ投影された配布物側のファイル数とは別なので注意）。

**ここが今回いちばん誤読されやすい。** 上流 CHANGELOG の一文が正典なので、それに従って書く。

| ハーネス | 強制力 | 上限に達したときの挙動 |
|---|---|---|
| **Claude Code** | **ネイティブ強制** | サブエージェントを停止。**final-message ターンすら無い**。レビューを書いていなければ**丸ごと失われる** |
| **opencode** | **ネイティブ強制**（`steps: 60` に改名） | **テキストのみの最終ターンを 1 回**許す。要約は返せるが、**ツールを呼べないのでレビューは書けない** |
| Codex CLI | **散文のみ** | TOML ペルソナに frontmatter が無いため、emit がペルソナ本文の記述を書き換える形 |
| Cursor / GitHub Copilot | **散文のみ** | per-agent cap キーが無い |
| Kiro CLI / Kiro IDE | **散文のみ** | per-agent cap キーが無い。ネイティブ JSON はスキーマが未知フィールドで fail-close するため、キーはそこに入らない |

> **無効なキーが出荷され続ける点に注意。** 上流は Cursor / GitHub Copilot / Kiro CLI / Kiro IDE について
> 「the inert key still ships on the tolerant `.md` surfaces and never enters the kiro agent JSONs,
> whose schema fail-closes on unknown fields」と書いている。
> つまりこの 4 ハーネスの `.md` ペルソナ面には **`maxTurns: 60` の行が物理的に存在するが、何の効果も持たない**。
> **ファイルにキーがあることを強制の証拠と読まないこと。**

**「レビュアーは 60 ターンで必ず止まる」と言えるのは Claude Code と opencode だけである。**
残り 5 ハーネスでは「エージェントが自分で守る」ことに依存し、決定的な保証は無い。
しかも強制される 2 つでも挙動が違う（Claude は最終ターン無し、opencode はテキストのみ 1 ターン）。
共通するのは、**上限に達した時点でまだ書かれていないレビューはその後も書けない**という点である。
**上限より前に verdict を書き終えていれば成果物は残る**ので、
「上限に達した＝レビューが失われる」ではない。失われるのは**書く機会**である。

なお 60 という値は恣意的なものではない。上流は
「the value sits at the top of the observed legitimate range from field timing data
（p90 40 / max 60 turns at the medium-effort reviewer tier）」と説明しており、
実測分布の上端に置くことで暴走だけを捕まえ、正当なレビューを打ち切らない狙いだとしている。

ペルソナ本文自身がこの最悪ケースを前提に書かれている。

> When you hit it you are STOPPED mid-task - in the worst case WITHOUT warning and
> WITHOUT a final-message turn: your caller receives no output, and an unwritten review is
> simply lost. Plan for that worst case every time: write the review BEFORE the cap,
> never on your last turn.

### incomplete-attempt guard も決定的ではない

2.6.8 は、レビュアーが verdict を書かずに死んだ場合の未定義分岐も塞いだ。
レビューが有効と認められるのは「**ちょうど 1 つの `## Review` 節 + ちょうど 1 つの正規 verdict**」の場合のみで、
それ以外は INCOMPLETE として `--retry-pending` で 1 回だけ無消費リトライし、
2 回目も不完全なら `NOT-READY` を確定受領証として記録する。
またレビュアー配車の前に既存の `## Review` 節を毎回削除するので、
改訂前の古い READY が改訂後の成果物を承認したかのように読まれる穴も塞がれた。

**ただしこの verdict のパースは conductor（LLM）のプロトコル遵守に依存しており、
ツール側の強制ではない。** 上流テスト `tests/unit/t279-reviewer-turn-budget.test.ts` の
冒頭コメントは **"Mechanism: none. Pure content checks over authored + shipped bytes"** と明記している。
このテストが保証しているのは「散文が全ハーネスに正しく配布されたか」であって、
実行時に上限やガードが効くことではない。

---

## 13.6 人間の承認ゲートは強化されたか（2.6.13 / 2.6.44）

[12 章](./12-release-impact-2602.md)と [11 章](./11-release-impact-2562.md)で扱った
「人間の承認ゲートは本当に人間が通しているのか」という論点が、今回 2 方向に動いた。
**どちらも過大評価しないよう注意して読む必要がある。**

### 2.6.13: conductor の自己申告を弾く — ただし fails open

`approve --user-input` / `reject --feedback` / `answer --details` に渡されたテキストが、
**明示的に「これは人間ではない」と自己申告している**場合に限り拒否するようになった
（「not a human approval」「agent-initiated approval」「AI authored this decision」
「conductor default」等のカテゴリ）。マッチすると状態を変えず監査受領証も出さずにエラー終了する。

**上流のコード内コメント自身がこう書いている。**

> Unlabelled or unknown wording deliberately fails open;
> this is a defense-in-depth tripwire, **not an authorship boundary.**

つまり検出対象は決め打ちの英語表現だけで、**言い換え・多言語・婉曲表現は素通りする**。
上流はこれを「issue #742 で観測された自己申告型の自動化に対する仕掛け線」と位置づけており、
人間が書いたことの証明ではないと明言している。

さらに `AIDLC_SKIP_HUMAN_PRESENCE_GUARD=1` を設定すると、この自己申告チェックだけでなく
**(1) `--user-input` の必須化 (2) キャンセル文言の拒否 (3) `HUMAN_TURN` の存在チェック**
までまとめて無効化される。決定論的な回復やテストのための逃げ道だが、
**これを設定した環境では人間ゲートの検証が事実上すべて外れる**ことになる。

`audit-format.md` も `HUMAN_TURN` について、
「**presence/freshness の証拠**であって、認証されたトランスクリプトでも、
後続の caller 提供テキストが人間由来である証明でもない」と明記するよう書き換えられた。

**したがって「人間ゲートが強化された」と単純に書くのは過大評価である。**
正確には「明示的に自己申告した場合にのみ弾く、狭い defense-in-depth が足された」。

### 2.6.44: Codex の構造化選択も HUMAN_TURN として数える

こちらは実質的な前進である。Codex の `request_user_input`（構造化選択ウィジェット）への回答が、
タイプ入力と同じ `HUMAN_TURN` 証跡を発行するようになった。
従来は選択で答えると人間のターンとして記録されなかった。

除外条件は実装で担保されている。空・取消・タイムアウト・自動解決・error・不正応答は数えない。
上流テストに **「empty, cancelled, timed-out, auto-resolved, error, and malformed responses
do not mint」** というテスト名がそのまま存在する。
なお `Abort` のような語でも、質問が実際にその選択肢を提示していた場合は
人間の判断として有効に扱われる（キャンセル定型文との区別）。

**ただしこれは Codex のアップグレードに追加操作を要求する**（→ [13.8](#138-ハーネス別のアップグレード手順)）。

---

## 13.7 知識・テスト・Construction

### DocumentKB（2.6.15）— CodeKB と紛らわしいので三者を区別する

「自分たちの文書をエージェントが引用できるようにする」機能が入った。
**名前が既存の 2 つと紛らわしいので、三者を分けて把握する必要がある。**

| | CodeKB MCP（外部） | `aidlc/spaces/<space>/codekb/` | `aidlc/spaces/<space>/knowledge/documentkb/`（新） |
|---|---|---|---|
| 実体 | AI-DLC 非同梱の外部 MCP サーバ | Reverse Engineering(2.1) の出力先。9 成果物 | ユーザー文書の索引 |
| 対象 | 既存コードの構造推定（call graph 等） | 既存コードの解析結果 | PDF / Word / Markdown / プレーンテキスト |
| 生成者 | 外部ツール | `aidlc-architect-agent` | `tools/aidlc-knowledge.ts`（決定的ツール、LLM 不介在） |
| 操作 | ハーネス依存の `@<server>` 登録 | ステージ実行 | `/aidlc knowledge onboard` / `sync` / `list` / `show` / `associate` / `rebind` |
| **生成・取り込み工程**がネットワークに出るか（※モデルとのやり取りは別） | 出る（サーバ次第） | フレームワーク自身は出ない。ただし**生成するのは `aidlc-architect-agent`（LLM）** | フレームワーク自身は出ない（`fetch(` / `http(s)://` の出現 0 件） |

**AI-DLC 自身のコードは通信しない。** `fetch(` および `http(s)://` の出現は
`aidlc-knowledge.ts` 内に **0 件**で、抽出は `spawnSync` で外部実行ファイルを呼ぶだけである。

> **ただし「抽出工程がローカル完結する」保証ではない。** 抽出コマンドの argv は
> **ハーネス設定で差し替えられる**（上流コードのコメント: "The argv for a MIME type:
> a harness-configured one, else the default"）。既定は PATH 上の `pdftotext` だが、
> **通信する extractor を指定すれば `spawnSync` はそれを止めない**。
> 実測が示すのは「AI-DLC 本体に直接の通信が無い」ことだけで、extractor の挙動は別に確認が要る。

> **⚠ ただし「文書が外部に出ない」という意味ではない。** ここで実測したのは
> **索引を作る工程**にネットワーク呼び出しが無いことだけである。
> 索引された内容は、その後**エージェントが読んで引用するために存在する**。
> リモートでホストされたモデルを使うハーネスでは、
> **読まれた時点でその内容はモデルとのやり取りに乗る**。
> 機微な文書を投入する判断は、索引工程の実測ではなく
> **利用しているモデルの提供形態**で行うこと。

破壊的操作（`remove`）は**意図的に実装されていない**。原本を消して `sync` する運用になっている。
抽出できなかった場合の状態は 6 種の判別共用体（`extracted` / `no_extractable_text` /
`extractor_unavailable` / `extraction_failed` / `unsupported_type` / `invalidated`）で表現される。

「S1」は第一スライスの意味で、後続は上流 issue #714 として予告されている。
**ただし「S2」という版番号やマイルストーン名は上流文書のどこにも存在しない。**

> **監査の例外がここに 1 つある。** `DOCUMENT_INDEXED` / `DOCUMENT_UPDATED` / `DOCUMENT_REMOVED` の
> 3 イベントは、**フレームワークで唯一「状態変更の後」に監査を出す**（audit-last）。
> 通常は audit-first atomicity（監査を先に出し、失敗したら state に触れない）が原則である。
> 上流の理由づけは、カタログが派生物で `sync` により再構築可能なので、
> 「取りこぼし（過少申告）」のほうが「幻のエントリ（虚偽の主張）」より安全だから、というもの。
> この例外はこの 3 イベントに限定され、権威ある state ファイルには広げないと明記されている。

### Testing Posture（2.6.16 / 2.6.38）

チームが表明したテスト方針が、Code Generation の実行契約として固定されるようになった。
org / team / project の 3 層から TDD / BDD / ATDD / test-after / custom を一意に解決する。
**`core/tools/aidlc-testing-posture.ts` の実装では** project > team > org >
フォールバック（test-after）の優先順で 1 つを選ぶ。
team と project が非互換な場合は例外を投げて拒否する
（「strict-additive memory does not permit runtime override」）。

承認は `plan 本文 + instructions 本文 + 契約ハッシュ` をまとめた
フィンガープリントに束縛される。したがって計画や指示の 1 文字の変更でも、
scope / テスト戦略 / プロジェクト種別の変更でも、**既存の承認は失効する**。
developer サブエージェントへの指示には `AIDLC-TESTING-CONTRACT: <hash>` の行が必須になり、
欠落・不一致・陳腐化したハッシュは拒否される。

2.6.38 で HTML コメント（`<!-- -->`）内の記述がメソドロジー判定から除外された。
ただし入力ハッシュは raw セクションを保持するため、
**コメントだけを変更しても既存の承認は失効する**（可視化ロジックとハッシュ化ロジックが別）。

### Build & Test → Code Generation のループバック（2.6.20）

Build and Test（3.6）が失敗したとき、その根本原因が生成コードや
コード生成時の方式選択（ライブラリ・版・コンテナイメージ・インスタンスタイプ・
アルゴリズム・フラグ）にある場合、Code Generation（3.5）へ戻って直せるようになった。

失敗処理は 4 段の階梯になった。

1. **In-stage fix（最大 2 回）** — このステージの管轄内（テスト設定・ビルドスクリプト・環境）のみ
2. **分類と影響見積り** — 原因が上流にあるなら修正案を探し、**工数・金銭コスト・リスクを見積もる**。
   見積りをしないまま「実行不可能」と結論することは禁止されている
3. **自律ループバック** — `Construction Autonomy Mode: autonomous` かつ影響見積り済みの修正案があり、
   かつ `## Loop-Back Log` のエントリが 3 件未満のときのみ
4. **停止して人間に問う** — gated / 未設定、上限到達、修正案なしの場合

> **⚠ 「1 intent につき 3 回まで」はコードで強制されていない。**
> `core/tools/*.ts` を `loop.?back` で検索してもヒットは 0 件である。
> 上限は `core/aidlc-common/protocols/stage-protocol-construction.md` の散文
> 「the count of `### Loop-back N` entries **IS the bound** (max 3 per intent)」にのみ存在する。
> 上流はこれを設計判断として正当化しているが（`STAGE_JUMPED` の監査行を parse するより、
> ジャンプで消えないアーティファクト台帳のほうが確実だ、という理由）、
> **ツールによる強制ガードではなく、conductor が守るべき運用規約である。**

> **⚠ 「v2 に初めて後戻り経路ができた」わけではない。**
> `core/tools/aidlc-jump.ts` も監査イベント `STAGE_JUMPED`
> （"Forward/**backward**/redo jump target reached"）も、`stage-protocol.md` の
> `### Artifact Re-use (backward jump / redo)` 節も、**すべて 2.6.2 の時点で既に存在する**。
> 2.6.20 の新しさは、汎用の後戻りではなく
> **「Build&Test → Code Generation という区間限定・システムが自律的に発火・回数上限つき」**
> という特殊化された経路が加わったことである。

### レビュー受領証のソース束縛（2.6.37）

コード生成のレビュー受領証が、**レビュアーが実際に見たワークスペースのソース状態に束縛**された。
git ネイティブの一時 index ハッシュで、tracked / staged / unstaged / untracked / deleted /
再帰初期化した submodule までを含めたツリー内容をフィンガープリント化する。
`approve` / `advance` / `finalize` / `complete-workflow` の 4 経路すべてが照合し、
陳腐化していればブロックする。ソース編集を元に戻せば content-addressed なので束縛は自動的に復元される。

- **決定的なオフスイッチ `AIDLC_SKIP_SOURCE_FRESHNESS=1` が存在する。**
  これでバイパスした swarm finalize は `Source Freshness Bypass: true` を記録し、
  後続の `merge` でも同じスイッチの再指定が必要になる。
- 既知の制約（上流 RFC #662）として、フィンガープリントは**ワークスペース全体**への束縛であり、
  Unit 単位の帰属証明ではない。

受領証が後続の書き込みで無効化された場合は、2.6.9 により
**次の序数で 1 回だけ**回復リクエストが許される（advisory / adversarial いずれでも 1 回のみ）。

---

## 13.8 ハーネス別のアップグレード手順

**今回は `dist/` の再コピーだけで済まないハーネスが 4 つある**
（codex / copilot は必ず、kiro / kiro-ide はプラグインを使っている場合に限り）。

凡例: **○** = `dist/` の再コピーだけで完了 / **✗** = 再コピーに加えて**必ず**追加操作が要る /
**△** = 再コピーのみで済むが、**特定の条件下でのみ**追加操作が要る（kiro / kiro-ide はプラグイン利用時、
copilot は進行中ワークフローがある場合）。

**なお 2.6.36 の learning selections ファイルの非互換は、どのハーネスでも起こりうる。**
進行中ワークフローが 2.6.36 より前に生成された selections ファイルを持っていれば、
○ の行のハーネスでも該当ステージの `surface` 再実行が要る（→ 本節末）。

| ハーネス | 再コピー以外に**ハーネス固有の**操作が要るか | 追加操作 |
|---|---|---|
| **claude** | ○ | なし |
| **cursor** | ○ | なし（2.6.48 は元から具体パスだったため対象外） |
| **codex** | ✗ | **2.6.44**: `bun scripts/package.ts codex trust --project "<絶対パス>"` を再実行し、生成 TOML で既存 trust テーブルを差し替え、**新しいセッションを開始**する |
| **copilot** | △ | **2.6.48** の対処は再コピーそのもの。**2.6.12** により、**進行中ワークフローがある場合に限り新しい会話を開始**する必要がある |
| **opencode** | ○ | **2.6.48** の対処は再コピーそのもの（プレースホルダ持ちペルソナが具体パス版に置き換わる）。追加操作は無い |
| **kiro（CLI）** | △ | **2.6.46**: 再コピーで verb interceptor が直る。**2.6.47**: プラグイン利用時は projection を再ビルド／再コピーし、`aidlc plugin sync` か `hooks/compose.ts` を**明示実行**する |
| **kiro-ide** | △ | **2.6.47**: プラグイン利用時は projection を再ビルド／再コピー。新規 `.kiro/hooks/aidlc-<plugin>-compose.json`（SessionStart 登録）が自動で効くので**明示実行は不要** |

### Codex の trust テーブルは必ず作り直す

2.6.44 で追加されたフックが `.codex/hooks.json` の PostToolUse 配列の**先頭に挿入**される。
その結果、既存の trust エントリの `post_tool_use:N` というインデックスが**すべて 1 つずつズレる**。
`dist/codex/` を再コピーしただけでは、既存の trust ハッシュが対応しなくなり新フックが動かない。

### Kiro は CLI と IDE で対処が違う

- **2.6.46（verb interceptor）は Kiro CLI のみ。** Kiro IDE の `dist/` に同ファイルの変更は無い。
- **2.6.47（プラグイン compose 配線）は両方が影響を受けるが、対処が違う。**
  旧 `hooks/aidlc-plugin-compose.kiro.hook` は**双方の projection から削除された**。
  **ただし `cp -R` は削除を反映しないので、既存インストールでは旧フックが残る。手で消すこと**
  （`find your-project -name "aidlc-plugin-compose.kiro.hook"`）。
  CLI は hook 登録を出さなくなったので明示実行が要る。IDE は新しい JSON 登録が自動で効く。
  **このファイル名に依存したスクリプトがあれば外すこと。**

### learning の selections ファイルは作り直しになる

2.6.36 で `aidlc-learnings.ts persist` の冪等キーが、
位置ベースの候補 id（`c1`, `c2` …）から**学習内容自体の SHA-256** に変わった。
旧来は同一ステージを 2 回実行すると 2 回目も `c1` を使い、
**別内容の学習をサイレントに取りこぼしていた**。

アップグレード後、旧形式の selections ファイルは
`selections-json is malformed: missing or non-string space` で失敗する。
**対処は該当ステージの `surface` を再実行して再生成すること。**
`persist` を単純にリトライしても直らない。

### `dist/` の変更ファイル数はハーネスの働きの量ではない

opencode 139 / copilot 139 / codex 123 / claude 116 / kiro 115 / kiro-ide 114 / cursor 114。
ほぼ同数だが、これは**全ハーネスに等量の固有開発があったという意味ではない**。
共有コア（`aidlc-lib.ts`・`aidlc-state.ts`・プロトコル・14 ペルソナ・stage-runner の SKILL.md 等）が
機械的に全ハーネスへ投影されるための構造的な効果である。
opencode と copilot が突出するのは、同一内容を中立エンジン木（`.aidlc/`）と
ネイティブ殻（`.opencode/` / `.github/`）の**2 箇所に投影する**ためで、
basename の異なり数で比べると差は縮む（claude 116/78・copilot 139/87・cursor 114/76・kiro-ide 114/76）。
ハーネス固有の実質差分は数ファイル単位に集中している。

---

## 13.9 その他

- **2.6.41 ワークスペース検出**: 旧はワークスペース直下（1 階層）しか調べず、
  `services/api/src/main.py` のような 2 階層以上にネストしたソースを見落とし、
  **Greenfield と誤判定して Reverse Engineering をスキップしうる**穴があった。
  新は最大 3 階層まで再帰し、ヒットしたディレクトリはそこで探索を打ち切って相対パスで記録する。
  **4 階層目は依然として検出対象外である。**
  除外されるディレクトリ名はドット始まり全部、`node_modules` `dist` `build` `.next` `target` `vendor` に加え、
  `aidlc` `docs` `examples` `samples` `demos` `reference` `testdata` `fixtures` `templates` `scripts`。
- **2.6.49 flow-style YAML**: frontmatter の `keywords: [a, b]` という 1 行記法が
  **黙って空リストとして解釈されていた**。構文エラーにならないので気づけない種類の不具合である。
  影響を受けるのは主に**カスタムスコープを自作するチーム**で、
  出荷の `classic` / `express` 自体はブロックスタイルなので被害者ではない。
- **2.6.43 `--resume`**: 明示的な `/aidlc --resume` が Resume / Redo / Jump / Start Fresh の
  4 択メニューを飛ばして直接継続するようになった。裸の `/aidlc` は従来どおり 4 択を出す。
- **2.6.40 Stop フック**: no-progress カウンタの識別子が「監査末尾の長さ」から
  実際のディレクティブ遷移ベースに変わり、**監査への追記やタイムスタンプの更新だけでは
  カウンタがリセットされなくなった**。人間待ちの carve-out は 5 → 6 ケース。
- **2.6.14 監査ブロック**: `Timestamp` / `Event` はレンダラの専有フィールドになり、
  `park` / `unpark` / `practices-promote` の書き込み失敗パスが渡していた重複行が消えた。
  `audit append --field Timestamp=...` は受理されるが**値は無視される**。
  **既存シャードは無改修で読める** — 読み取り側は非 global の `match()` で
  ブロック内最初の `**Timestamp**` を拾う実装のままなので、重複行があっても常に正しい行を取る。
- **2.6.17 プラグインテストキット**: プラグイン作者向けに 3 層のテスト手段が入った。
  **dist 利用者には影響なし**（CHANGELOG 原文「No upgrade action is needed」）。

---

## 13.10 版番号の同定について — 今回の区間ではズレが無かった

[12.10](./12-release-impact-2602.md) で、コミット subject に書かれた `(2.5.x)` が
CHANGELOG の見出しと食い違う例が **12 件中 3 件**あったことを報告した。

**今回、28 コミットを機械的に照合した結果、食い違いは 1 件も無かった。**

ただし**これを「上流の運用が改善した」と読むのは早い**。比較できているのは 2 区間だけで、
前回 3/12・今回 0/28 という 2 点から傾向は言えない。言えるのは
**この区間ではズレが無かった**ということだけである。

ただし運用は完全ではない。CHANGELOG 見出しを追加した 24 コミットのうち、
**3 件は subject に版番号を書いていない**（`30d65cb0` → 2.6.48、`e229a606` → 2.6.41、
`244b52ba` → 2.6.37）。残り 4 コミットは版を伴わない変更（test 2 / ci 1 / docs 1）である。

**したがって「版の判定は `CHANGELOG.md` の `## [x.y.z]` 見出しで行う」という原則は維持する。**
subject の版表記は、あれば一致するようになったが、無いことがある。

欠番も引き続き存在する。2.6.3〜2.6.7 / 2.6.10 / 2.6.11 / 2.6.19 / 2.6.21〜2.6.35 は
公開 CHANGELOG に現れない。10 章・11 章・12 章で扱った現象と同じである。

---

## 13.11 本ノートの限界

- 本章は**上流リポジトリの差分の静的読解**に基づく。実機での再現は行っていない。
  特に「アップグレード手順」節の追加操作は、CHANGELOG の Upgrade 文と
  実ファイルの差分から導いたもので、実際に実行して確認したものではない。
- 上流のテストスイートは実行していない（調査環境に `bun` が無い）。
- `express` / `classic` の実際の走行、DocumentKB の索引、Testing Posture の解決、
  ループバックの発火は、いずれも**実機未検証**である。
- 「2.6.49 調査で残った未確認事項」は [docs/REMAINING_TASKS.md](./docs/REMAINING_TASKS.md) に列挙した。
