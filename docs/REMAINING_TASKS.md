# Remaining Tasks

> 未着手・検討中。完了したら削除またはアーカイブする。

---

## 公開後の任意改善

- [ ] LOW 指摘の文言改善（「端到端」→「エンドツーエンド」等）
- [ ] Spec PDF / Method Paper 全文精読後の差分追記
- [ ] `docs/reference/*` 精読後のエンジン内部メモ追加
- [ ] `docs/guide/agents/*.md`・`workshop-mode.md` が 2.5.11 時点で既に存在したか（新規追加か索引漏れか）の切り分け

## 上流追従（2.9.0 まで同期済み）

- [x] 上流 2.5.11 → 2.5.37 の差分反映（2026-08-05）
- [x] 上流 2.5.37 → 2.5.62 の差分反映（2026-08-11）→ [11-release-impact-2562.md](../11-release-impact-2562.md)
- [x] 上流 2.5.62 → 2.6.2 の差分反映（2026-08-14。HEAD `4569754e`）→ [12-release-impact-2602.md](../12-release-impact-2602.md)
- [x] 上流 2.6.2 → 2.6.49 の差分反映（2026-08-22。HEAD `71d9a9e0`）→ [13-release-impact-2649.md](../13-release-impact-2649.md)
- [x] 上流 2.6.49 → 2.6.55 の差分反映（2026-08-22。HEAD `840ba653`）→ [14-release-impact-2655.md](../14-release-impact-2655.md)
- [x] 上流 2.6.55 → 2.6.123 の差分反映（2026-08-29。HEAD `2fbee12f`）→ [15-release-impact-26123.md](../15-release-impact-26123.md)
- [x] 上流 2.6.123 → 2.7.0 の差分反映（2026-09-01。`main` HEAD `96b11d39`。**上流の `v2` ブランチ削除に伴う参照先の一斉更新を含む**）→ [16-release-impact-2700.md](../16-release-impact-2700.md)
- [x] 上流 2.7.0 → 2.8.1 の差分反映（2026-09-09。`main` HEAD `c03f9e28`。**上流の `dist/` 削除に伴う導入手順の全面改訂を含む**）→ [17-release-impact-2801.md](../17-release-impact-2801.md)
- [ ] **追従フローの定例化** — 担当者と頻度を決める
  - **⚠ 2.7.2 で前提が一部変わった。** ネイティブ CLI に `aidlc update` / `aidlc version` /
    `aidlc doctor` が入り、**導入済みの版とリリースの比較**はコマンドでできるようになった。
  - **⚠ ただし `aidlc update --check` は上流 `main` の追跡には使えない。**
    比較対象は**公開済みリリース**であり、`main` はそれより先に進む。
    現に、本ノートの基準 `2931ef02` の時点で **`main` は `v2.9.0` タグより 12 コミット先行しており**、
    2026-09-23 時点では 50 コミット以上先行している（→ 18 章）。
    `update --check` に頼ると「最新です」と言われながら、
    **本ノートが追跡している未リリースのコミットを丸ごと取りこぼす。**
  - **一次のチェックは記録済み HEAD SHA と `origin/main` の比較である。**
    版定数も CHANGELOG も変えないコミットがあるため、**版だけを見ると取りこぼす**。
    ```bash
    # 本ノートが記録している同期地点（README / 各章の `基準:` 行）
    last=2931ef02
    # ⚠ 先に fetch する。`git ls-remote` は SHA を表示するだけで
    #    refs/remotes/origin/main を更新しないため、直後の log / diff が古い参照を見る
    git fetch origin main --tags
    git rev-parse origin/main            # 現在の HEAD
    # 差があれば、その範囲を調べる
    git log --oneline $last..origin/main
    git diff --stat $last..origin/main
    ```
  - 版・リリース状態の確認は**補助**として次を併用する:
    `rg 'AIDLC_VERSION' core/tools/aidlc-version.ts` ／ CHANGELOG の差分 ／
    `git ls-remote --tags origin`（**タグの実在確認**。実装版とリリースは一致しない）
  - `aidlc update --check` は**リリース追従の確認としてのみ**使う
  - 確認コマンド（導入済みなら）: `aidlc version` / `aidlc update --check`
  - 確認コマンド（**上流リポジトリのローカル clone 内**で実行。本リポジトリには `core/` は無い）:
    `rg 'AIDLC_VERSION' core/tools/aidlc-version.ts` と CHANGELOG の差分。
    **実装バージョンと最新タグは一致しないことがあるので `git ls-remote --tags origin` も見ること**
