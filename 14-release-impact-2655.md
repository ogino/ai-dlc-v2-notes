# 14. リリース差分 2.6.49 → 2.6.55

作成日: 2026-08-22
基準: `awslabs/aidlc-workflows` branch `v2` HEAD `840ba653`（取得 2026-08-22）／実装バージョン **2.6.55**
前回: [13-release-impact-2649.md](./13-release-impact-2649.md)（2.6.2 → 2.6.49、HEAD `71d9a9e0`）
次回: [15-release-impact-26123.md](./15-release-impact-26123.md)（2.6.55 → 2.6.123、HEAD `2fbee12f`）

> **⚠ 本章の一部の記述は [15-release-impact-26123.md](./15-release-impact-26123.md) で訂正されている。**
> 本章は 2026-08-22 時点の測定記録なので本文は当時のまま残してある。
> どの記述が訂正されたかは 15 章を参照すること。

CHANGELOG の実エントリ **6 件**（2.6.50 / 51 / 52 / 53 / 54 / 55）。**欠番なし。**
コミット **6 件**、変更ファイル **195 件**（+7,533 −3,741 行）。

**今回は「何も動いていない」ことのほうが際立つ。** 13 章では
「`stage-graph.json` は `scopes` を除けば同一」と書いたが、今回は
**`stage-graph.json` も `scope-grid.json` もバイト単位で完全一致**する。
スコープ所属すら動いていない。中核メトリクスは**全項目が不変**である。

