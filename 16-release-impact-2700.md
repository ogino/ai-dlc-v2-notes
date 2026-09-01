# 16. リリース差分 2.6.123 → 2.7.0 — ブランチ再編

作成日: 2026-09-01
基準: `awslabs/aidlc-workflows` branch `main` HEAD `96b11d39`（取得 2026-09-01）／実装バージョン **2.7.0**
前回: [15-release-impact-26123.md](./15-release-impact-26123.md)（2.6.55 → 2.6.123、HEAD `2fbee12f`）

CHANGELOG の実エントリ **2 件**（2.6.124 と 2.7.0）。
コミット **4 件**、変更ファイル **74 件**（+1,056 −306 行）、期間 **4 日**（上流コミット日 2026-08-29 〜 09-01、両端を含む）。

前回が 62 版・1,774 ファイルだったのに対し、今回は 2 版・74 ファイルである。
**だが本章は「小さい差分」の章ではない。参照先そのものが差し替わった。**
`v2` ブランチは削除され、`main` が GA の正本になった。
10 章から 15 章までが名指ししている `v2` ブランチは、もう存在しない。

---

## 16.1 いちばん大きい変更はコードではなくブランチ

| | 変更前（2026-08-29 時点） | 変更後（2026-09-01 時点） |
|---|---|---|
| 既定ブランチ | `main`（1.x 系の内容） | **`main`（2.x 系の内容）** |
| 2.x の置き場所 | `v2` ブランチ | **`main`**（`v2` は**削除済み**） |
| 1.x の置き場所 | `main` | **`v1` ブランチ**（新設。HEAD `27b1e1d0`） |
| GitHub Release | Latest は **`v1.0.1`**（1.x） | Latest は **`v2.7.0`**（2.x 系で初のリリース公開） |

**旧 `v2` の先端から現在の `main` までは線形**である。`2fbee12f..origin/main` はコミット 4 件で、
rebase も squash も行われていない。**旧 `v2` の SHA は今も `main` から到達できる。**

**⚠ ただし `main` というブランチ自体は force-update されている。**
旧 `main`（1.x の内容）と現 `main`（2.x の内容）に共通の祖先関係は無い。
**旧 `main` を clone 済みの環境では通常の `git pull` は通らない。**
**既定は別ディレクトリへの clone し直しにすること。** 既存 checkout をそのまま作り直す
（`git fetch && git reset --hard origin/main`）と、**未コミットの変更と、旧先端からしか辿れない
ローカルコミットが失われる**。どうしても同じ checkout を使うなら、退避してから行うこと。
1.x を追い続けるなら `v1` ブランチへ切り替える。

### インストール手順が変わった

上流の README・`docs/guide/01-getting-started.md`・各ハーネス手引きが一斉に書き換わった。

```bash
# 旧（もう動かない。v2 ブランチが無い）
git clone https://github.com/awslabs/aidlc-workflows.git
cd aidlc-workflows
git checkout v2

# 新
git clone --branch main https://github.com/awslabs/aidlc-workflows.git
cd aidlc-workflows
```

> **⚠ 「もう動かない」の出方は 2 通りある。** 新規 clone での `--branch v2` や
> `git fetch origin v2` は失敗するが、**既にローカル `v2` ブランチを持つ使い回しの clone では
> `git checkout v2` が今も成功する**。リモートの削除はローカル ref を消さず、`checkout` は fetch もしない。
> 使い回しのワークスペースで動く CI は、**エラーを出さずに旧コミットのまま走り続ける**。
> `git ls-remote --heads origin v2` が空であることで確認できる。

Spec PDF の参照も `blob/v2/assets/...` から `blob/main/assets/...` に変わった。
ファイル自体は旧 `v2` HEAD にも現 `main` にも存在する（`main` は旧 `v2` の継続なので同じものである）。

> **⚠ Web の旧 URL は 302 で `main` に転送される（2026-09-01 実測）。**
> `blob/v2/...` も `tree/v2` も `Location: .../main/...` を返すので、**ブラウザでは今も開ける**。
> つまり **Git 操作は失敗するのに Web リンクは生きている**という食い違いがある。
> 「リンクが開けたから `v2` はまだある」と判断しないこと。

本ノートの該当箇所は
[README.md](./README.md)・[06-harnesses-install.md](./06-harnesses-install.md)・
[09-references.md](./09-references.md)・[SOURCES.md](./SOURCES.md) で更新した。

### 2.x が初めて GitHub Release として公開された

