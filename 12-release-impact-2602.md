# 12. リリース差分 2.5.62 → 2.6.2

作成日: 2026-08-14
基準: `awslabs/aidlc-workflows` branch `v2` HEAD `4569754e`（**コミット日 2026-08-14 / 取得 2026-08-14**）／実装バージョン **2.6.2**
（CHANGELOG の 2.6.2 見出しは `- 2026-08-13` と書かれているが、これは上流が記した**リリース日表記**であり、
コミット日とは 1 日ずれる。本ノート内で日付が食い違って見える箇所はこの違いによる。）
前回: [11-release-impact-2562.md](./11-release-impact-2562.md)（2.5.37 → 2.5.62、HEAD `2ce654d1`）
次回: [13-release-impact-2649.md](./13-release-impact-2649.md)（2.6.2 → 2.6.49）

CHANGELOG の実エントリ **12 件**（2.5.63 / 64 / 67 / 68 / 69 / 71 / 72 / 73 / 74 / 75 / 2.6.1 / 2.6.2）。
変更ファイル **1,083 件**（新規 361・削除 14）。うち **276 件が新規ハーネス Cursor**（`dist/cursor` 259 ＋ `harness/cursor` 17）。

**この差分は前回までと性質が違う。** 2.5 系にも成果物の追加や移動はあったが（2.5.71 の `traceability`、
2.5.73 のパス移動、2.5.75 の可観測性成果物）、**ステージの構成そのものは 32 本のまま動かなかった**。
2.6.1 は**設計ステージの成果物名を作り直し、永続化状態のスキーマを上げた**。
進行中のワークフローは 2.6.2 で先に進められず、自動移行も無い（→ [12.3](#123-移行手順必ず読むこと)）。

---

## 12.1 数値の変化

| 項目 | 2.5.62 | 2.6.2 | 測定方法 |
|---|---:|---:|---|
| フェーズ | 5 | 5 | `core/aidlc-common/stages/` の直下ディレクトリ |
| **ステージ** | **32** | **33** | 同配下の `*.md`（再帰） |
| エージェント | 14 | 14 | `core/agents/` |
| スコープ | 9 | 9 | `core/scopes/` |
| **センサー** | **5** | **6** | `core/sensors/` |
| 監査イベント種別 | 82 | 82 | `aidlc-audit.ts` の `VALID_EVENT_TYPES` |
| TypeScript フック | 17 | 17 | `core/hooks/*.ts` |
| フック登録数（`dist/claude`） | 18 | 18 | `settings.json` の `hooks` を JSON パースして計上 |
| **CLI tools** | **37** | **38** | `core/tools/*.ts` |
| **ハーネス** | **6** | **7** | `harness/` 配下 |
| アーティファクト語彙 | 122 | 122 | アーティファクトレジストリ |

フェーズ別のステージ内訳は、増えたのが inception だけである。

| フェーズ | 2.5.62 | 2.6.2 |
|---|---:|---:|
| initialization | 3 | 3 |
| ideation | 7 | 7 |
| **inception** | **8** | **9** |
| construction | 7 | 7 |
| operation | 7 | 7 |

> **「変わっていない」数値ほど数え直した。** ideation と operation はファイル一覧を `diff` にかけて
> 基準と完全一致することを確認している。監査イベント 82 は、`core/tools/aidlc-audit.ts` と
> `core/knowledge/aidlc-shared/audit-format.md` の `git diff --stat` がいずれも**無出力**（＝ファイル自体が無変更）
> であることまで確認した。

> **ただし「イベント種別が同数＝監査に変化なし」ではない。** 記録される行の中身は変わっており、
> 例えば `ARTIFACT_CREATED` / `ARTIFACT_UPDATED` の File 値に `.json` が現れるようになった
> （従来は `<name>.md` 決め打ちで、JSON 成果物が監査を素通りしていた）。

> **アーティファクト語彙 122 → 122 は「差し引きゼロ」であって「無変更」ではない。**
> 途中 126 まで増え、2.6.1 の再編で −9 / +5 されて 122 に戻っている。

> **時点限定**: 本節の数値は 2.6.2 時点の実測。2.6.49 ではスコープが 11、
> 監査イベントが 86 種（22 分類）、CLI tools が 41 に変わっている。
> なお本節の「アーティファクト語彙 122」はアーティファクトレジストリを数えたもので、
> 13 章が載せる「`produces` 異なり値 121」とは**計数対象が違う**。2 つの数字を直接比較しないこと。
> フェーズ 5・ステージ 33・エージェント 14・センサー 6・フック 17・ハーネス 7 は 2.6.49 でも不変。
> → [13-release-impact-2649.md](./13-release-impact-2649.md)

---

## 12.2 設計ステージの再編（2.6.1）— 今回の中心

`application-design`（**ステージ 2.6**）が **`domain-design`** に改名され、
**`contract-design`**（**ステージ 2.8**、CONDITIONAL）が新設された。
`delivery-planning` は**ステージ** 2.8 → 2.9 へ繰り下がる。
※ ここでの `2.6` / `2.8` / `2.9` は**ステージ番号**であり、上流のバージョン番号ではない。CHANGELOG は上流自身の言葉でこう書いている。

> The framework now ships **33 stages** (was 32).

### 消えた成果物と、その行き先

`application-design` が出していた 5 本のうち、名前が残るのは `components` と `decisions` の 2 本だけである
（ただし `components.md` は名前が同じでも中身が作り直されている）。残る 3 本は吸収・分割・廃止された。

| 旧成果物 | 行き先 |
|---|---|
| `components.md` | `domain-design` の `components.md` へ（fenced YAML のカタログ＋そこから導出する人間向けビューに再設計） |
| `component-dependency.md` | `components.md` の `depends_on` / `dependents` に吸収 |
| `services.md` | インフラ側は `components.md` の `external_dependencies`、通信契約側は `contract-design` へ分割 |
| `component-methods.md` | **廃止**。メソッド／API レベルは Contract Design と Functional Design で確定させる |
| `decisions.md` | 維持・強化（ADR が `ADR-NNN` 連番になり、4 項目目が `Alternatives Rejected` に統一） |

Functional Design と Infrastructure Design の成果物も置き換わった。

| ステージ | 旧 | 新 |
|---|---|---|
| functional-design | `business-logic-model` / `business-rules` / `domain-entities` | `functional-spec` / `rules` / `entities` |
| infrastructure-design | `deployment-architecture` / `infrastructure-services` / `shared-infrastructure` | `infrastructure-specification`（`shared-infrastructure` は CONDITIONAL セクションに格下げ） |
| infrastructure-design | `monitoring-design` / `cicd-pipeline` | 独立のまま維持 |

**消える成果物名は 9 個、現れるのは 5 個である**（12.1 のアーティファクト語彙 −9 / +5 と一致する）。

| 消える 9 個 | 現れる 5 個 |
|---|---|
| `component-methods` / `services` / `component-dependency` | `contract-summary` |
| `business-logic-model` / `business-rules` / `domain-entities` | `functional-spec` / `rules` / `entities` |
| `deployment-architecture` / `infrastructure-services` / `shared-infrastructure` | `infrastructure-specification` |

これとは別に、**ステージ側でも 2 件動く**（`application-design` → `domain-design` の改名と、
`delivery-planning` のステージ番号 2.8 → 2.9）。成果物名を参照している自作ツールは前者 9 個を、
ステージ名や番号を参照しているものは後者 2 件を手当てすること。
`core/` と `dist/` の側に旧名は残っていない（残っているのは CHANGELOG、後述の `docs/rfcs/`、
移行ガードのエラーメッセージ本文、テストの negative assertion のみ）。

### 新設された contract-design

Unit をまたぐ境界と、外部に公開する API を先に固定して並行開発できるようにするためのステージである。
per-unit ではなく**ワークフローにつき 1 回**走り、`contract-summary.md` を出す。
CONDITIONAL で、上流の記述では次の場合だけスキップされる。

> Skip only for a single self-contained unit with no inter-unit boundaries and no externally consumed API

対象スコープは `enterprise` / `feature` / `mvp` / `workshop`。

### CHANGELOG に書かれていない影響

`produces_kinds` の変更により、**`ui` kind のユニットが新たに `monitoring-design.md` を負う**。
2.5.62 では `monitoring-design` の対象は `[service, packaging]` で ui は含まれていなかったが、
2.6.2 では `[service, ui, packaging]` になっている。ui ユニットを回している場合、生成される成果物が増える。
**CHANGELOG にこの記載は無い。**

> 一方 `infrastructure-specification` のほうは ui にとって**新規の負担ではない**。
> 統合前の `deployment-architecture` が既に `[service, ui, packaging]` を対象にしており、
> 同じ枠が名前を変えただけである。増えたのは `monitoring-design`（と、全 kind 対象の `traceability`）だけ。

---

## 12.3 移行手順（必ず読むこと）

**進行中のワークフローは 2.6.2 では先に進められない。** 永続化状態のスキーマが v7 → v8 に上がり、
`next` / `report` / `/aidlc --doctor` のすべてが旧状態を拒否する。
（正確には、拒否が確認できているのはこの 3 経路である。`park` / `continue` と各フックには
同じ状態バージョン検査が掛かっていないことを確認しており、それらの挙動は**未検証**。）

上流はこれを意図的な設計として明言している（`core/tools/aidlc-utility.ts`）。

> The framework ships no user-visible migration pre-1.0, so fail loud here with archive-and-reinit guidance rather than let a stale-graph state look healthy.

**状態スキーマの移行コードは 1 行も存在しない。** 取れる手は 2 つだけである。
（`core/` には `aidlc-docs/` の配置変更にともなうディレクトリ移行コードが別途あるが、
状態スキーマとは無関係で、v7 の状態を v8 に変換するものではない。）

1. 進行中のワークフローを**旧シェルで完走させてから**上げる
2. 状態をアーカイブして（例: `mv aidlc aidlc.v7-archive`）作り直す

ガードが無ければどう壊れたかも、同じコメントに書かれている。存在しないステージ名の行を持つ状態が
「チェックボックスは黙って no-op のまま `Current Stage` だけ進み、`report` で落ちる」——
**失敗が遅れて出る**壊れ方だった。早期に落とすためのガードである。

### `dist/` の再コピーだけでは終わらない

CHANGELOG が名指しで警告している。`cp -R` によるマージコピーは新しいランナーを**足す**が、
古いランナーを**消さない**。

> **delete the renamed stage-runner skill left behind by the merge copy** — the old `skills/aidlc-application-design/` directory (now `skills/aidlc-domain-design/`)

残ると、グラフに存在しないステージを起動する死んだ `/aidlc-application-design` ランナーになる。
**`--doctor` にこの残骸を検出するチェックは無い**（`aidlc-utility.ts` を読んで確認。
本ノートは上流のテストを実行していないので、**静的な確認であって実機検証ではない**）。手で消すしかない。

検出も削除も自前でやる必要がある。プロジェクト直下で:

```bash
# 残骸の検出（ハーネスによって .claude / .kiro / .cursor / .aidlc / .codex 配下）
find . -type d -name 'aidlc-application-design'
# 見つかったら削除
find . -type d -name 'aidlc-application-design' -exec rm -rf {} +
```

なお上流が配布する `dist/` 自体は正しく、旧名のスキルディレクトリは含まれていない。
つまりこれは**利用者側のアップグレード操作でのみ起きる問題**である。

### ビジネスルール ID の書き換え

2.6.1 の指示どおり `BR-NNN` 形式で `rules.md` を書いていると、トレーサビリティ・センサーが落ちる。
`BRx.y`（例 `BR1.1`）へ手で直す必要がある。**自動変換ツールは無い。**

これは 2.6.1 が持ち込んだ自己矛盾を 2.6.2 が塞いだもので、経緯は次のとおり。
2.6.1 でセンサー側に `BR{group}.{seq}` を要求する正規表現が入ったが、エージェントへの指示側
（ステージ本文・ナレッジ・org メモリ）は `BR-NNN` のままだった。2.6.2 はセンサーではなく**指示側を**揃えた。

### 2.6.2 が塞いだ状態ガードの穴

2.6.1 のガードは**素通りする経路が複数あった**。2.6.2 の修正内容から逆算すると、2.6.1 では
State Version が欠落・空・ゼロバイトの状態がガードをすり抜け、`State Version: 8 garbage` も v8 として通り、
v9 のような未来のバージョンには誤って「アーカイブして作り直せ」と案内していた。
**2.6.1 だけを入れた環境は、移行ガードが効いていない可能性がある。2.6.2 まで上げること。**

---

## 12.4 Cursor ハーネス（2.5.63 / 2.5.69）

7 つ目のハーネスとして Cursor が加わった。`dist/cursor` の 259 ファイルはすべて生成物である。
手書きは `harness/cursor/` の 17 ファイルのうち 15 で、残る 2 は生成データ（`harness.json` / `stage-graph.json`）。
規模は他ハーネス（claude 251・codex 305・kiro 265）と同程度で、突出して大きいわけではない。

Cursor 固有なのは主に配布方法である。ハーネス唯一の同梱インストーラ（`install.ts`、1,131 行）を持ち、
Cursor が @-import を展開しないため、ルールファイル（`.mdc` 5 本）は「import せよ」ではなく
「読め」という指示文になっている。ツール本体は Claude 版とほぼバイト一致で、共通の枠組みに素直に乗っている。

権限は 2 層で、`cli.json` の allow リスト（`Shell(bun)` のみ）と、`preToolUse` フックの `failClosed` である。

2.5.69 の修正は、この 2 層目の**許可側**のバグだった。正常系で標準出力に何も書かずに終了していたため、
空出力を不正 JSON とみなす IDE 側では**すべてのツール呼び出しが拒否され**、
沈黙を許可と解釈する CLI 側では通っていた。**CLI だけで検証すると見つからない類のバグである。**

### 既存ハーネス利用者への影響

Cursor 追加そのものによる影響は無い。ただし同じ範囲に入った `aidlc-review-freeze` の変更は
**全ハーネスに効く**。`sudo` / `env -S` / `xargs` / `timeout` / `nice` / `exec` といったラッパを剥がしてから
実コマンドを判定するようになったため、**従来素通りしていたラップ済みの書き込みが今後ブロックされうる**。
方向としては厳格化だが、挙動は変わる。

---

## 12.5 トレーサビリティ（2.5.71）と可観測性（2.5.75）

6 本目のセンサーとして `traceability` が追加された。要求 ID から成果物への対応表
（`traceability.json`）を検査し、上流 ID と実際のカバレッジを**双方向**で突き合わせる。
参照先が実在するかまで見る点と、構造化 JSON を入力に取る唯一のセンサーである点が既存 5 本と違う。

**「センサーが advisory だから影響は無い」と読むと誤る。** センサー自体は非ブロッキングだが、
`traceability` が 8 ステージの `produces` に追加された結果、**per-unit の 5 ステージでは
成果物の実在チェックが `UNIT_COMPLETED` を拒否する**。上流も CHANGELOG でこう案内している。

> revisit that stage and create its declared traceability artifact before attempting completion

2.5.75 では可観測性が NFR の一級成果物になり、`observability-requirements`（nfr-requirements 産出）と
`observability-design`（nfr-design 産出）が追加された。`monitoring-design.md` は
「その戦略のプラットフォーム固有実装」に再定義されている。
これらも `produces` 経由の同じハードガードに掛かるため、service kind のユニットでは新たな失敗要因になる。
テンプレートや必須セクションの追加は無い。

---

## 12.6 実行モデル: wave（2.5.67）

Construction の per-unit 実行が、Bolt の DAG から作った 1 バッチを 1 個の `run-stage` に載せる
「wave」方式になった。対象は **4 ステージのみ**である。

| wave 対象 | 非対象 |
|---|---|
| functional-design / nfr-requirements / nfr-design / infrastructure-design | code-generation（`workspace_requires: true` のため恒久的に除外） |

**利用者が並列度を指定するノブは無い。** 並列度は DAG とディレクティブのサイズ上限から自動的に決まる。
止める手段は `Construction Iteration: unit-major` に切り替えることだけである。

コストへの影響は一方向ではない。壁時計時間は短くなるが、各ビルダーに親ステージファイルと
インラインコンテキストを逐語で複製配布するため、**トークン消費は増える可能性が高い**。
ただし上流にベンチマークは無く、これは実装から読み取った推定である。

**アップグレード前に中断していた継続トークンは失効する。** `next` で入り直す必要がある。

---

## 12.7 Kiro の MCP レジストリ（2.5.74）

**Kiro CLI に**（Kiro IDE ではない）MCP サーバのレジストリが同梱されたが、**既定ですべて無効**である。
`mcp.json` を持つのは `dist/kiro` のみで、`dist/kiro-ide` には無い。per-agent の付与も Kiro CLI の 14 ペルソナだけで、
他 6 ハーネス（claude / codex / copilot / cursor / kiro-ide / opencode）のエージェント定義に `@server` は 1 件も無い。

**「使えるようにする」と「承認を省く」は別物なので、分けて理解すること。**

| 段階 | 何のためか | 出荷時の状態 | 利用者の操作 |
|---|---|---|---|
| サーバの `disabled: true` を外す | **利用可能化** | 5 サーバすべて `disabled: true` | **同梱 5 本ではこれが唯一の必要操作** |
| ペルソナの `includeMcpJson` | ペルソナに MCP 設定を見せる | **14 ペルソナすべてで既に `true`** | 不要（上流が設定済み） |
| `allowedTools` への `@server` 追加 | **呼び出しの自動承認** | **どのペルソナにも無い** | **不要。付けると都度承認が消える** |

つまり `disabled` を外すだけでサーバは使えるようになり、そのうえで**呼び出しのたびに承認を求められる**。
`@server` を `allowedTools` に足すのはその承認を省くための操作であって、動作させるために必要なものではない。
**動かすつもりで `@server` を付けると、必要のない自動承認まで与えてしまう。**

> **この表は同梱の 5 本についての話である。** 同梱されていないサーバ（例: CodeKB）を使う場合は、
> `mcp.json` への追加と、ペルソナの `tools` への `@<server>` 付与の**両方が別途要る**
> （→ [04-agents.md](./04-agents.md) 4.4）。`tools` への付与は到達可能にするためのもので、
> `allowedTools` とは別物なので、その場合も呼び出しごとの承認は残る。

設計上重要なのは、上流が**その `@server` をどのペルソナにも与えていない**ことである
（14 ペルソナすべてで `allowedTools` を確認。実際の値は `fs_read` / `thinking` 等のみで、
テストが不変条件として固定している）。
なおサーバ自体は 14 ペルソナの `tools` には並んでおり、**到達はできるが毎回承認が要る**という形になっている。

**指揮役（conductor）はこの 14 に含まれない。** `dist/kiro/.kiro/agents/` には 15 個の設定があり、
うち 14 が `*-agent.json`（ペルソナ）、残る 1 つが指揮役の `aidlc.json` である。
指揮役は `includeMcpJson` キーを**持たず**、`allowedTools` も `fs_read` / `thinking` / `todo_list` のみで、
`tools` にも `@server` が無い。**指揮役だけは MCP に到達しない。**

参考までに、Claude 版の MCP 設定には `disabled` フィールドが無く既定で有効なので、
**この点では Kiro のほうが保守的**である。

**何もしなければ従来どおり動く。後方互換である。**

---

## 12.8 その他

| 版 | 変更 | 影響 |
|---|---|---|
| 2.5.64 | composed-workflow の決定性とテスト基盤の強化 | テストログのディレクトリ名が変わり、`validate-grid` が全ステージの明示エントリを要求 |
| 2.5.68 | Codex のモデル指定を更新（`gpt-5.4` → `gpt-5.6-terra`） | 再コピーのみ |
| 2.5.72 | `*-design` ステージの成果物を設計レベルに制限（コードを書かせない） | 散文の制約が増えただけで、**機械的な強制は無い**（新規テストもセンサーもゼロ） |
| 2.5.73 | `unit-test-instructions.md` が build-and-test から per-unit の code-generation へ移動 | **パスが変わる**。外部から参照していれば修正が要る |

---

## 12.9 上流ドキュメントの読み方の注意 — `docs/rfcs/` は仕様書ではない

今回 `docs/rfcs/` が新設され、4 ファイルが入った。**これらを現行仕様として引用してはならない。**
上流自身が作業用メモだと明記している。

| ファイル | 冒頭の自己申告（逐語） |
|---|---|
| `IMPLEMENTATION-PLAN.md` | `Working plan for review. Not committed with the feature work — this file is scratch for the implementation conversation.` |
| `reviewer-reliability-and-stage-decomposition.md` | `**Status:** Draft for team discussion` |

コミットしないと宣言されたファイルが実際にはコミットされている。事実として記録するに留め、
上流の意図は推測しない。

**記述は実装より古い。** RFC が「これから追加する」としている `REVIEW_REQUESTED` / `REVIEW_COMPLETED`
監査イベントは、**基準の 2.5.62 時点で既に出荷済み**だった（Track 1 は実装済み、Track 2 は未出荷）。

**計画と出荷状態も食い違う。** 計画書は合計 32 ステージ維持を目標に、nfr-requirements の
nfr-design への統合と `blueprint-shape` センサーの追加を含んでいたが、**どちらも 2.6.2 時点で未出荷**である。
安定 ID スキーム（`cmp-NNN` / `ent-NNN` / `rule-NNN`）も未出荷で、component の識別子は名前のままである。
実際に出荷されたステージ数は、CHANGELOG が言うとおり **33** である。

→ **数値は計画書ではなく CHANGELOG と実ファイルから取ること。**
なお上流の通常ドキュメント（`docs/guide/` など）の側は 33 に追従済みで、`32 stages` の記述は残っていない。

---

## 12.10 版番号の同定について

**コミットの subject 行に書かれた版番号は当てにならない。** PR ブランチ時点の番号が残っているためで、
今回の 12 件中 3 件が CHANGELOG の見出しと食い違っていた。

| コミット subject の表記 | 実体 |
|---|---|
| `(2.5.60)` composed-workflow determinism | 2.5.64 |
| `(2.5.66)` cursor preToolUse | 2.5.69 |
| `(2.5.71)` kiro MCP registry | 2.5.74 |

さらに **2.5.65 / 2.5.66 / 2.5.70 は欠番**である（基準時点でも 2.5.61 が欠番で、
`2.5.60` が `2.5.62` より新しい日付という逆転もある。欠番は今回に限った現象ではない）。

版の同定は **`CHANGELOG.md` の見出しと `core/tools/aidlc-version.ts`** のみを根拠とする。

---

## 12.11 本ノートの限界

- **上流のテストは実行していない。** 本ノートの記述は静的な読解と `git` による差分・件数の実測に基づく。
  実行環境（`bun`）を用意していないため、`dist/` のドリフト検証もテストも走らせていない。
- **既存ワークフローでの移行を実地検証していない。** v8 ガードの挙動はコードとテストの読解による。
- 欠番（2.5.61 / 65 / 66 / 70）の理由は**未確認**。
- wave のコストと所要時間への実影響は**未計測**（上流にベンチマークが存在しない）。
- Cursor 本体の `failClosed` と権限 JSON の一次仕様は未確認（根拠は上流リポジトリ内の記述のみ）。
- 2.5.72 の「コードを書かせない」制約が実際に守られるかは、機械的強制が無い以上**モデル任せ**であり、
  効果は未検証。