**変わったのは実行時の規則だけ**である。継続トークンの単回使用化、承認返答の literal 化、
レビュアー受領証の前倒し、監査の発火条件、ステージ日誌の先行作成。
そのうち [14.2](#142-継続カーソルの全ハーネス化2651-今回の中心) は
**アップグレードのやり方そのものに制約を課し、環境によっては動かなくなる**。

---

## 14.1 数値の変化 — 全項目不変

| 項目 | 2.6.49 | 2.6.55 | 測定方法 |
|---|---:|---:|---|
| フェーズ | 5 | 5 | `stage-graph.json` の `phase` 集計 |
| ステージ | 33 | 33 | 同（件数） |
| スコープ | 11 | 11 | `core/scopes/*.md` の件数 |
| エージェント | 14 | 14 | `core/agents/*.md` |
| センサー | 6 | 6 | `core/sensors/*.md` |
| TypeScript フック | 17 | 17 | `core/hooks/*.ts` |
| CLI tools | 41 | 41 | `core/tools/*.ts` |
| プロトコルモジュール | 8 | 8 | `core/aidlc-common/protocols/*.md` |
| 監査イベント種別 | 86 | 86 | `core/tools/aidlc-audit.ts` の `VALID_EVENT_TYPES` |
| 監査カテゴリ | 22 | 22 | `audit-format.md` の `## Event Registry (N events, M categories)` 見出し |
| ハーネス | 7 | 7 | `harness/` 直下（`dist/` 直下は `plugins` と PDF を含むため 7 にならない） |

**各項目はそれぞれ独立に、両リビジョンで数え直している。**
以下の 2 つの `diff` が示すのは**このうち最初の 2 項目に関わる構造データ**についてであって、
表の全項目を代表するものではない。

```
diff <(git show '71d9a9e0:<stage-graph.json>') <(git show '840ba653:<stage-graph.json>')  → 無出力
diff <(git show '71d9a9e0:<scope-grid.json>')  <(git show '840ba653:<scope-grid.json>')   → 無出力
```

### 変更 195 ファイルの内訳

| 領域 | ファイル数 | 内容 |
|---|---:|---|
| `dist/` | 119 | **7 ハーネス × 17 ファイルで完全に均等**（共有コアの機械的投影） |
| `tests/` | 40 | |
| `core/` | 16 | conductor / プロトコル 2 / ステージ 2 / フック 3 / 監査書式 / tools 6 ほか |
| `docs/` | 11 | |
| `harness/` | 7 | 各ハーネスの `skills/aidlc/SKILL.md`（**7 ハーネス × 1**） |
| ルート | 2 | `CHANGELOG.md` / `README.md` |
| **合計** | **195** | |

`dist/` が 7 で割り切れることが、今回の変更が**ハーネス固有ではなく共有コア由来**であることを示している。

---

## 14.2 継続カーソルの全ハーネス化（2.6.51）— 今回の中心

### 13 章の記述は誤りではない。一般化された

[13 章](./13-release-impact-2649.md)では 2.6.12 を
「**Copilot 固有**のアトミックなエンジンカーソル」として書いた。**それは 2.6.12 時点として正しい。**
当時の CHANGELOG 自身が「`sessionless:` and **non-Copilot continuation remain stateless in this release**」と
明記していた。

2.6.51 はそれを 7 ハーネスに一般化し、**2.6.12 が意図的に残した抜け穴**
（sessionless 経路での同一トークン再生、上流 issue #762）を塞いだ。
CHANGELOG も「closing the sessionless same-token replay carve-out **accepted in PR #749**」と書く。
PR #749 は 2.6.12 の導入 PR である（`git log --grep='#749'` が返す `d5b40846` の subject が
「stabilize Copilot Stop continuation and Resume waits (2.6.12) (#749)」）。

Copilot に残るのはセッション所有権と delivery evidence による**マーカーの enrichment だけ**になった。
上流ガイドの書き換え後の一文:

> Copilot's session ownership and delivery evidence **enrich** that marker but **do not own replay**.

### 何が塞がれたか

旧実装は継続の権威を `"stateless" | "current" | "superseded"` の 3 値で判定し、
`stateless` の場合は**提示トークンと現在トークンのダイジェスト比較を一切行わずに**後続を配信していた。
`stateless` になる条件は「ハーネスが copilot でない」「マーカーが v1／欠落／不正」
「`owner_session` が `sessionless:` で始まる」等である。

つまり **同じ継続トークンを 2 回提示すれば 2 回とも後続が返った。**

新実装では `authority` フィールドと `"stateless"` 戻り値そのものが削除され、
提示トークン全体の SHA-256 とマーカー内の `continue_token_sha256` を比較して、
一致した場合のみ後続へアトミックに置換する。不一致は明示的に拒否される。

### 単回使用はどう担保されるか

カーソルは**ハーネスディレクトリではなく記録ディレクトリ側**に置かれる。

```
aidlc/spaces/<space>/intents/<record>/.aidlc-active-directive.json
```

（intent 生成前は `intents/.aidlc-active-directive.json` = `bare-space` バケット）

手順は「排他ロック取得 → 現マーカー読込 → 候補を `openSync(..., "wx", 0o600)` で新規作成 →
**同一ファイルシステム内 `renameSync()` で publish** → ロック解放」で、
**rename が成功した後に初めて stdout が書かれる**。

---

### ⚠ 環境によっては動かなくなる — 今回いちばん重要な変更

**公開の失敗が、全ハーネスでハードエラーになった。**
旧実装は Copilot 以外なら例外を握り潰して、用意したディレクティブをそのまま配信していた（fail-open）。
新実装は打ち切る。

> The directive could not be published, so **no work directive was issued**.
> Retry the command; if coordination remains busy, run `/aidlc --doctor`.

そして上流は前提条件を明記している。

> The exactly-one-winner claim requires **one local filesystem** with coherent cross-process
> visibility, exclusive directory creation, stable regular-file reads, and atomic
> same-filesystem rename for the marker and lock paths.
> **NFS/SMB/FUSE/object-synchronized folders that do not honor those primitives are unsupported.**
> The implementation does **not `fsync`** the candidate or parent directory,
> so sudden power-loss durability is outside the claim.

**含意は 2 つある。**

1. **記録ディレクトリの置き場所が、初めて可用性の条件になった。**
   上流が要求するのは「アトミックな同一 FS 内 rename・排他ディレクトリ作成・
   プロセス間の一貫した可視性」であり、**これらを満たさない**ファイルシステムを unsupported としている。
   ネットワーク共有や同期フォルダが常にそうだとは上流も本ノートも言っていない
   （実装や構成によって満たす場合もある）。**言えるのは「満たさない構成では
   従来 fail-open で動いていたものが、作業ディレクティブを 1 つも出さない状態になりうる」**ことである。
   自分の構成が該当するかは実際に確認する必要がある。
   これはハーネス固有ではなく**環境固有**の問題なので、ハーネス別の表には現れない。
2. **保証されるのは「プロセス間で勝者が 1 つ」であって「電源断耐性」ではない。**
   `fsync` していないと上流自身が書いている。

なお `run-stage` にもマーカーが出るようになった。旧実装は state ファイル未生成の
pre-state 継続ではマーカーを書かなかったが、新実装は必ず書く。
1 勝者保証が intent 生成前まで広がる代わりに、**上の書き込み失敗リスクもそこまで広がる**。

### アップグレード手順 — 「再コピーだけ」では済まない

CHANGELOG の Upgrade 文は今回いちばん長い。分解すると次のとおり。

| 原文 | 何を要求しているか |
|---|---|
| `replace the whole dist/<harness>/ shell in one quiescent swap (no AI-DLC command or hook running)` | AI-DLC のコマンドが 1 つも走っておらず、フックも発火していない瞬間にコピーを完了させる |
| `mixed old/new tool files are unsupported` | **AI-DLC コマンドが全部落ちる。** 旧 orchestrate は削除済みシンボルを named import するため、モジュールのロードに失敗する。「挙動が混ざる」のではなく「起動しない」 |
| `then run a fresh next` | ステージ進捗は巻き戻らない。失うのは**ステージ規則の分割配信の途中位置**だけ |
| `a matching in-flight token instead migrates atomically` | project / intent / state / トークンダイジェストの **4 点すべてが一致**したときのみ移行する。1 つでもズレたら stale（**CHANGELOG の一文ではなく、`docs/reference/06-hooks-and-tools.md` の「Atomic continuation cursor」節が列挙するカーソル同一性の構成要素からの読み取り**） |
| `Rolling back to an older release restores its sessionless replay behavior until re-upgraded` | **ダウングレードはセキュリティ後退である。** #762 の再生の抜け穴が復活する。壊れはしないが弱くなる |

> **⚠ この制約は既存の「○ / ✗ / △」分類では表せない。**
> あの記号は「**ハーネスごとに**追加操作が要るか」を分類するものだが、
> 2.6.51 の制約は **7 ハーネスすべてに同一に効く**。7 行すべてが同じ印になり情報量がゼロになる。
> 本ノートでは記号を増やさず、**表の前に「全ハーネス共通のスワップ手順」を置き、
> ○/✗/△ は「その手順を満たしたうえで、さらにハーネス固有の操作が要るか」と定義し直した**
> （→ [06-harnesses-install.md](./06-harnesses-install.md)）。

> **版番号の罠。** 上流の未マージブランチ `origin/fix/issue-762-single-use-continuation` に、
> 同一件名で subject が `(2.6.19)` のコミットが存在する。しかしそれは `v2` の祖先ではなく、
> CHANGELOG に 2.6.19 の見出しは無い（2.6.18 の次が 2.6.20）。
> **出荷版は 2.6.51 である。**

---

## 14.3 承認チェックポイントの返答検証（2.6.50）

承認ゲートの返答が、**その時点で提示されている選択肢のラベルとの完全一致**を要求されるようになった。

```
approvalInput === "Approve" ||
(approvalInput === "Accept as-is" && revisionCount >= 3)
```

| 場面 | 受理される文字列 |
|---|---|
| 承認 | `Approve` |
| 改訂 3 回後のエスケープハッチ | `Accept as-is`（**`Revision Count >= 3` のときのみ**） |
| 却下 | `Request Changes` |
| 要約確認 | `Looks correct` / `Request changes`（**2.6.50 より前から完全一致**。`aidlc-log.ts` の `"Looks correct"` の出現数が 2.6.49 と 2.6.55 で同数であることを実測） |

`OK` / `いいね` / `approve` / `Approved` / `Looks good` はいずれも拒否される。

**改訂フィードバックが決定と別のフィールドになった。**

```
旧: report --stage <slug> --result rejected --user-input "<フィードバック本文>"
新: report --stage <slug> --result rejected --user-input "Request Changes" --reason "<フィードバック本文>"
```

内部でも `feedback = (flags.userInput ?? flags.reason)` から
`flags.reason ?? flags.userInput` へ優先順位が逆転した。
旧実装は**フィードバック本文を「選択された決定」として扱っていた**ことになる。

### ⚠ 2.6.50 も新旧混在で壊れる

これは 2.6.51 と**同じ型の制約**である（失敗モードだけが違う）。
旧散文 `stage-protocol.md` は `--user-input "Accept as-is after N cycles"` を指示していたが、
新ガードは `Accept as-is` の完全一致しか通さない。
旧散文 + 新ツールの組み合わせでは、**改訂ループのエスケープハッチが使えなくなる**。
`--result rejected --user-input "<feedback>"` も同様に拒否される。

2.6.51 の失敗が「コマンドが起動しない」なのに対し、2.6.50 の失敗は「**ゲートで詰まる**」である。
どちらも**ツールと散文を同時に入れ替えないと壊れる**。

### ⚠ これを「人間ゲートの強化」と要約すると過大評価になる

[13.6](./13-release-impact-2649.md) で
「人間の承認ゲートは presence evidence にすぎず fails open である」と書いた。
**その評価は 2.6.50 の後も変わらない。**

**前進した点** — 記録される決定の**忠実性**。
旧経路は「空でない任意の文字列」が `GATE_APPROVED` を commit できたので、
conductor による要約・言い換え・意訳がそのまま監査行になり得た。
今後 `GATE_APPROVED` に載る値は必ず提示された選択肢のどれかである。

**変わっていない点** — **作成者性は 1 ミリも証明されていない。**
人間が「OK」と言い、conductor がそれを `Approve` に翻訳して渡した場合、
エンジンには**区別できない**。それを禁じているのは散文だけである。

> **受理される文字列空間は狭く・完全に既知になった。**
> 旧来は「キャンセル定型文でなく、自己申告文言でもない任意の散文」が必要だったが、
> 今は `Approve` という 1 リテラルで足りる。
> **ただし「捏造が容易になった」と書くのも言い過ぎである。**
> 旧来も「それらしい散文を 1 つ書く」だけで通っており、どちらも決定論的な障壁ではない。
> **実測から言えるのは「2.6.50 は捏造を難しくしていない」ことまでで、
> 「難しさが下がった」ことまでは示されていない。**

逃げ道も従来どおり残っている。autonomous Construction は免除され、
`AIDLC_SKIP_HUMAN_PRESENCE_GUARD=1` は**この新ガードも含めて**全部無効化する。
`audit-format.md` の「`HUMAN_TURN` は operational evidence であって
tamper-proof な human-authorship boundary ではない」という記述は当区間で無変更である。

---

## 14.4 レビュアー受領証をゲート開放前に必須化（2.6.52）

### ツール側に決定的なガードが入った

`verifyReviewerPrecondition` の出現が **5 → 8**（定義 1 + 呼出 4 → 定義 1 + 呼出 7）に増えた。
追加されたのは `gate-start` / `revise` / `approve` 内の revision-backstop 再入路である。

**つまり `[?]` への遷移そのものが拒否されるようになった。**
従来は完了経路（`approve` / `advance` / `finalize` / `complete-workflow`）でしか見ていなかった。

```
Refusing to present the approval gate for "<slug>": it declares a reviewer ...
```

上流 issue #551 は、conductor 自身の自白がそのまま貼られた報告だった。

> I made a sequencing error on **rough-mockups**. Per protocol, the reviewer is supposed to run
> before the approval gate, but **I presented the gate and called `report --result approved` first**
> — which the audit log confirms already completed the stage and advanced the phase —
> and only then ran the reviewer and fixed the findings it raised.

2.6.52 はこの経路を塞いだ。

**珍しいことに、今回は実装が先で散文が追随している。**
`stage-protocol-reviewer.md` の差分は 2 箇所だけで、
"completion" → "**gate and** completion" という語の同期にとどまる。散文の要求は増えていない。

### ⚠ ただし「レビュアーが検査した」ことは依然として証明されていない

ここを取り違えると評価を誤る。**ツールが証明するのは何か**を分解する。

| 命題 | 誰が保証するか |
|---|---|
| ゲート開放・完了の前に `REVIEW_COMPLETED` 行が存在すること | **ツール（決定的）** |
| その受領証が現在の成果物のバイトに対応すること | **ツール（決定的）** |
| 配車から verdict までの間に成果物が書き換わっていないこと | **ツール（決定的、2.6.52 で新規）** |
| **レビュアー・サブエージェントが実際に起動したこと** | **保証なし** |
| **レビュアーが成果物を読んだこと** | **保証なし** |
| verdict（READY / NOT-READY）の値の正しさ | conductor（LLM） |
| `## Review` 節の検証・INCOMPLETE 判定・`--retry-pending` | conductor（LLM、散文のみ） |
| レビュアーの 60 ターン上限 | 7 ハーネス中 2 のみネイティブ（**当区間で不変**） |

`core/tools/aidlc-log.ts` は **`SUBAGENT_COMPLETED` を 1 度も参照しない**（出現 0 件）。
`.aidlc-reviewer-dispatch.json` も参照しない。

CHANGELOG の "prove that the configured reviewer **inspected** the **current artifact**" のうち、
**"current artifact"（どのバイトか）だけがツールの保証**であり、
**"inspected"（検査した）は依然として LLM のプロトコル遵守**である。

conductor は配車記録と受領証を 2 コマンドで連続発行でき、
**その間にレビュアーを一度も起動しなくてもツールは通す。**

したがって [13.5](./13-release-impact-2649.md) の
「incomplete-attempt guard は conductor 依存でツール側の強制ではない」という結論は**維持される**。
上流テスト `tests/unit/t279-reviewer-turn-budget.test.ts` は当区間で**1 バイトも変更されておらず**、
冒頭の "Mechanism: none. Pure content checks over authored + shipped bytes" もそのまま残っている。

### フィンガープリントは 3 点照合になった

| 項目 | 導入版 | 導入コミット |
|---|---|---|
| `Artifact Fingerprint`（`REVIEW_COMPLETED` 側） | 2.5.39 | `ebfa3cee` |
| `Source Fingerprint` | 2.6.37 | `244b52ba`（→ [13 章](./13-release-impact-2649.md)で既述） |
| **`Artifact Fingerprint`（`REVIEW_REQUESTED` 側）** | **2.6.52（新規）** | `a6f5e66c` |

> **⚠ 版を調べるときに `git log -S ... --all` を使ってはいけない。**
> `Source Fingerprint` を `--all` 付きで検索すると、**未マージブランチ**の `15d68f91`
> （subject に `(2.5.6)`）が返る。これは `v2` の祖先ではなく
> （`git merge-base --is-ancestor` が偽）、出荷版は `244b52ba` / 2.6.37 である。
> [14.8](#148-版番号の同定について) の `(2.6.19)` と同じ罠である。

旧実装は「記録時のフィンガープリント」と「現在のバイト」しか比べておらず、
**配車後・verdict 記録前の書き換えを検出できなかった**。2.6.52 がこの穴を塞いだ。

新しいバイパス env として `AIDLC_SKIP_REVIEWER_GATE_GUARD=1` が追加されたが、
**ゲート開放にのみ効き、完了 4 経路には効かない**
（コード内コメント: "Completion paths never honor this variable."）。

---

## 14.5 監査の発火条件（2.6.54）— ゲートは 2 段階で締められてきた

`SUBAGENT_COMPLETED` を書く条件が変わった。**ただしこれは 1 回の変更ではない。**

| 期間 | ゲート条件 | 何が漏れるか |
|---|---|---|
| v2-unified 〜 **2.6.18** | 監査シャードが在るか（`existsSync(auditFile)`） | 一度でも AI-DLC を動かしたプロジェクトなら、**AI-DLC と無関係なセッション中でも書く** |
| **2.6.20 〜 2.6.53** | 状態ファイルが在るか（`existsSync(stateFilePath)`） | 完了後も状態ファイルは残るため、**完了後・無関係セッション中も書く** |
| **2.6.54** | `Status: Running` | — |

この 3 段階は `git log -S` で確認した。

```
git log -S'existsSync(auditFile)' -- core/hooks/aidlc-log-subagent.ts
  868f0256  2.6.54  ← 削除（Status: Running に置換）
  85a9443c  2.6.20  ← 状態ファイル存在チェックに置換
  7b824b34  v2-unified  ← 導入
```

上流 CHANGELOG の書き方もこれと整合する。

> instead of writing whenever **a state file or audit shard** remains from an earlier workflow

「state file **or** audit shard」と 2 つ並記しているのは、旧ゲートが時期によって 2 種類あったためである。
（ただしこの一文だけでは「2 つが順に置き換わった」とまでは言えない。順序は上の `git log -S` で確定させた。）

**このフックは常時発火する。** `SubagentStop` は matcher が空（= always）で
プロジェクト全体に登録されるため、**AI-DLC と無関係な Task ツール呼び出しでも起動する**。
だからゲートの緩さがそのまま「無関係なエントリの混入」に繋がっていた。
併せて、ヘルスハートビートの書き込みも**無条件だったものがゲートの後ろに移動**した。

> **監査ログを証跡として読むときの注意。**
> **2.6.54 より前に生成された監査シャードには、そのワークフローと無関係な
> `SUBAGENT_COMPLETED` が混入している可能性がある。**
> ただしこれは**コード構造から導かれる可能性**であって、
> 実データでの混入を確認したものではない（本ノートの調査環境に監査シャードのサンプルが無い）。

監査イベントの**種別数は 86 で不変**であり、`SUBAGENT_COMPLETED` のフィールド構成も変わっていない。
**発火条件だけが変わった。**

---

## 14.6 ステージ日誌の先行作成（2.6.55）

「stage diary」は `memory.md`（`<record>/<phase>/<stage>/memory.md`）のことで、
Interpretations / Deviations / Tradeoffs / Open questions の 4 見出しに
conductor が観察を追記していく日誌である。**既存の概念で、2.6.55 の新設ではない。**

変わったのは**誰がいつ作るか**である。

- **旧**: conductor が「無ければテンプレートをコピーする」。
  初回入場では必ず不在なので、**存在しないパスへの read ツール呼び出し**が事実上の既定動作だった。
- **新**: エンジンが run-stage 指令を組み立てる時点で作成する。散文は禁止に変わった。

> **NEVER probe for `memory.md`, or any other maybe-absent file, with a read tool:
> reading an absent path is a failed tool call.**

**この結果 `aidlc-orchestrate next` の書き込みが 1 つ増えた。**
**ただし「これで `next` が初めて書き込むようになった」わけではない。**
同じ区間の 2.6.51 が、`run-stage` を含むディレクティブ発行のたびに
`.aidlc-active-directive.json` をアトミックに公開するようになっている（→ [14.2](#142-継続カーソルの全ハーネス化2651-今回の中心)）。
2.6.55 の時点で `next` は既に書き込む側であり、日誌の作成は**もう 1 つの書き込みの追加**である。
run-stage 指令を出す際に実際にファイルを作る。
ただし Stop フックの内部プローブは `AIDLC_STOP_HOOK_PROBE=1` により従来どおり write-free である。
（プローブが書き込むと「エンジンのほうが人間より新しい」が常に真になり、
transcript を持たないハーネスの会話ターン判定が壊れるため。）

> **⚠ 問題は全ハーネス共通、観測は Kiro CLI の ACP 経由の実行に固有。**
> （**ACP はハーネスではなく、Kiro CLI の接続方式である。** ハーネスは 7 種のままで、
> ACP はそのうち Kiro CLI に対する経路の 1 つ。ACP 経路だけが ndjson トレースを残すため、
> 回復済みの失敗ツール呼び出しがそこでのみ可視化された。
> **Kiro IDE は別のハーネスであり、この観測の対象ではない。**）
> 原因である散文は `core/aidlc-common/conductor.md` にあり、**7 ハーネスすべてに投影される**。
> ACP が特別なのは**検出器がそこにしか無いから**で、ndjson トレースが tool_call を逐一記録するため
> 回復済みの失敗ツール呼び出しが唯一可視化される。
> 上流 issue #695 自身が「テストハーネス外では不可視」「回復済み失敗を監査する仕組みが無い」と明言している。
> **Kiro IDE での実測は無い**（Kiro CLI とは別物）。

per-Unit Construction の指令には 28 KiB のトランスポート上限があり
（`core/tools/aidlc-orchestrate.ts` の `const DIRECTIVE_MAX_BYTES = 28 * 1024`）、
**指令に実際に載った Unit の日誌だけ**が作られる。
上限で載らなかった Unit の日誌は、後続の指令がそれを運ぶまで未作成のままである。

---

## 14.7 Kiro IDE のフックペイロード記述の訂正（2.6.53）

上流が**自分の記述を訂正した**リリースである。フック登録と enforcement の挙動は変わっていない。

- **旧**: 「Kiro IDE exposes no tool arguments to hooks」など、**全世代で空**と読める断定
- **新**: 「tool-argument delivery is **not uniform across supported generations**」

注目すべきは、上流が**実測と伝聞を明示的に区別した**ことである。

| 世代 | 内容 | 根拠 |
|---|---|---|
| 0.12 / 1.0.165 | PostToolUse の tool input が空（`{}`） | **上流リポジトリで実測** |
| 1.0.309 | PreToolUse に `prompt` / `explanation`、shell/write matcher に `command` / `cwd` 等が入る | **報告のみ**（issue #763）。上流自身が「reported, **not measured** in this repository」と明記 |

**本ノートに訂正は不要だった。** [06-harnesses-install.md](./06-harnesses-install.md) と
[13 章](./13-release-impact-2649.md)を走査したが、
「tool arguments が空」という断定は**存在しなかった**。

---

## 14.8 版番号の同定について

**6 件すべて、コミット subject の版表記と CHANGELOG 見出しが一致した。欠番も無い。**

13 章では「24 件中 3 件が subject に版番号を書いていなかった」と記録したが、今回は全件記載されている。
ただし [13.10](./13-release-impact-2649.md) に書いたとおり、
**これを「上流の運用が改善した」と読むのは早い**。比較できているのは 3 区間だけである。
言えるのは**この区間ではズレも欠番も無かった**ということだけで、
版の判定を `CHANGELOG.md` の `## [x.y.z]` 見出しで行う原則は変えない。

実際、今回は**別の形の罠**があった。未マージのリモートブランチに
subject が `(2.6.19)` のコミットが存在し、これは `v2` の祖先ではない
（→ [14.2](#142-継続カーソルの全ハーネス化2651-今回の中心)）。
**subject を見て版を決めると、出荷されていない版番号を書いてしまう。**

---

## 14.9 本ノートの限界

- 本章は**上流リポジトリの差分の静的読解**に基づく。実機での再現は行っていない。
- 上流のテストスイートは実行していない（調査環境に `bun` が無い）。
- 「新旧ツールファイルの混在はロードエラーになる」は、削除されたシンボルと
  named import の関係からの**推論**である。実際に半々のツリーを組んで実行してはいない。
- ネットワークファイルシステム上での挙動は、**上流が unsupported と宣言している事実のみ**を確認した。
  実際にどのタイミングでどのエラーが出るかは未測定。
- 2.6.54 より前の監査シャードへの混入は**コード構造から導かれる可能性**であり、
  実データで確認したものではない。
- 「2.6.55 調査で残った未確認事項」は [docs/REMAINING_TASKS.md](./docs/REMAINING_TASKS.md) に列挙した。