タグ自体は以前から打たれていた（`v2.1.1` 2026-06-26 / `v2.2.0` / `v2.3.0` 2026-07-09）。
しかし **GitHub Releases として公開された 2.x は `v2.7.0` が最初**である（公開 2026-09-01T03:04:51Z）。
タグ `v2.7.0` は `main` の HEAD `96b11d39` を指す。

これで **Latest リリースが `v1.0.1` から `v2.7.0` に移った**。

**ただし「pin できるようになった」わけではない。** タグは以前からあり、
`git clone --branch v2.3.0` のような固定はもともと可能だった。
変わったのは **Releases のページと Latest の表示**であり、
「リリース一覧から辿れる 2.x が 1 つも無い」状態が解消された、というのが正確なところである。

> **⚠ 上流の `docs/roadmap.md` はこれをまだ反映していない。**
> 更新後も「GitHub still marks `v1.0.1` as Latest, tracked by #635」「no public v2 native release exists yet」
> と書いてある。roadmap を更新した #991 のマージ（03:03:11Z）が、リリース公開（03:04:51Z）の
> **約 2 分前**だったためと見られる。

### ⚠ `v2_backup` は旧 v2 のバックアップではない

リモートには `v2_backup` というヘッドが残っている。名前から退避先に見えるが、実体は
`d898b74e`（subject: *feat: add AI-DLC evaluation framework + CI workflows (#359)*）で、
**削除された `v2` の HEAD のコピーではない**。旧 v2 の内容を取りに行く先として当てにできない。
必要なのは `main`（またはタグ `v2.7.0`）である。

### `v1` ブランチは旧レイアウトのまま

`v1` には `core/tools/aidlc-version.ts` が無い。バージョン表記は `aidlc-rules/VERSION` = `1.0.1` で、
`aidlc-rules/` + `scripts/` を中心とするルール配布型の構造である
（[08-v1-vs-v2.md](./08-v1-vs-v2.md) が扱っている 1.x の姿）。
**2.x の本ノートの記述を `v1` ブランチで照合しないこと。**

---

## 16.2 数値の変化 — 全項目不変

| 項目 | 2.6.123 | 2.7.0 | 測定方法 |
|---|---:|---:|---|
| フェーズ | 5 | 5 | `core/aidlc-common/stages/` の第 1 階層 |
| ステージ | 33 | 33 | `core/aidlc-common/stages/` の `.md` 数 |
| スコープ | 11 | 11 | `core/scopes/` |
| エージェント | 14 | 14 | `core/agents/` |
| センサー | 6 | 6 | `core/sensors/*.md`（一覧はバイト単位で同一） |
| TypeScript フック | 18 | 18 | `core/hooks/` |
| `core/tools/*.ts` | 51 | 51 | 直下のみ。`import.meta.main` を持つのは 32 本 |
| 監査イベント種別 | 91 | 91 | `core/tools/aidlc-audit.ts` の `VALID_EVENT_TYPES` |
| ハーネス | 7 | 7 | `dist/` 直下 8 ディレクトリのうち `dist/plugins/` はサンプルプラグイン |

スコープ別ステージ数も不変である
（feature 33 / enterprise 33 / workshop 26 / classic 26 / mvp 23 / infra 13 /
security-patch 10 / refactor 10 / express 10 / bugfix 9 / poc 8）。

### `core/` の変更は 4 ファイル 4 行だけ

`git diff --stat 2fbee12f..origin/main -- core/` の結果:

> **⚠ `--depth 1` の shallow clone では再現できない。** `2fbee12f` が手元に無いためである。
> 再現するには `git fetch --unshallow`（または `git fetch origin 2fbee12f`）が要る。


| ファイル | 変更 |
|---|---|
| `core/knowledge/aidlc-shared/state-template.md` | 1 行 |
| `core/tools/aidlc-state.ts` | 1 行 |
| `core/tools/aidlc-utility.ts` | 1 行 |
| `core/tools/aidlc-version.ts` | 1 行（バージョン定数） |

`dist/` 側は 7 ハーネス × 同じ 4 ファイル = 28 ファイル、各 1 行。
残る 42 ファイルは文書とリポジトリ運用ファイルである。

---

## 16.3 2.6.124 — 状態ファイルから絶対パスが消えた

区間内で**実装が動いたのはこの 1 版だけ**である（#962 / issue #937）。

| 場所 | 旧 | 新 |
|---|---|---|
| `aidlc-utility.ts` の `handleIntentCreateStateBuild` | `- **Project Root**: ${projectDir}` | `- **Project Root**: .` |
| `aidlc-state.ts` の `handleFork` | `Worktree Path` に絶対パス | `relative(pd, wtPath).replaceAll("\\", "/")` |

`state-template.md` の説明も「プロジェクト相対、通常 `.`。実行時に再導出されるので絶対パスとして信頼しない」に変わった。
回帰テストは `t70` / `t78` / `t152`（Windows portability）。

**移行作業は無い。** CHANGELOG が「移行処理は行われず、必要もない」と明記している。理由は次のとおり:

- `Project Root` は実行時に `projectRootFor` が実 `aidlc/` ディレクトリまで遡って再導出する。
  コミットされた値は**到達しないフォールバック**である
- `Worktree Path` は**表示用のパンくず**であり、解決に使われない

したがって既存の `aidlc/**/aidlc-state.md` に残る絶対パスは、
**次にエンジンがそのフィールドを書くまで無害（inert）**である。
`dist/<harness>/` の再コピーはツールを更新するだけで、既存の状態ファイルは書き換えない。

### 本ノートの読者にとっての意味

これは機能追加ではなく**移植性と情報保護の修正**である。
`aidlc-state.md` は通常 Git 管理下に置かれるので、従来は
`/Users/<個人名>/...` のような**ホームパスがコミット履歴に残っていた**。
複数マシン間や共有リポジトリで状態を共有する構成では、この 1 版だけでも上げる価値がある。

[07-learning-loop-state.md](./07-learning-loop-state.md) の `aidlc-state.md` の記述に
この 2 フィールドを追記した。

---

## 16.4 2.7.0 の CHANGELOG はロールアップであって新機能一覧ではない

**本章でいちばん誤読しやすいのがここである。**

2.7.0 のエントリは 9 個の箇条書きで「33 ステージ」「Application Design が Domain Design に改称、
Contract Design を追加」「Classic と Express が第一級スコープ」「`reviewer:` を宣言するステージは
`review_artifact:` が必須」といった**大きな話を並べている**。
しかし同じエントリの冒頭にこう書いてある。

> AI-DLC 2.7.0 consolidates the 2.6.x release cycle into a new minor baseline
> **without changing runtime behavior from 2.6.124**.

実際、2.7.0 のコミット `a2adefd1` が触ったのは以下だけである。

| ファイル | 変更 |
|---|---|
| `core/tools/aidlc-version.ts` + `dist/` 7 本 | `AIDLC_VERSION` を `2.6.124` → `2.7.0` |
| `README.md` | バージョンバッジ |
| `CHANGELOG.md` | `## [2.7.0]` の追記 |

**ロジックの変更は 1 行も無い。** 変わった `.ts` は `AIDLC_VERSION` の 1 行だけである。

### 「33 ステージ」はいつ入ったのか

CHANGELOG の書き方だと 2.7.0 で入ったように読めるが、そうではない。

- `core/aidlc-common/stages/` のファイル一覧は 2.6.123 と `main` で**バイト単位で同一**
- `inception/domain-design.md` と `inception/contract-design.md` を追加し
  `inception/application-design.md` を削除したのは `74a51a10`（#711）＝ **2.6.1**

つまり 9 個の箇条書きは **2.6.1 〜 2.6.124 の 2.6.x サイクル全体のロールアップ再掲**である
（公開 CHANGELOG 上の 2.6.x エントリは **95 件**。欠番があるので版番号の差とは一致しない）。
本ノートでいえば 12 章から 15 章までで既に扱った内容にあたる。

### では 2.7.0 のエントリは何のためにあるのか

**2.6.x を飛ばして上げる人向けの移行チェックリスト**である。エントリ自身がこう釘を刺している。

> Upgrades from any release before 2.6.124 must also apply every intervening
> **Upgrade**, **Breaking**, and migration note below;
> **this roll-up does not replace those one-time actions.**

**ロールアップを読んだから個別版の Upgrade 文を読まなくてよい、ということにはならない。**
15 章 15.3 で扱った 2.6.121 の `review_artifact:` 追加のような一度きりの移行は、
2.7.0 のエントリを読んだだけでは実行されない。
[06-harnesses-install.md](./06-harnesses-install.md) 6.4 の注記はこの前提のまま有効である。

### ⚠ 上げたあとに `/aidlc plugin sync` が要る

2.7.0 の Upgrade 文は、再コピーで終わりだとは言っていない。

> **Upgrade:** replace the complete `dist/<harness>/` tree in one quiescent operation,
> then run `/aidlc plugin sync` for every installed plugin.

**プラグインを入れている場合、`dist/` を置き換えただけでは合成が失われる。**
エンジンを入れ替えるとコンパイル済みのグラフが素の状態に戻るためで、
これは 2.6.110 で顕在化した挙動である（→ [15.7](./15-release-impact-26123.md#157-プラグイン--作成ツールチェーンと再インストールで消える合成)）。
**2.7.0 に固有の話ではないが、上げるたびに必要である。**

エントリはもう 1 つ前提条件を書いている。CHANGELOG の原文は次のとおり。

> Before replacing an older shell, finish and archive every workflow created before 2.6.1;
> the new shell rejects that stale state before `--new-intent` can create fresh work.

つまり **2.6.1 より前に作成したワークフローが残っていると、新しいシェルはそれを拒否する**。

**⚠ これは 2.7.0 が新設した制約ではない。** 2.7.0 が変えたのはバージョン定数だけなのだから、そうではありえない。
拒否しているのは **2.6.1 の永続 state スキーマ v8**（→ [12-release-impact-2602.md](./12-release-impact-2602.md)）であり、
2.7.0 のエントリはそれを**移行チェックリストとして再掲している**にすぎない。
2.6.1 の CHANGELOG 日付は **2026-08-13** なので、それ以前に着手したワークフローが対象になる。
**本ノートはこの拒否を実機で確認していない**（CHANGELOG の記述のみ）。

---

## 16.5 roadmap の更新

`docs/roadmap.md` が `Status as of 2026-08-20` から `2026-09-01` に更新された。
判定（Shipped / Partial）が変わったゴールは無いが、内訳が動いた。

| ゴール | 変化 |
|---|---|
| 5 Cyclic flows | #616（Build & Test → Code Generation の有界ループバック）が `implements` から **`shipped`** へ。判定は **Partial のまま**。残るのは「一般の cross-stage サイクル」 |
| 6 Traceability | #716（stale 結果の伝播）と #813（Unit 単位帰属）が Remaining から **Delivered** へ移動。判定は **Partial のまま**。残るのは cross-unit discovery propagation（#299 / #300） |
| 2 Customization | #797（プラグインの doctor 拡張）と #892（単体の作成ツールチェーン）を Delivered に追加 |
| 7 Org repository | #894（DocumentKB の要約とタグ）を Delivered に追加 |

### Known gaps の書き換えが 1 件、本ノートに効く

| 旧 | 新 |
|---|---|
| センサー障害は advisory。blocking 深刻度は #431 で未解決 | **write 起動のセンサーは advisory のまま。ゲート束縛センサーは blocking 深刻度と人手 override に対応** |

これは 15 章 15.5 で実測した内容（2.6.72 で実装済み、ただし出荷 6 センサーは全て `advisory`）と
矛盾しない。上流の roadmap が実装に追いついたということである。

### 進行中 PR の入れ替え

| 追加 | 内容 |
|---|---|
| #969 | Pull Request 経由の Construction 統合 |
| #968 | **Devin CLI / Desktop ハーネス** |
| #907 | mabl 検証プラグイン |
| #753 | Evaluator 統合 |

除去されたのは #797 / #813 / #716 / #616 / #646 / #754 / #788 の 7 件。
このうち #797 / #813 / #716 / #616 / #646 / #788 は出荷済みとして別の節に移った。
**#754（Bolt 用語の整合）だけは移動先が無く、更新後の roadmap のどこにも現れない。**
取り下げか、記載漏れかは判断できない（15 章 15.9 で扱った 2.6.86 の Bolt 定義変更で
実体が済んだ可能性はあるが、上流はそう書いていない）。
#968 が入ると**ハーネスが 8 種目になる**が、本章時点では PR 段階であり `dist/` に実体は無い。

### ⚠ 上流 roadmap の記載が 1 版古い

更新後の `docs/roadmap.md` は次のままである。

> The current v2 version is **2.6.124** (`origin/main` tip `82d2e304`).

`main` の HEAD は `96b11d39`、タグ済みの版は 2.7.0 なので、**この行は上流内部で不整合**である。
#991（roadmap 更新）が #992（2.7.0 リリース）の直後にマージされ、追随しなかったものと見られる。
**版の判定は `CHANGELOG.md` の `## [x.y.z]` 見出しのみで行う** という 15 章 15.11 の結論をここでも適用する。

---

## 16.6 リポジトリ運用ファイルの追加

`.github/workflows/` が **4 本から 8 本**になった。すべて `96b11d39` で新規追加されたものである。

| ワークフロー | 実体 |
|---|---|
| `release.yml` | **v1 用ワークフローへの dispatch シム** |
| `release-pr.yml` | 同上 |
| `codebuild.yml` | 同上 |
| `pull-request-lint.yml` | 実体あり。`pull_request_target`（`main` と `v1`）+ `merge_group` |

前 3 者がシムなのは、GitHub が `workflow_dispatch` を**既定ブランチからしか登録しない**ためである。
v1 のリリース処理は `v1` ブランチにあるが、それを起動するには `main` 側に同じパスのスタブが要る。
`release.yml` は tag 入力を `^v[0-9]+\.[0-9]+\.[0-9]+$` で検証し、
その tag に v1 側の実装があることを `gh api repos/$REPO/contents/...?ref=$TAG` で確かめてから
`gh workflow run release.yml --ref "$INPUT_TAG"` を呼ぶ。**子ワークフローの結果は報告しない。**

あわせて `.github/ISSUE_TEMPLATE/`（bug_report / documentation / feature_request / rfc + config）、
`.github/labeler.yml`、`.github/pull_request_template.md` が追加された。
`labeler.yml` は `v1` にも存在するので引き継ぎだが、
**`ISSUE_TEMPLATE/` と `pull_request_template.md` は `v1` に無く、この移行で新設されたものである。**

**いずれも AI-DLC の実行時挙動には関係しない。** 上流にコントリビュートする場合にのみ関わる。

---

## 16.7 全 2 版の一覧

| 版 | 要旨 | 本章での扱い |
|---|---|---|
| 2.7.0 | 2.6.x のロールアップ。ランタイム挙動の変更なし | [16.4](#164-270-の-changelog-はロールアップであって新機能一覧ではない) |
| 2.6.124 | `aidlc-state.md` の `Project Root` / `Worktree Path` をプロジェクト相対に | [16.3](#163-26124--状態ファイルから絶対パスが消えた) |

CHANGELOG に現れないコミットが 2 件ある。

| コミット | 内容 |
|---|---|
| `518578ad`（#972） | Mermaid の `style` 指定 170 箇所に `color:#000` を追加。ダークモードでノードラベルが読めない問題の修正。**170 挿入 / 170 削除、他の変更なし** |
| `96b11d39`（#991） | 本章 16.1・16.5・16.6 のブランチ再編 |

---

## 16.8 過去章への影響 — 訂正ではなく注記

今回、**過去章の事実誤りは見つからなかった**。15 章までの測定値は 2.7.0 でもすべて有効である。

ただしブランチ再編によって、**10 章から 15 章が名指ししている `v2` ブランチが存在しなくなった**。
10 章は `対象:` 行に「v2 ブランチ」と書くのみで HEAD SHA を持たない。11〜15 章は `基準:` 行に HEAD SHA がある。

| 章 | `基準:` 行の HEAD | 現状 |
|---|---|---|
| 11 章 | `2ce654d1` | `main` から到達可 |
| 12 章 | `4569754e` | 同上 |
| 13 章 | `71d9a9e0` | 同上 |
| 14 章 | `840ba653` | 同上 |
| 15 章 | `2fbee12f` | 同上 |

5 件とも `git merge-base --is-ancestor <SHA> origin/main` で祖先であることを確認済みである。

**過去章は日付付きの測定記録なので本文は書き換えない。**
記録された時点でブランチ名が `v2` だったことは事実だからである。
再現するときは `git clone --branch main` した上で SHA を直接指定すればよい。

一方、**生きた手順・生きた参照先は書き換えた**。両者の線引きは次のとおりである。

| 区分 | 扱い | 対象 |
|---|---|---|
| 日付付きの測定記録 | **書き換えない** | 10 章の `対象:` 行と 11〜15 章の `基準:` 行、`CONVERSATION_LOG.md` と `REVIEW-8AI.md` の過去追記 |
| 実行される手順 | **書き換えた** | README・6 章・9 章・SOURCES の clone コマンド |
| 現在の参照先の宣言 | **書き換えた** | README 冒頭、SOURCES の正本宣言、`docs/TRACEABILITY.md`、`docs/DEVELOPMENT_RULES.md` |

---

## 16.9 本ノートの限界

- 本章は**上流リポジトリの差分の静的読解**に基づく。実機での再現は行っていない。
- 上流のテストスイートは実行していない（調査環境に `bun` が無い）。
- 2.6.124 の「既存の絶対パスは次の書き込みまで無害」は**コードの読解による判断**であり、
  実際に古い状態ファイルを持つプロジェクトで確認したものではない。
- 2.7.0 の「ランタイム挙動の変更なし」は CHANGELOG の記述と `git diff` の一致で確認したが、
  **挙動そのものを実行して確かめたわけではない**。
- `release.yml` 等の dispatch シムは YAML の読解のみで、実際に起動していない。
- 「2.7.0 調査で残った未確認事項」は
  [docs/REMAINING_TASKS.md](./docs/REMAINING_TASKS.md) に列挙した。