- [ ] **2.6.1 の破壊的変更を読者向け移行手順として点検する** — 永続 state が v8 に上がり、`/aidlc next` / `/aidlc report` / `/aidlc --doctor` が pre-v8 state を拒否する。アップグレード時は `skills/aidlc-application-design/` の**手動削除**が要る（`cp -R` マージでは残る）。実機での再現は未実施
  - **⚠ 2.8.x では `aidlc config` の取り込み規則が別途効く。** 記録済み SHA-256 署名が一致しない
    変更済みファイルは「曖昧」として取り込みを拒否される。手動削除の要否は再確認が要る

## 2.6.2 調査で残った未確認事項

- [ ] `04-agents.md:107` の「`Task` はエージェント禁止」— 全ハーネスの agent 定義を走査して確認していない
- [ ] `05-scopes-depth-test.md:87-88` の `aidlc-graph.ts ars` サブコマンド引数の完全一致
- [x] 「どのハーネスの設定にもメトリクス送出エンドポイントは同梱されていない」— **Cursor を含む `dist/` 全体を再走査し、2.6.2 でも同梱ゼロを確認済み**（2026-08-14）
- [x] `11-release-impact-2562.md:178` の `tools/data/model-rates.json` — **`core/tools/data/model-rates.json` として実在を確認済み**（2026-08-14）
- [ ] `11-release-impact-2562.md:8` の「変更ファイル 1,425 件」— 2.5.37 → 2.5.62 区間の再計測は未実施
- [ ] Cursor ハーネス: `bun scripts/package.ts cursor --check` による dist ドリフト検証、および上流テストの実行（調査環境に `bun` が無い）
- [ ] 2.5.63 が core の `aidlc-review-freeze.ts` に入れたシェルラッパ剥がし強化（`sudo` / `env` / `xargs` / `timeout` 等）が、**既存ハーネスで false positive を生むか**未検証

## 2.6.49 調査で残った未確認事項

- [ ] プロトコルモジュール分割（4 → 8）による固定コンテキスト削減の定量値 — 上流が数値化していない
- [ ] `tests/integration/t304` / `t307` のループバックテストが「3 回」の数値上限自体をアサートしているか — ファイル存在のみ確認
- [ ] DocumentKB の後続段階（「S2」相当）の正式名称・時期 — 上流 issue #714 として予告されるのみ
- [ ] `core/tools/aidlc-swarm.ts` 本体が Testing Contract を検証する具体的コード位置
- [ ] `bun scripts/package.ts codex trust` の内部ハッシュ生成ロジックが 2.6.44 で変わったか
- [ ] Kiro IDE の新 `.kiro/hooks/*.json` が実機 Kiro IDE 1.x で実際に SessionStart 発火するか — 実機未検証
- [ ] `<active-space>` プレースホルダが claude / codex / cursor / kiro / kiro-ide の現行 dist に残っていないかの横断 grep — 該当コミットの変更ファイル範囲でのみ確認
- [ ] 上流ドキュメントの stale 3 箇所（`docs/guide/05-scopes-and-depth.md` のキーワード表 fallback 行、`docs/reference/03-orchestrator.md`、`core/scopes/aidlc-feature.md` 本文）が意図的な緩さか単なる更新漏れか — 上流 Issue / PR 未参照

## 2.6.55 調査で残った未確認事項

（`scratch/` は Git 管理外の作業ディレクトリなので、根拠は本リポジトリには残らない。判断の経緯は [14-release-impact-2655.md](../14-release-impact-2655.md) の 14.9 を参照）

- [ ] 2.6.54 より前に生成された**実データ**の監査シャードに、無関係な `SUBAGENT_COMPLETED` が
      実際に混入しているか — 調査環境にサンプルが無く、コード構造から導かれる可能性のみ
- [ ] 上流 issue #695 の `Directory not found` fumble が本当に `memory.md` のプローブだったか —
      `Closes #695` は上流の主張で、差分中に両者を結ぶ記述は無い
