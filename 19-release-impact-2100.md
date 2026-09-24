# 19. リリース差分 2.9.0 → 2.10.0 — Guard Policy と 3 件の不具合の解消

作成日: 2026-09-24
基準: **タグ `v2.9.0`（`22f5d1b1`）→ タグ `v2.10.0`（`2a883858`）**（取得 2026-09-24）
前回: [18-release-impact-290.md](./18-release-impact-290.md)（2.8.1 → 2.9.0、終点 `2931ef02`）

コミット **67 件**、変更ファイル **620 件**（+117,900 −12,997 行）、期間 **10 日**（上流コミット日 2026-09-15 〜 09-24）。
`v2.10.0` の公開は **2026-09-24 05:52 UTC**。取得時点の `main` は `20008a5d`（タグから +1、テストのみ）。

**要点は 3 つある。**

1. **18.6 で警告した 3 件の不具合（#1070 / #1166 / #1249）は、すべて `v2.10.0` で安定版に入った**（→ 19.4）
2. **Change Control が Guard Policy に改名され、既定スコープ `classic` を含む 8 スコープで「承認後の変更」の締め付けが緩んだ**（→ 19.3）。
   **初回の Plan Approval は引き続き必須**である
3. **リリース時にフルテストスイートを要求しなくなった** —— 18.15 で「テストが発行の前提条件に戻った」と書いた状態は、本区間で再び覆った（→ 19.10）

---

## 19.1 まず読む — 本章はタグからタグで測っている

18 章は**リリース前の `main` の 1 点（`2931ef02`）を終点**にしていた。
**`2931ef02` は `v2.9.0` の 12 コミット後**であり、`v2.9.0` タグそのものではない。
18.1 の教訓（基準 SHA とリリースタグの取り違え）を繰り返さないため、**本章は両端ともタグで測った。**

| | コミット | 備考 |
|---|---|---|
| 始点 = タグ `v2.9.0` | `22f5d1b1` | 2026-09-15 |
| 18 章の終点 | `2931ef02` | `v2.9.0` + 12。**本章の区間に含まれる** |
| 終点 = タグ `v2.10.0` | `2a883858` | 2026-09-24 |

したがって**本章の 67 コミットには、18 章の終点までの 12 コミットも入っている。**
18 章の数値と本章の数値を足し合わせてはいけない。

区間内の preview タグは 3 本（`v2.9.1-preview.20260915.1` / `.20260920.1` / `.20260921.1`）。
本章の各項目には、`git tag --contains <sha> --sort=creatordate` で求めた**初出タグ**を付けた。

---

## 19.2 アップグレードの手順

CHANGELOG 2.10.0 の逐語:

> **Upgrade:** run `aidlc update`, then run `aidlc config --yes` in each project to refresh its harness runtime.
> Manual-copy users must replace the complete `runtime/<harness>/` tree from `aidlc-copy-runtime-2.10.0.tar.gz`.

- **ネイティブ導入**: `aidlc update` の後、**プロジェクトごとに `aidlc config --yes`**（2.9.0 と同じ形）
- **手動コピー**: `aidlc-copy-runtime-2.10.0.tar.gz`。資産は 15 件で構成は `v2.9.0` と同型
- 当ノートの手動コピー手順（[6 章](./06-harnesses-install.md)）は **`v2.10.0` の実物で流し直した**。
  あわせて、**別のハーネスが同じ管理ディレクトリを使っているときに止める検査**を足した（→ 19.6）

**⚠ Bedrock を出荷既定のまま使っているプロジェクトは、`aidlc config --yes` の前に 19.7 を読むこと。**

---

## 19.3 🔴 Change Control → Guard Policy — 何が緩み、何が残ったか

**初出リリース `v2.10.0`**（`4c85c8d5` / #1262。preview には入っていない）。

### 名前と値