- [ ] read-probe の失敗が、Kiro CLI を ACP 経由で動かした場合以外（他の 6 ハーネス、および Kiro CLI の TUI 経路）でも起きていたか — 上流にも実測が無い（**ACP はハーネスではなく Kiro CLI の接続方式**）
- [ ] 2.6.55 による改善量（削減されたターン数）— 上流も定性的記述にとどまる
- [ ] 「新旧混在はロードエラー」は ESM の named-import 契約からの**推論**。
      実際に半々のツリーで実行して確かめていない
- [ ] ネットワーク FS（NFS / SMB / FUSE / オブジェクト同期フォルダ）上で、
      実際にどのタイミングでどのエラーが出るか — 上流の unsupported 宣言を確認したのみ
- [ ] `AIDLC_SKIP_REVIEWER_GATE_GUARD=1` を一般利用者が設定できないようにする機構の有無
- [ ] 上流テストスイートは未実行（調査環境に `bun` が無い）

## 2.6.123 調査で残った未確認事項

（`scratch/` は Git 管理外の作業ディレクトリなので、根拠は本リポジトリには残らない。判断の経緯は [15-release-impact-26123.md](../15-release-impact-26123.md) を参照）

- [ ] `bugfix` / `refactor` の**旧**承認ゲート数の絶対値 — 新値 6 / 7 は CHANGELOG が明示しているが、
      旧 `bugfix` 4 は旧ドキュメントの例から読めるだけで、旧 `refactor` は算術推定にとどまる
- [ ] 2.6.121 の `incompleteFallback` が advisory ステージで実際にゲートを開けるか — コード読解による推論
- [ ] `Review Challenge` を conductor が自作して受領証を通せる具体的攻撃経路 — 同上
- [ ] 2.6.74（品質目標）の実効性 — エージェントが実際に閾値の緩和を拒否するか
- [ ] 2.6.116 / 2.6.112 の効果 — 散文ルールのみでガードが無いため、動作としては未検証
- [ ] 2.6.71 の localization が各ハーネスでどこまで日本語化されるか
- [ ] 2.6.119 / 2.6.117 の文脈削減が実トークン消費・コスト・レイテンシをどれだけ下げるか
      — 測ったのは Git オブジェクトのバイト数のみ
- [x] `audit-format.md` の "Interaction Events (10 events)" と実際の表行数（9 行）の不一致 —— **`main` で解消（下記「2026-09-23 解決」の項）**
- [ ] 2.6.92 の混在セパレータ問題が実データで悪用可能だったか — コード構造上の経路のみ確認
- [ ] Copilot / opencode がエンジン再インストール後に compose フックで自己修復するか
      — 上流は Claude / Codex / Cursor / Kiro IDE と Kiro CLI しか名指ししていない
- [ ] `aidlc-plugin-validate.ts` と compose のテストペイロード判定の非対称
      （validate は `tests` / `fixtures` / `*.test.ts` のみ error、compose は `__tests__` / `*.spec.ts` も drop）が
      意図的か検証漏れか
- [ ] 上流 `10-authoring-a-plugin.md` が「neither an AIDLC project nor a framework checkout」と書く一方、
      `aidlc-plugin-test.ts` は `--install <project-root>` が必須 — 誤記か意図的か断定できない
- [ ] 2.6.99 の active-directive ロックが `withAuditLock` と物理的に同一実装か — シンボルレベルまで未追跡
- [ ] （継続）上流テストスイートは未実行 → 「2.6.55 調査で残った未確認事項」の同項目を参照。本区間でも解消していない

## 2.7.0 調査で残った未確認事項

（判断の経緯は [16-release-impact-2700.md](../16-release-impact-2700.md) を参照）

- [ ] 2.6.124 の「既存の絶対パスは次の書き込みまで無害」— `projectRootFor` の再導出とフォールバック順序を
      コード読解で判断したもので、旧い `aidlc-state.md` を持つ実プロジェクトでは確認していない
- [ ] 既存の `aidlc-state.md` に残る絶対パスを**明示的に消す**手段があるか — 上流は移行を提供しないと明言しており、
      手で書き換えてよいかは不明（`Project Root` は到達しないフォールバック、`Worktree Path` は表示用と読める）