| | `v2.9.0` | `v2.10.0` |
|---|---|---|
| スコープ frontmatter | `change_control:` | **`guard_policy:`** |
| 値 | `strict` / `relaxed` | `strict` / `relaxed` / **`off`** |
| 切替コマンド | `/aidlc --change-control …` | **`/aidlc --guard-policy …`** |
| メモリの見出し | `## Change Control` | **`## Guard Policy`**（旧見出しも読まれる） |
| 監査行 | `CHANGE_CONTROL_SET` | **`GUARD_POLICY_SET`**（旧名は読み込み専用で残る） |

**スコープ別の既定値は 11 本とも変わっていない**（`strict` は `enterprise` / `infra` / `security-patch` の 3 本、残り 8 本は `relaxed`）。
**既定で `off` のスコープは無い。**

### 柵（fence）の仕組み

上流は PreToolUse フックの拒否を「柵」と呼び直し、判定を `aidlc-lib.ts` の 1 か所に集めた。
**柵は 5 つ、そのうち人間が作業単位で切り替えられるのは 4 つ**である。

| 柵 | 止めるもの | `relaxed` で下がる | `off` で下がる |
|---|---|:---:|:---:|
| `plan-approval` | 承認済みの計画に基づかないコード生成 | ✔ | ✔ |
| `review-freeze` | 終端レビュー受領後の成果物の書き換え | ✔ | ✔ |
| `state-transition` | ライフサイクルの直接呼び出し | | ✔ |
| `reviewer-scope` | レビュアの隣接ユニットへの読み取り | | ✔ |
| `human-presence` | 新しい人間のターンを要する操作 | **切替不可** | **切替不可** |

表の「下がる」はポリシー語による。**これとは別に、`state-transition` 以外の 4 つには環境変数の kill switch がある**
（`human-presence` は `AIDLC_SKIP_HUMAN_PRESENCE_GUARD=1` だけで外れ、作業単位の切替は無い。エージェントのツール呼び出しからの代入は後述の `runtime-integrity.ts` が拒否する）。

**⚠ `plan-approval` の ✔ は「初回の Plan Approval が不要になる」という意味ではない**（→ 下の「✅」）。下がって効くのは、承認後の内容変更に対する再承認の拘束である。

下がった柵は**拒否せずに通し、1 行の通知と `GUARD_STOOD_ASIDE` 監査行を残す**（上流の用語で "stand aside"）。
**柵が下がっているかどうかだけで決まり、誰の指示で動いているか（人間の発言、エンジンの指示、どちらでもない）は判定に使われない**
（`decideGuard` の表。人間の「いいから書いて」という返答では柵は下がらない）。

### ✅ 初回の Plan Approval は `relaxed` / `off` でも必須である

**「既定スコープで Plan Approval が外れた」は誤りである。** plan-approval フックの stand-aside 分岐は、
**承認済みで実行可能な計画を持たない対象を引き続き拒否する**（`codeGenerationExecutionAllowed`。コードのコメント逐語
"It does not supply missing approval, artifacts, target or attempt authority."）。参照文書の逐語:

> Initial Plan Approval, including the autonomous-mode plan stop, and other human gates remain required.

### 🔴 実際に緩んだ 2 点 —— どちらも `classic` の既定で効く

`relaxed` は `v2.9.0` にもあった。**同じ `relaxed` という値の意味が広がった**のが本区間の変更である。

| | `v2.9.0` の `relaxed` | `v2.10.0` の `relaxed` |
|---|---|---|
| 承認後にワークスペースのソースが動いた | 1 回だけ受け入れて `CHANGE_ACCEPTED` | 同じ |
| **承認後に計画・unit test instructions・Testing Contract を編集した** | **承認をやり直す**（逐語 "reopen approval under both values"） | **再承認なしで続行できる** |
| **終端レビュー受領後に成果物を書き換えようとした** | **拒否**（逐語 "The freeze itself is never relaxed"） | **通して `GUARD_STOOD_ASIDE`** |

`v2.10.0` の参照文書の逐語:

> A fence lowered by `relaxed`, `off`, or `config set guard.plan-approval off` permits continuation
> with the current content through the engine without mandatory reapproval.

つまり **`classic` で動かすと、承認した計画の中身を後から変えても再承認を求められず、
レビューが終わった成果物を書き換えても止められない。** 監査行は残るが、**決定的な拒否ではなくなった。**
上流自身の Security posture 節もこれを認めている（逐語）:

> The conductor still asks every approval question; **a lowered fence does not enforce that prose obligation.**
> It lets undirected work through, with a `GUARD_STOOD_ASIDE` row

**⚠ "undirected work" という語は上流で定義されていない。** `v2.10.0` の `docs/` 全体で 3 か所に出てくるだけである。
実質的な定義に近いのは参照文書の次の一文で、柵の判定は「指示が求めた範囲の外」と判定された行為にだけ働く（逐語）:

> A fence only reaches this function once its own predicate has already found the action outside what the instruction asked for

### 締まった点

緩んだだけではない。**柵を下げる権限が人間に絞られた。**

- **ツール呼び出しから `relaxed` / `off` を設定することや、`guard.<fence> off` は拒否される**（`aidlc-guard-switch.ts`）。
  人間が `/aidlc --guard-policy relaxed` と**自分でタイプ**したときだけ、human-turn フックが適用する
- 新しいフック `runtime-integrity.ts` が、`AIDLC_UNATTENDED=` / `AIDLC_SKIP_HUMAN_PRESENCE_GUARD=` の代入と、
  セッション記録の編集を拒否する
- チャットからの `intent create --guard-policy relaxed|off` は、選んだスコープの既定と異なる値なら拒否される（作成後に人間が切替をタイプする）

### 組織として旧来の締め付けに戻す方法

**メモリ層（`org.md` / `team.md` / `project.md`）の `## Guard Policy` に `Mode: strict` を書く。**
逐語: "Strict here holds for every intent and cannot be changed from chat."
メモリが strict を保持している間は、作業単位の `Guards Off` も無効になり、拒否文は切替コマンドの代わりにメモリファイルの場所を示す。

**⚠ 実機では試していない**（コードと参照文書の読解による）。

### 既存レコード

- `Guard Policy` 行（旧 `Change Control` 行）が無い既存の `aidlc-state.md` は、**引き続き `strict` として読まれる**
  （18.3.1 と同じ。`aidlc-lib.ts`）。**`doctor` はこれを検知しない**
- 旧名（`change_control` キー・`Change Control` 行・`## Change Control` 見出し）は**このリリースでは読まれ、次の minor で削除**と上流は予告している（`aidlc-lib.ts` の文言。`## Review` と同じく予告が守られるとは限らない → 19.11）

### Kiro IDE では stand-aside が見えない

`GUARD_STOOD_ASIDE` の 1 行は、Claude Code では `systemMessage` として人間に出る。
**Kiro IDE 1.x はフックの標準出力を転送しないため、通知は出ず、監査行だけが記録になる**
（上流の実測。監査台帳がまだ無い intent では行も残らない）。

---

## 19.4 ✅ 18.6 の 3 件の不具合はすべて安定版に入った

| 不具合 | 修正コミット | 初出タグ | `v2.10.0` |
|---|---|---|:---:|
| ① ゲートの Review brief が動かない（#1070） | `be94bde7` | `v2.9.1-preview.20260920.1` | ✅ |
| ② ゲートのセンサーが発火しない（#1166） | `c97fa7ba` | `v2.9.1-preview.20260920.1` | ✅ |
| ③ コアフック 3 本が `bun` を直接名指し（#1249） | `f79e321b` | **`v2.10.0`** | ✅ |
| 参考: `worktree info` が監査行を検証せずに返す（#1281） | `4173be2c` | `v2.9.1-preview.20260921.1` | ✅ |

**18.6 の「センサーを使うなら手動コピー経路で、`bun` を PATH に置く」という回避策は、`v2.10.0` に上げれば不要になる。**
版の判定は引き続き `git merge-base --is-ancestor f79e321b <tag>` で足りる（3 件のうち最も新しい修正）。

**⚠ 実機での再現・解消確認はしていない**（版間の到達判定とコード読解による）。