- [ ] `release.yml` / `release-pr.yml` / `codebuild.yml` の dispatch シムは YAML 読解のみ。実際に起動していない
- [ ] 上流 `docs/roadmap.md` の「current v2 version is 2.6.124」が単なる追随漏れか、
      2.7.0 を roadmap 上の版として数えない意図かは判断できない
- [ ] #968（Devin CLI / Desktop ハーネス）がマージされた場合のハーネス 8 種目の扱い — PR 段階のため未追跡
- [ ] `v2_backup`（`d898b74e`）が何のために残されているか — 旧 `v2` HEAD ではないことのみ確認済み
- [ ] （継続）上流テストスイートは未実行

## 2.8.1 調査で残った未確認事項

（判断の経緯は [17-release-impact-2801.md](../17-release-impact-2801.md) を参照）

- [ ] **ネイティブインストーラを実機で走らせていない** — `install.sh` / `install.ps1` の記述は
      スクリプトとドキュメントの読解によるもので、導入結果を確認していない
- [ ] **`aidlc config` がプラグインの compose フックを自動実行するか** — 上流ドキュメントは
      「プラグイン合成ファイルと記録済みステージ寄与を保持してグラフを再生成する」と書くが、
      compose フックの自動実行は明記していない。`/aidlc plugin sync` の完全代替かは不明
- [ ] **Windows ARM64 でのネイティブバイナリの挙動** — `install.ps1` にアーキテクチャ判定が無く、
      `aidlc-windows-x64.exe` を固定で取得する。エミュレーション前提かどうか上流に記述が無い
- [x] **次のリリースタグがいつ付くか（2026-09-17 解決）** — `v2.8.1`（09-09）/ `v2.8.2`（09-11）/ **`v2.9.0`（09-15、現 Latest）** が公開された。
      **タグ `v2.8.1` = `215afe1a` で、当方の基準 `c03f9e28` の 5 コミット後を指す**（→ [18.1](../18-release-impact-290.md)）
- [x] **リリース時のフルテストスイートの無効化がいつ解除されるか（2026-09-17 解決）** —
      `TEMPORARY` は HEAD から消えた。コメント解除ではなく**専用ジョブ 4 本への分割**で復活し、
      `native-smoke` が `test` に依存するようになった（→ [18.15](../18-release-impact-290.md)）
- [ ] **🔴 現 Latest の v2.9.0 に残るセンサー・フック系の不具合 3 件（①②はネイティブ導入限定、③は Bun 経路でもフックの PATH 次第で起こる）を追跡する** —
      **#1070（Review brief）/ #1166（ゲートのセンサー）/ #1249（3 つのコアフックが `bun` を直接名指し）**。
      **#1070・#1166 は preview `.20260920.1` 以降で修正済み。#1249 はどの preview にも未収録。**
      **#1249 は doctor が警告できず、Stop フック・runtime-graph 再構築・Write 契機センサーが無言で死ぬ。**
      `git fetch origin --tags` の後に **`git merge-base --is-ancestor f79e321b <tag>`** で判定する
      （`f79e321b`＝#1249 の修正が 3 件中いちばん新しいので、現時点ではこれ 1 つで 3 件とも見られる。
      **`be94bde7` や `c97fa7ba` で判定すると #1249 を取りこぼす。**
      **修正が増えるたびに基準コミットを見直すこと。** → [18.6](../18-release-impact-290.md)）
- [ ] **上流 open PR #1157（Kiro IDE と Kiro CLI の配布統合）を追跡する** —
      マージされると**ハーネスが 7 → 6 になりうる**
- [ ] **`bugfix` / `refactor` の frontmatter 所属数（9 / 10）と出荷表の EXECUTE 数（7 / 8）の食い違い** —
      本区間で生じた差ではなく、2.8.1 でも同じ。上流へ照会していない（→ [18.12](../18-release-impact-290.md)）
- [x] **`audit-format.md` の宣言件数と表行数の不一致（2026-09-23 解決）** —
      `Interaction Events` は基準 `c03f9e28` が「宣言 10 / 行 9」、タグ `v2.8.1` 以降が「宣言 11 / 行 10」だった。
      **上流 `main` の `261083ce`（#1150）で宣言 13 / 行 13 に是正された。安定版 `v2.9.0` には未収録。**