### 🔴 追記（2026-09-25）: `v2.10.0` に残る既知の不具合 —— ネイティブ導入でチーム担当の Construction が始まらない

**上の「3 件は解消」は 18.6 の不具合についての話である。** `v2.10.0` 公開後、次の不具合の修正が `main` に入った。
**修正 `057b13be`（#1309、issue #1286）はどのリリースにも未収録**（2026-09-25 時点）。

| 項目 | 内容 |
|---|---|
| 条件 | **ネイティブ（コンパイル済み）`aidlc`** で、Delivery Planning が `Construction Iteration: unit-major` と **`Unit Ownership: team`** を記録した intent |
| 症状 ① | エンジン自身が呼ぶ状態操作 `refresh-unit-progress` / `sync-unit-scope-stage` / `fold-unit-merge` が、ディスパッチャの許可リストに無いため拒否され、**チーム担当の Construction が開始できない**（`v2.10.0` の `core/tools/aidlc.ts` にこの 3 語は 1 つも無い） |
| 症状 ② | `aidlc unit land` の状態反映が、バンドル内のスクリプトパスをコマンドとして渡してしまい `unknown command '/$bunfs/root/aidlc-state.ts'` で失敗する（`v2.10.0` の `aidlc-unit.ts` の `runStateFold`） |
| 影響しない場合 | 単独（solo）運用。**手動コピー（Bun）経路**（Bun 実行ではディスパッチャを通らないため。上流のテストで見つからなかった理由もこれ、とコミット本文にある） |

**チーム担当の Construction を使う予定があるなら、当面は手動コピー（Bun）経路にするか、修正を含む版を待つ。**
判定は `git merge-base --is-ancestor 057b13be <tag>`（**`f79e321b` での判定ではこの修正を取りこぼす**）。

**⚠ 実機では再現させていない**（コミット本文、issue #1286、`v2.10.0` のコード読解による）。

---

## 19.5 Construction の既定の進め方が変わった（新規 intent のみ）

**初出 `v2.9.1-preview.20260920.1`**（`261083ce` / #1150）。

ユニット分解があり、ソースを作るユニット単位のステージを含む**新規の単独ワークフロー**は、既定で次のようになる:

- `Construction Iteration: unit-major`（1 ユニットの設計〜コード生成を終えてから次へ）
- `Construction Execution: serial`
- `Construction Checkpoints: enabled` —— ユニット／バッチの区切りで検証を回す
- 検証には、**人間が承認した `Construction Verification Command`** を使う（Delivery Planning が提案する）

**既存の intent（チェックポイント設定を持たないもの）は従来どおり**の段階的ゲートで進む。
新規 intent でも明示的な stage-major の選択は尊重される。監査イベントが 3 種増えた（→ 19.9）。

---

## 19.6 複数ハーネスの共存 —— ただし組み合わせに制限がある

**初出 `v2.9.1-preview.20260920.1`**（`c66c4222` / #1199）、AGENTS.md の中立化は `.20260921.1`（`13a1a859` / #1268）。

- 1 つのプロジェクトに**衝突しない**ハーネスを複数置けるようになった。既存プロジェクトでは、2 つ以上のハーネスがあるときに `--harness` の指定が必須になる（新規導入では、`--from`・設定済みの既定・単一の候補からハーネスが決まらないまま非対話で走らせると必須。自動化では明示を勧める）
- **Kiro CLI と Kiro IDE（どちらも `.kiro/`）、OpenCode と GitHub Copilot（どちらも `.aidlc/`）は共存できない**
- ルートの `AGENTS.md` の管理ブロックは**ハーネス中立の共有文面**になった。codex / cursor / kiro / kiro-ide / opencode で同一
- **copilot の `AGENTS.md` ブロックは専用のままで、`AGENTS.md` を持つ他のハーネス（codex / cursor / kiro / kiro-ide / opencode）とは共存できない**（上流ガイド逐語 "Copilot's `AGENTS.md` stays exclusive"）。**copilot と共存できるのは claude だけ**である
- claude はどのハーネスとも共存できる
- 共有の `AGENTS.md` ブロックを**別リリースの兄弟ハーネス**が持っている状態は競合として拒否される。全ハーネスを同じリリースで更新すること
- cursor の手動コピー用インストーラ（ルート `install.ts`）は単一ハーネス専用。複数ハーネスのプロジェクトには `aidlc config --harness cursor` で加える
- `opencode.json` の `instructions` に `.aidlc/onboarding.md` が加わった
- `.gitignore` の管理ブロックは導入済みハーネスの和集合になる