- [ ] **上流内に残る `dist/` 前提の記述** — `core/tools/aidlc-init.ts:6523,6531` と
      `docs/guide/12-cli-commands.md:1128` が旧手順を指示している。上流の同期漏れか意図的かは不明
- [ ] **上流内のインストール手順の不一致** — `README.md:20` は `curl | sh`、
      `docs/guide/harnesses/README.md:18-23` は `mktemp` + `curl -o` + `sh` の 2 段階。
      どちらが正式かを上流が示していない
- [ ] **`dist/` / `dist-release/` の生成物の実バイトを検証していない** — ワーキングツリーに存在しないため、
      パッケージャのコードからの読解にとどまる
- [ ] **#968（Devin CLI / Desktop ハーネス）の判定基準を再定義する** — 従来は「`dist/` に実体があるか」で
      判定していたが、`dist/` が消えたため使えない。`harness/` 直下かリリース資産で判定する。
      なお **#996「feat: Devin Harness」も別に OPEN** で、Devin ハーネスの PR が 2 本並存している
- [x] **🔴 v2.8.0 の Copilot / Cursor フック不具合の解消時期を追う** —— **2026-09-17 解決。`v2.8.1`（2026-09-09 20:12 UTC）以降に `52da70ad` が入っている。**
      ~~旧記述~~: —— 修正は **`52da70ad`**（#1065）で入ったが未リリース。**公開時の版番号は未確定**（コミット本文は 2.8.2、CHANGELOG は 2.8.1 に統合）。
      **`52da70ad` を含むリリースが公開されるまで、この 2 ハーネスは v2.8.0 で導入できない**
      （**版番号は未確定**。新タグが出たら **`git fetch origin --tags` してから**
      `git merge-base --is-ancestor 52da70ad <tag>` で修正の有無を確認する）

> **📌 2026-09-17 時点の上流（本区間は追従済み）**
>
> | 項目 | 値 |
> |---|---|
> | `main` HEAD | `2931ef02` / `AIDLC_VERSION` = **2.9.0** |
> | 本区間の基準からの差 | **50 コミット / 473 ファイル / +57,789 −8,314** |
> | リリース | v2.8.1（09-09）・v2.8.2（09-11）・**v2.9.0（09-15、Latest）** |
> | preview チャネル | `v2.8.3-preview.20260914.1` / `v2.9.1-preview.20260915.1`（**新設。上流 #1008**。2026-09-23 時点では 4 本） |
> | Copilot / Cursor 不具合 | **v2.8.1 以降で解消済み**（`52da70ad` を含む） |
>
> **⚠ 当初この表に「`be94bde7` / 45 コミット / 408 ファイル」と記録したが、その数値は誤りだった**
> （`be94bde7` の実測は 48 / 473。45 や 408 に一致する地点は区間内に存在しない）。
>
> **この区間の追従は [18 章](../18-release-impact-290.md) で完了した。**

- [x] **上流 2.8.1 → 2.9.0 の追従（2026-09-17 完了）** —— `c03f9e28` → `2931ef02`、50 コミット / 473 ファイル。
      #1000（Plan Approval の内容束縛・Change Control）と #1151（`classic` 26 → 18）を含む。→ [18 章](../18-release-impact-290.md)
- [ ] （継続）上流テストスイートは未実行

## 運用

- [ ] GitHub リポジトリの Topics / Description 整備
- [ ] 必要なら GitHub Pages や簡易目次の追加
- [ ] **`DENYLIST_PATTERNS` シークレットの設定**（未設定の間はリークチェックが**失敗する**。fail-closed のため、設定するまで PR はマージできない。手順は `PUBLIC_CONTENT_POLICY.md`）
- [x] master ブランチ保護ルールの設定（直接 push 禁止・PR 必須・CI パス必須。2026-08-05 設定済み）
  - **fork PR との相互作用**: leak-check は fork PR で必ず失敗するため（secrets 不達）、外部 PR は管理者バイパスなしにはマージできない。手順は `PUBLIC_CONTENT_POLICY.md` の「fork PR の運用手順」を参照