**手動コピーでの注意（6 章に反映済み）:**

- **共存できない組み合わせを上書きすると、先に入っていたハーネスが壊れる。**
  6 章の手順に、管理ディレクトリの `tools/data/harness.json` の `distribution` を見て止める検査を足した。
  あわせて copilot と `AGENTS.md` を共有するハーネスの組み合わせも止める。
  `v2.10.0` の実物で「copilot の後に opencode → 中止・copilot は無傷」「codex と copilot → 順序によらず中止」「claude と codex／claude と copilot → 成功」を確認した
- **copilot の管理ディレクトリを `.github/` 丸ごとにしていた当ノートの誤りを直した**（→ 19.14）
- **OpenCode では `AGENTS.md` と `opencode.json` を一緒に取り込むこと。** 片方だけ新しくすると、
  `opencode.json` が参照する `.aidlc/onboarding.md` と `AGENTS.md` の文面が噛み合わない
- **cursor のルート `install.ts` は 6 章の手順の対象外**である（本区間以前からの既知の欠落）

---

## 19.7 ⚠ Bedrock の出荷既定が外れた —— `aidlc config --yes` で消えうる

**初出 `v2.9.1-preview.20260921.1`**（`c8ad4116` / #1101）。CHANGELOG の逐語:
"configuration keeps the current provider unless Amazon Bedrock is explicitly selected."

- Claude の `settings.json` の `env` に出荷されるのは **`AWS_AIDLC_DEFAULT_SCOPE` だけ**になった
- `aidlc-init.ts` は、**プロバイダの回答が Bedrock として記録されておらず、`env` が旧来の出荷既定と一致する**場合に
  `CLAUDE_CODE_USE_BEDROCK` / `AWS_REGION` を**削除**し、旧既定のモデル別名も外す

**したがって、出荷既定のまま Bedrock で動いていたプロジェクトは、`aidlc config --yes` で Bedrock 設定が外れうる。**
Bedrock を使い続けるなら、`aidlc config` で Bedrock を明示的に選ぶか、実行後に `settings.json` を確認すること。
**⚠ 当方は実行して確かめていない**（コード読解）。

**モデル階層（tiers）について:** Claude の `balanced` は**引き続き `sonnet` / `medium`**。
**モデル指定**が `null`（ハーネス任せ）になったのは codex / opencode の `balanced` である（effort / variant の `medium` は残る）。

---

## 19.8 Bolt の識別子が intent 単位になった

**初出 `v2.9.1-preview.20260921.1`**（`2009d8df` / #1261）。

同名ユニットを持つ 2 つの intent が、worktree のパス・`bolt-<slug>` ブランチ・後片付けで衝突していた。修正後:

```
.aidlc/worktrees/bolt-<id8>_<slug>        （旧 bolt-<slug>）
refs/aidlc/reviewed-source/<id8>/<slug>/<commit>
refs/aidlc/parked/<id8>/<slug>/...
```

旧形式のディレクトリは、メタデータの intent が一致するときだけ引き継がれる。
**レジストリに uuid を持たない intent は、ここで止まる**（コミット本文逐語 "an intent without a registry uuid fails closed"）。
Bolt worktree のパスを前提にしたスクリプトがあれば見直しが要る。

---

## 19.9 数値の変化

| 指標 | `v2.9.0` | `v2.10.0` | |
|---|---:|---:|---|
| TypeScript フック | 18 | **19** | `runtime-integrity.ts` |
| `core/tools/*.ts` | 71 | **76** | CLI は 40 のまま |
| 監査イベント | 99 | **105** | 25 分類のまま |
| `AIDLC_*` 識別子※ | 135 | **143** | |
| ステージ / スコープ / エージェント | 33 / 11 / 14 | 33 / 11 / 14 | 不変 |
| センサー / プロトコル / ハーネス / バイパス | 6 / 9 / 7 / 12 | 6 / 9 / 7 / 12 | 不変 |

- 追加ツール 5 本: `aidlc-construction-checkpoints` / `aidlc-swarm-checkpoints` / `aidlc-guard-fences` / `aidlc-guard-switch` / `aidlc-guard-operation`
- 追加監査イベント 6 種:
  - `CHECKPOINT_VERIFICATION_RECORDED` / `CONSTRUCTION_POLICY_RECORDED` / `VERIFICATION_COMMAND_RECORDED`（#1150、初出 `.20260920.1`）
  - `GUARD_POLICY_SET` / `GUARD_RESTORED` / `GUARD_STOOD_ASIDE`（#1262、初出 `v2.10.0`）
- スコープ別のステージ所属数・ステージ frontmatter も不変
- ※ `core/` と `harness/` に現れる `AIDLC_[A-Z0-9_]+` の一意語の数（`git grep -hoE 'AIDLC_[A-Z0-9_]+' <tag> -- core harness | sort -u | wc -l`）。**環境変数だけの数ではない** —— 定数 `AIDLC_VERSION`、接頭辞の断片 `AIDLC_DISABLE_` 等、ログのラベル、テスト用の名前も含む。`process.env.AIDLC_*` の参照に絞ると 96 → 101
- 18 章の「環境変数 136」は同じ数え方での終点 `2931ef02` の値で、**タグ `v2.9.0` では 135**（数え方の違いではない）

**⚠ 評価上の注意:** 監査イベントの増加 6 種のうち 3 種は **Guard Policy の記録**である。
`GUARD_STOOD_ASIDE` が増えることは「統制が強くなった」ことを意味しない —— **拒否が記録に置き換わった**ことの痕跡である（→ 19.3）。

---

## 19.10 🔴 リリース時のフルテストスイートが再び外れた

18.15 で「`native-smoke` の `needs` に `test` が加わり、テストが発行の前提条件に戻った」と書いた。**本区間でこれが覆った。**

| `.github/workflows/release.yml`（両タグで実測） | タグ `v2.9.0` | タグ `v2.10.0` |
|---|---|---|
| テスト系ジョブ（smoke / unit 4 シャード / deep / 集約） | 4 本 | **0 本** |
| `native-smoke` の `needs` | `[validate, verify, test]` | **`[validate, verify]`** |

経緯は 3 段である（3 コミットとも初出は `v2.10.0`）:

1. `bf879397`（#1216）が、テストジョブを残したまま**別ワークフローのフルスイート結果（evidence）を要求する**検査を足した
2. `90af77e2`（#1311）がテストジョブを外した（evidence 検査が前提条件を担う形になった）
3. **リリース準備コミット `2a883858`（#1380）がその evidence 検査も外した**

CHANGELOG の逐語:

> Stable release publication no longer requires a separate successful Full Suite evidence artifact for the tagged commit.

**残っている検査**: タグと出所の検証、`bun run check`（静的検査）、`native-smoke`（2 本の単体テスト）、
クロスプラットフォームのビルド、Windows / Unix のライフサイクル導入テスト、チェックサム、来歴証明。
**フルスイートの合否はリリースワークフローの外（PR の CI）に委ねられた。**

---

## 19.11 `## Review` 埋め込み形式の廃止は 2.10.0 でも起きなかった

18.11 で「**次の minor で廃止**」という上流の予告を引いた。`v2.10.0` は minor だが、
**Migration 節は `v2.9.0` と一字一句同じで、読み取りのフォールバックも残っている。** 廃止は持ち越された。

- `reviewer:` → `review_artifact:` の要件（`aidlc-plugin-validate.ts`）は md5 同一で維持
- review-brief の標準エラーは JSON になった

**予告は生きたままなので、`## Review` を追記する実装を抱えている場合の対応の必要性は変わらない。**
ただし「2.10.0 で動かなくなる」という読みは外れた。

---

## 19.12 その他の変更

- **廃止済みの `--init` / `--force` フラグ**は intent の本文に混入せず消費されるようになった。未知の位置引数は拒否される
- 拒否メッセージの多くが、受け付ける値や次に打つべきコマンドを示すようになった
- Construction と CodeKB が、リンクされた Git worktree から正しいプロジェクトを解決するようになった
- `Interaction Events` の宣言と行の数が一致した（13 / 13。18 章で記録した食い違いは解消）
- **`docs/roadmap.md` は依然として「最新安定版 2.8.2」のまま**（18.16 と同じ状態）

---

## 19.13 過去章への影響

| 章 | 影響 |
|---|---|
| [18 章](./18-release-impact-290.md) | 18.6 の 3 件は `v2.10.0` で解消（19.4）。18.15 の「テストが前提条件に戻った」は覆った（19.10）。18.11 の廃止予告は持ち越し（19.11） |
| [6 章](./06-harnesses-install.md) | 手順を `v2.10.0` で再検証し、ハーネス衝突の検査を追加（19.6）。版を含むコマンドを 2.10.0 に更新 |
| [2 章](./02-architecture.md) | フック 19、`core/tools/*.ts` 76。Bedrock の出荷既定の撤去は `v2.10.0` でリリース済み（19.7） |
| [5 章](./05-scopes-depth-test.md) | スコープ 11・所属数は不変。**frontmatter のキーが `guard_policy:` になった** |
| [7 章](./07-learning-loop-state.md) | 監査イベント 99 → 105 |

---

## 19.14 本章を書く過程で見つかった当ノートの誤り

| 誤り | 実際 |
|---|---|
| **6 章の手動コピー手順で、copilot の管理ディレクトリを `.github/` としていた** | アーカイブの `.github/` にあるのは `agents/`・`hooks/`・`skills/` だけ。**`.github/` 丸ごとを入れ替えると、既存の `.github/workflows/` や `CODEOWNERS` が退避側に移り CI が止まる。** 3 つのサブディレクトリを個別に指定するよう直し、実物で確認した（18 章の時点から存在した誤り。当時は claude と codex しか流していなかった） |
| 2 章「（Claude の `balanced` について）`main` ではモデル pin が消え」、README「モデル pin が全面的に撤去された」 | **Claude の `balanced` は `v2.10.0` でも `sonnet` / `medium`。** `null` になったのは codex / opencode の `balanced`（`core/tools/aidlc-tiers.ts` のタグ間 diff） |
| 本章の草稿「既定スコープで Plan Approval の決定的な強制が外れた」 | **初回の Plan Approval は `relaxed` でも拒否される。** 緩んだのは承認後の編集と review freeze（→ 19.3）。フックの stand-aside 分岐を読んで訂正した |
| 本章の草稿「copilot の `AGENTS.md` は専用」とだけ書いていた | 専用であるため**他のハーネスと共存できない**ことまで書く必要があった（上流ガイドの共存節を読んで補った） |

---

## 19.15 本章の限界

- **上流のテストスイートは実行していない**（調査環境に `bun` が無い）
- **19.3 の Guard Policy の挙動は、コードと参照文書の読解による。** `relaxed` のプロジェクトで計画を承認後に編集し、
  再承認なしで進むことを実機で確かめてはいない。メモリ層の `Mode: strict` による巻き戻しも同様
- 19.4 の不具合解消は**到達判定とコード読解**であり、ネイティブ導入で再現・解消を確かめていない
- 19.7 の Bedrock 設定の削除は `aidlc config --yes` を実行して確かめていない
- 手動コピー手順は **claude / codex / copilot / opencode** の導入・更新と、**kiro → kiro-ide の衝突検査**まで流した。cursor（対象外）と kiro / kiro-ide の更新は流していない
- 620 ファイルのうち、`docs/` の小さな変更と AIDA（上流リポジトリ自身の PR レビュー用ワークフロー）の変更は個別に読んでいない
