# 17. リリース差分 2.7.0 → 2.8.1 — `dist/` 廃止とネイティブ配布

作成日: 2026-09-09
基準: `awslabs/aidlc-workflows` branch `main` HEAD `c03f9e28`（取得 2026-09-09）／実装バージョン **2.8.1**
前回: [16-release-impact-2700.md](./16-release-impact-2700.md)（2.6.123 → 2.7.0、HEAD `96b11d39`）

CHANGELOG の実エントリ **4 件**（2.7.1 / 2.7.2 / 2.8.0 / 2.8.1）。
コミット **11 件**、変更ファイル **2,636 件**（+47,594 −978,021 行）、期間 **8 日**（上流コミット日 2026-09-01 〜 09-08、両端を含む）。

削除行が 97 万行あるが、**これは規模縮小ではない**。
`dist/` ディレクトリ 2,201 ファイル（−972,067 行）が版管理から外れた分がほぼ全量である。
`dist/` を除いた実質の差分は **435 ファイル / +47,594 −5,954 行**で、ソースは増えている。

前回の 16 章は「参照先そのものが差し替わった」章だった。
**本章は「配布物そのものが無くなった」章である。**
6 章が全編にわたって説明してきた `cp -R dist/<harness>/ …` という導入手順は、上流から消えた。

---

## 17.1 いちばん大きい変更は `dist/` の消滅

| | 変更前（2.7.0 / 2026-09-01） | 変更後（2.8.1 / 2026-09-08） |
|---|---|---|
| `dist/` | リポジトリにコミットされた配布物 2,201 ファイル | **リポジトリに存在しない**。`.gitignore` に `/dist/` `/dist-release/` |
| 導入方法 | `git clone` → `cp -R dist/<harness>/. <project>/` | `install.sh` / `install.ps1` でネイティブ `aidlc` を導入 → `aidlc config` |
| 前提ランタイム | Bun が必須 | **Bun / Node.js とも不要**（単一バイナリ） |
| 手動コピー派の入手元 | リポジトリの `dist/<harness>/` | リリース資産 `aidlc-runtime-X.Y.Z.tar.gz` 内の **`runtime/<harness>/`**。**ネイティブ `aidlc` の導入が前提**（下記） |

`.gitignore` の変更（逐語）:

```diff
-# Generated AI-DLC artifacts (recreated by workflow runs / doctor)
-dist/claude/aidlc-docs/
-dist/kiro/aidlc-docs/
-dist/codex/aidlc-docs/
+# Generated AI-DLC projections (materialized locally by scripts/package.ts)
+/dist/
+/dist-release/
```

上流自身のコメントが示すとおり、`dist/` は「**ローカルで `scripts/package.ts` が materialize する生成物**」という位置づけに変わった。

> **⚠ 「実装版」と「リリース」を分けて読むこと。**
> 配布方式の変更が入った実装版は **2.7.2**（`12b8d6e0` / #756）である。
> **ただし `v2.7.1` と `v2.7.2` のタグは存在しない**（実測: リモートのタグは `v2.7.0` と `v2.8.0` のみ）。
> したがって:
>
> | 観点 | 境界 |
> |---|---|
> | **実装版** | 2.7.1 以前が `dist/` コピー方式、**2.7.2 以降**がネイティブ配布 |
> | **利用者が入手できるリリース** | v2.7.0 までが `dist/` 方式、**v2.8.0 が新方式を載せた最初のリリース** |
>
> 本ノートで「2.8.x で変わった」と書いている箇所は**リリース観点**である。
> 上流のコミットや CHANGELOG を追う場合は **2.7.2** を境界として見ること。


### 新しい導入手順

上流 `README.md:20-26` の記載（逐語）:

```bash
curl -fsSL https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.sh | sh
```

```powershell
irm https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.ps1 | iex
```

導入後:

```bash
cd /path/to/your-project
aidlc config --harness claude
aidlc doctor
```

`install.sh` のオプション（`scripts/install.sh:82` 逐語）:

```
Usage: install.sh [--version <x.y.z>] [--from <dir>] [--offline] [--profile <startup-file>] [--json|--quiet] [--no-color] [--yes]
```

導入先は Unix が `~/.local/bin`（`$AIDLC_BIN_DIR`）と `${XDG_DATA_HOME:-~/.local/share}/aidlc`（`$AIDLC_INSTALL_ROOT`）、
Windows が `%LOCALAPPDATA%\aidlc` で、コマンド名は `aidlc.cmd` になる。
root での実行は拒否される。Homebrew / Nix が管理する既存の `aidlc` があれば置き換えず譲る。

### ⚠ 上流内部でインストール手順が食い違っている

同じ HEAD の中に 2 通りの手順が併存している。

| 出典 | 手順 |
|---|---|
| `README.md:20` | `curl -fsSL … /install.sh \| sh`（パイプ直結） |
| `docs/guide/harnesses/README.md:18-23` | `mktemp -d` → `curl -o "$tmp/install.sh"` → `sh "$tmp/install.sh"` → `rm -rf "$tmp"` |

`README.md` は 2026-09-08 の #1053「docs: simplify installation and quick start」で
2 段階版からパイプ直結版へ**短縮された**が、ハーネス側ガイドは 2 段階版のまま残っている。
**2 段階版はダウンロードした内容を実行前に確認できる。**
どちらを採るかは導入側の方針だが、上流が一方に統一していない事実は把握しておくとよい。

> **⚠ 手動コピー経路でも、ネイティブ `aidlc` バイナリの導入は必須である。**
> `runtime/<harness>/` の投影は**ネイティブ `aidlc` を呼ぶ形**に書き換えられており、
> **tar.gz にバイナリ本体は入っていない**。
> 上流 `README.md:33-36` も「**Install the matching native `aidlc` command**, download
> `aidlc-runtime-X.Y.Z.tar.gz` …, and copy `runtime/<harness>/` into your project」と
> **バイナリ導入を先に置いている**。
> **手動コピーは「ネイティブ導入の代替」ではなく「プロジェクト内ファイルを手で置く」という選択である。**
> バイナリを入れずに `runtime/<harness>/` だけ置くと、フックもコマンドも起動できない。
> **また上流は「Install the *matching* native `aidlc` command」と書いており、
> バイナリとアーカイブの版を揃える必要がある**（`aidlc version` で確認できる）。

### `dist/` のコピーは今も可能か

チェックアウトからの生成自体は今も動く。`bun scripts/package.ts <harness>` は
`dist/<harness>/` と `dist-release/<harness>/` の**両方**を生成する。
ただし上流はこれを利用者向けの経路とは認めていない（`docs/guide/12-cli-commands.md:243-246` 逐語）:

> The native config command is the preferred project installation and refresh path.
> Framework developers may generate the ignored Bun-shaped `dist/` projection locally with
> `bun scripts/package.ts`; **release users should not copy from a checkout.**

2 つの生成物は呼び出し形が違う。`dist/` は従来どおり `bun <harness>/tools/…` を呼ぶ Bun 前提の投影で、
`dist-release/` はネイティブ `aidlc` を呼ぶ形に書き換えられている。
リリース資産 `aidlc-runtime-X.Y.Z.tar.gz` に入る `runtime/<harness>/` は**後者**である。

### ⚠ 上流にも `dist/` 前提の記述が残っている

2.8.1 は `aidlc doctor` の修復指示から `dist/` を外した（`core/tools/aidlc-utility.ts` の差分、逐語）:

```diff
-    fix: `copy the workspace shell from \`dist/${harnessDir().replace(/^\./, "")}/\` into your project root`,
+    fix: "run `aidlc config` in the project root to create the harness tree and workspace shell",
```

**だが直しきれていない。** `core/tools/aidlc-init.ts:6523,6531` と
`docs/guide/12-cli-commands.md:1128` には、いまも `dist/<harness>/` の再コピーを促す文言が残る。
上流の文言側の同期漏れであり、読者が旧手順に誘導されうる。

---

## 17.2 数値の変化 — 動いたのは `core/tools` だけ

| 項目 | 2.7.0 | 2.8.1 | 測定方法 |
|---|---:|---:|---|
| フェーズ | 5 | 5 | `core/aidlc-common/stages/` の第 1 階層 |
| ステージ | 33 | 33 | `core/aidlc-common/stages/` の `.md` 数 |
| スコープ | 11 | 11 | `core/scopes/*.md` |
| エージェント | 14 | 14 | `core/agents/*.md` |
| センサー | 6 | 6 | `core/sensors/*.md` |
| プロトコル | 8 | 8 | `core/aidlc-common/protocols/*.md` |
| TypeScript フック | 18 | 18 | `core/hooks/*.ts` |
| 監査イベント種別 | 91 | 91 | `aidlc-audit.ts` の `VALID_EVENT_TYPES` |
| ハーネス | 7 | 7 | **`harness/` 直下**（測定方法を変更。下記） |
| **`core/tools/*.ts`** | **51** | **69** | 直下のみ。**+18** |
| うち `import.meta.main` を持つもの | 32 | 39 | +7 |

> **⚠ ハーネス数の測定方法を変えた。**
> 16 章までは「`dist/` 直下のディレクトリ数」で数えていたが、`dist/` が消えたため**再現できない**。
> 本章以降は `harness/` 直下で数える。両版とも 7 で一致する
> （`claude` / `codex` / `copilot` / `cursor` / `kiro` / `kiro-ide` / `opencode`）。
> なお `harness/` ツリーは 2 版の間で**ファイル一覧が完全一致**している。

### ワークフロー体系は 1 つも動いていない

- ステージ定義ファイルは **33 → 33 で集合が完全一致**する。
- frontmatter の差分は **`name:` 9 行の追加のみ**で、
  `slug` / `phase` / `execution` / `gate` / `reviewer` / `review_artifact` に変更はない。
  この `name:` は表示名で、2.7.0 ではコンパイル済みグラフ側に固定されていた値が
  ソース側へ移されたものである（`compileStageGraph()` が既存 `stage-graph.json` に依存しなくなったため）。
- `plugins/` も 2 版で一致（`test-pro` 1 件 / 17 ファイル）。

### 追加された 18 本のツール（削除はゼロ）

```
aidlc-archive          aidlc-color             aidlc-command
aidlc-completions      aidlc-config-diagnostics aidlc-distribution
aidlc-doctor           aidlc-init              aidlc-install-paths
aidlc-lifecycle        aidlc-machine-config    aidlc-model-policy
aidlc-plugin           aidlc-release           aidlc-settings
aidlc-transaction      aidlc-update            aidlc-windows-uninstall
```

**18 本すべてがインストール・配布・更新・設定のライフサイクル系である。**
ワークフロー実行系のツールは 1 本も増えていない。
`core/` の増加 +22,148 行のうち **18,393 行（83%）がこの 18 本**である（`git diff --numstat` で実測）。

### 変更規模の内訳

| 対象 | ファイル | 追加 | 削除 |
|---|---:|---:|---:|
| `core/` | 106 | +22,148 | −953 |
| `harness/` | 78 | +752 | −428 |
| `tests/` | 166 | +18,128 | −1,554 |
| `docs/` | 63 | +2,758 | −2,097 |
| `scripts/` | 10 | +2,998 | −437 |
| その他（`.github` / 設定 / `CHANGELOG` 等） | 12 | +810 | −485 |
| `dist/`（削除） | 2,201 | 0 | −972,067 |
| **合計** | **2,636** | **+47,594** | **−978,021** |

`harness/` の 78 ファイルは**大半が呼び出し文字列の書き換え**である
（`bun {{HARNESS_DIR}}/tools/…` → `{{INVOKE}} engine …`）。
**ただし例外がある** —— `harness/claude/settings.json` は
無制限 `Bash` の自動許可とモデル固定の削除という実質的な変更を含む（→ [17.6](#176-claude-code-出荷設定からモデル固定と無制限-bash-許可が消えた)）。

> **⚠ ファイル数を単独で引用しないこと。**
> リポジトリ全体のファイル数は 3,375 → 1,217 と激減するが、
> これは `dist/` の版管理除外が原因で、`dist/` を除けば 1,174 → 1,217 と**増えている**。
> 「2,158 ファイル減った」とだけ書くと、実態と逆の印象になる。

---

## 17.3 ⚠ 2.8.1 は CHANGELOG にあるが、リリースされていない

**本章でいちばん実務に効く注意点である。**

| | 実測値 |
|---|---|
| `core/tools/aidlc-version.ts` の `AIDLC_VERSION` | **2.8.1** |
| リモートに存在するタグ | **`v2.8.0` まで**（`git ls-remote --tags origin 'v2.8*'`） |
| `v2.8.0` が指すコミット | `0d399dd8`（HEAD はその 2 コミット先） |
| GitHub Release の Latest | **v2.8.0**（2026-09-08 09:24 UTC） |

リリースワークフローは `on: push: tags: ["v*.*.*"]` でしか起動せず、
さらに `validate` ジョブが `test "$RELEASE_TAG" = "v$version"` で
タグと `AIDLC_VERSION` の一致を強制する。
**`v2.8.1` タグが push されるまで、2.8.1 の資産は 1 つも存在しない。**

したがって CHANGELOG 2.8.1 の Upgrade 行が案内する

> **Upgrade:** `aidlc update`, or `install.sh --version 2.8.1` / `install.ps1 -Version 2.8.1`

のうち、**`--version 2.8.1` の指定は現時点では成立しない**（指定先のリリースが無いため）。
`aidlc update` も latest として 2.8.0 を返す。

> **🔴 v2.8.0 では GitHub Copilot と Cursor のフックが動作しない（2026-09-09 追記）。**
> **根拠は上流コミット `52da70ad`（#1065）の本文と 2.8.1 の CHANGELOG である。**
> **本調査で実機再現はしていない**（→ 各文書の「限界」節）。
> **リリース済みの 2.8.x は v2.8.0 だけなので、この 2 ハーネスでは現時点の推奨導入先が壊れている。**
>
> 上流コミット `52da70ad`（#1065）の本文（逐語要旨）:
> 2.8.0 のネイティブ化はすべての `bun <dir>/hooks/aidlc-<name>.ts` を `aidlc engine hook <name>` に
> 書き換えたが、**アダプタ経路へ回したのは kiro と codex だけ**だった。
> Copilot と Cursor のアダプタは引数 1 個のフック経路に落ち、対象が捨てられる。
>
> | ハーネス | v2.8.0 での症状 |
> |---|---|
> | **GitHub Copilot** | 全イベントで `undefined is not an object (evaluating 'input.length')` でクラッシュ |
> | **Cursor** | `guards` が対象に一致せず、**fail-closed な preToolUse の裏で無出力終了 → 全ツール呼び出しがブロックされる** |
>
> **Cursor の症状は 2.5.63〜2.5.68 の既知不具合と同じ失敗様式である**（空 stdout × `failClosed`）。
>
> 修正は **2.8.1**（`52da70ad` ほか）だが、**2.8.1 はまだリリースされていない**。
> したがって現時点の選択肢は次のいずれかになる。
>
> - **Copilot / Cursor 以外のハーネスを使う**（Claude Code / Codex CLI / Kiro / opencode は影響を受けない）
> - **`v2.8.1` タグの公開を待つ**
> - ソースから生成する経路を採る（`bun scripts/package.ts`。**bun が要る**）
>
> **本ノートが `install.sh`（= `releases/latest` = v2.8.0）を案内している箇所は、
> この 2 ハーネスについては上記の制約付きで読むこと。**

> **⚠ 本章の測定後、上流はさらに進んでいる（2026-09-09 時点の追記）。**
> 本章の `基準:` は `c03f9e28` だが、その後 **`52da70ad` まで 3 コミット**進んだ
> （179 ファイル / +26,596 −3,694）。`AIDLC_VERSION` は **2.8.1 のまま**で、タグも **v2.8.0 のまま**である。
> 内訳は #1064（Codex の pre-tool 入力書き換え許可）、
> **#1000（Plan Approval の内容・試行への束縛、human-only exit、Kiro IDE の Windows シェル保護、Change Control）**、
> **#1065（Copilot / Cursor のフック経路修正。下記）**。
> **本章の測定値は `c03f9e28` 時点の記録として維持する。** 次回区間で扱う。

**実際に到達できる最大は 2.8.0 である。**
2.8.1 の 2 件の修正（`aidlc config` ウィザードで Enter が既定値として受理されない不具合、
`aidlc update` が同一版で umask 依存の整合性検査に失敗する不具合）は、まだ利用者に届いていない。

> **版を固定するときは、用途で使い分けること。**
> - **導入・インストールを固定する → `v2.8.0`**（資産が実在する唯一の 2.8.x リリース）
> - **本ノートのソース照合を固定する → SHA `c03f9e28`**（タグは無く、リリース資産も無い）
>
> **`v2.8.1` と書くと、存在しないタグを指す手順書になる。**

### 2.x が初めて「実体のある」リリースになった

16 章では「2.x が初めて GitHub Release として公開された」と記録した。
今回、その意味が変わった。

| タグ | Latest | 公開日 | 資産 |
|---|---|---|---:|
| v2.8.0 | **Latest** | 2026-09-08 | **13 件** |
| v2.7.0 | — | 2026-09-01 | **0 件** |

v2.7.0 は Release ページに 2.x が載っただけで、**添付資産は 1 件も無かった**。
v2.8.0 で初めて実配布物が付いた（実測した 13 件）:

```
aidlc-darwin-arm64    aidlc-darwin-x64      aidlc-linux-arm64
aidlc-linux-arm64-musl aidlc-linux-x64      aidlc-linux-x64-musl
aidlc-windows-x64.exe  aidlc-runtime-2.8.0.tar.gz
install.sh            install.ps1           checksums.txt
aidlc-release.intoto.jsonl                  version.json
```

`README.md` の `curl … /releases/latest/download/install.sh` が実際に成立するようになったのは、
**2.8.0 からである**。

### 対応プラットフォーム

リリースされるのは 7 ターゲット:
`linux-x64` / `linux-x64-musl` / `linux-arm64` / `linux-arm64-musl` /
`darwin-x64` / `darwin-arm64` / `windows-x64`。

- **Windows は x64 のみ。** `scripts/install.ps1` にアーキテクチャ判定が無く、
  `aidlc-windows-x64.exe` を固定で取得する。Windows ARM64 での挙動は上流に記述が無い（未確認）。
- **Alpine（musl）は C++ ランタイムが要る。** `install.sh` が失敗を検出して
  `apk add libgcc libstdc++` を案内する。上流 CI の `musl-smoke` ジョブも同じ前提で動く。

---

## 17.4 2.7.1 — solo ワークフローで Plan Approval がデッドロックしていた

**本調査で確認できた範囲では、4 版のうちワークフローの挙動そのものを直したのはこの 1 版だけである**
（`core/` の全変更ファイルを分類した結果によるもので、全 hunk を精読したわけではない）。

`core/tools/aidlc-orchestrate.ts` の `emit()` が、
Stop フックの read-only な `next` プローブでも「active-directive マーカー」を publish していた。
publish は `code_generation_authority_revision` を進め、`resetPlanApprovalRuntime` を呼ぶ。
人間の「Approve Plan」を記録するにはターン境界が要るため、
**そのターンで発行された承認チャレンジが、同じターンの Stop プローブ自身に破棄されていた。**

修正前は抑止条件が「プローブ **かつ** team unit ownership」だったため、
team 運用では発生せず、**solo（非 team）ワークフローだけが踏んだ**。

```diff
-function isTeamStopHookProbe(projectDir: string | undefined): boolean {
-  if (process.env.AIDLC_STOP_HOOK_PROBE !== "1" || !projectDir) return false;
-  try { return isTeamUnitOwnership(readStateFile(projectDir)); } catch { return false; }
-}
+function isStopHookProbe(): boolean {
+  return process.env[STOP_HOOK_PROBE_ENV] === "1";
+}
```

症状は、Approve Plan を選んだあと Code Generation に永久に入れず、
developer への dispatch が毎回「not currently approved」で拒否され続けるというもの。
conductor 自身の（プローブでない）`next` は従来どおり publish するので、前進動作は変わらない。

### ⚠ CHANGELOG の「回避策」記述はそのまま採らないこと

CHANGELOG 2.7.1 は次のように書く。

> `AIDLC_DISABLE_PLAN_APPROVAL_GUARD=1` and driving the workflow as team-owned
> are no longer needed as workarounds for this deadlock.

**このうち環境変数の側は、完全な回避策ではなかった。**
この変数を読むのは `core/hooks/aidlc-plan-approval-guard.ts` の 1 箇所だけで、
PreToolUse フックによる拒否は止まる。
しかし `beginCodeGeneration`（`core/tools/aidlc-testing-posture.ts`）の
承認レシート検査は環境変数と無関係に throw するため、**承認そのものは復旧しない**。

完全に避けられたのは **team-owned で駆動する側**である（上記の抑止条件がプローブの publish を止めるため）。
上流の記述を「どちらでも回避できた」と読むと、当時の被害範囲を過小評価することになる。

---

## 17.5 設定に階層ができた — ガードレールを恒久的に記録できる

2.8.x で `aidlc.settings.json` という設定ファイル体系が入った。

| レイヤ | 位置 |
|---|---|
| 出荷既定 | tier / preset テーブル |
| machine | `${AIDLC_INSTALL_ROOT:-~/.local/share/aidlc}/aidlc.settings.json` |
| project | `<project>/aidlc.settings.json`（プロジェクトルート直下） |
| local | `<project>/aidlc.settings.local.json`（`.gitignore` に自動追記） |
| 環境変数 | 最優先 |

優先順位は **出荷既定 < machine < project < local < 環境変数**で、
マージは leaf 単位である（`agents.architect.model.claude` を local が上書きしても
同じエージェントの `.codex` は machine の値のまま残る）。
machine 専用キー（`update-check` / `offline` / `release-base-url` / `ca-bundle`）を
project / local に書くと fail-closed で拒否される。未知キーも同様である。

### ⚠ ガードレール 9 種を設定ファイルに記録して無効化できるようになった

`core/tools/aidlc-settings.ts` の `RECORDABLE_PROJECT_BYPASSES`（逐語）:

```
AIDLC_SKIP_ARTIFACT_GUARD
AIDLC_SKIP_REVISION_BACKSTOP
AIDLC_SKIP_SUMMARY_CONFIRMATION_GUARD
AIDLC_SKIP_HUMAN_PRESENCE_GUARD
AIDLC_DISABLE_ENSEMBLE_EVIDENCE
AIDLC_DISABLE_PLAN_APPROVAL_GUARD
AIDLC_DISABLE_REVIEWER_SCOPE_HOOK
AIDLC_DISABLE_REVIEW_FREEZE_HOOK
AIDLC_DISABLE_USAGE_TRACKING
```

`core/tools/aidlc-lib.ts` の `resolveProjectFlag()` は、
**環境変数が未設定なら `aidlc.settings.json` の `flags.bypasses` を見て `"1"` を返す。**

これは両刃である。

- 従来これらは実行のたびに環境変数を置く必要があり、**痕跡が残らなかった**。
  設定ファイルに記録されれば、誰がいつ何を無効化したかが**版管理の対象になる**。
  なお設定は書き換えも削除もでき、環境変数が最優先である。**不可逆になったわけではない。**
- 一方で、**コミットされた設定ファイルを読まないとガードの実効状態が判断できない**。
  「環境変数を確認したから大丈夫」という点検は成り立たなくなった。

`AWS_AIDLC_DEFAULT_SCOPE` / `AIDLC_USE_SWARM` / `AIDLC_HOOK_DEBUG` / `AIDLC_SENSOR_TIMEOUT_MS`
の 4 つも `resolveProjectFlag()` 経由で設定ファイルから解決される。
ただし**参照先は別**である —— バイパス 9 種は `flags.bypasses` の配列、
この 4 つは `PROJECT_FLAG_FIELDS` による個別フィールド（`defaultScope` / `swarm` /
`hookDebug` / `sensorTimeoutMs`）へのマッピングである。

### `aidlc config` の初回ウィザード

未設定プロジェクトで TTY から引数なしで `aidlc config` を実行したときだけ、初回ウィザードが動く。

> **⚠ 6 ステップは「カスタマイズを選んだ場合」の経路である。常にこの 6 問から始まるわけではない。**
> 上流 `docs/guide/18-install-and-lifecycle.md:231-243`（逐語要旨）:
> TTY での初回実行は**質問ではなく検出**から始まる（PATH 上のハーネス CLI、プロジェクト状態、
> ローカルの AWS 資格情報とリージョン、非対話フックランタイム）。
>
> | 検出結果 | 最初に出るもの |
> |---|---|
> | ハーネス **1 種** | 名前を示したうえで **3 択**（推奨既定 / 6 ステップのカスタマイズ / 何も書かず終了） |
> | ハーネス **複数** | **番号付きのハーネスピッカーが先**。その後に上記へ進む |
> | ハーネス **なし** | 既定なしの完全なピッカー |
>
> **「推奨既定」を選んだ場合、6 問は出ない**（選ばれるバンドルは選択肢の行に表示される）。

カスタマイズを選んだ場合の 6 ステップは次のとおり。

1. Harness（7 種から選択）
2. Model provider（`amazon-bedrock` / `other`。bedrock なら region と profile）
3. Model effort preset（`balanced` / `thorough` / `minimal`）
4. Plugins（`all installed` / `none optional` / `choose`）
5. MCP servers（`on` / `off`）
6. 記録先（project 共有 / project 個人 / machine）

最後に `Apply? [Y/n]` のゲートがあり、**それより前にはファイルを 1 つも書かない**。
2.8.1 が直したのは、このゲートで Enter を押すとキャンセル扱いになる不具合である
（ただし 17.3 のとおり 2.8.1 は未リリース）。

`aidlc config` は `dist/<harness>/` の手動コピーの置き換えであり、
ハーネスツリー・`aidlc/` ワークスペースシェル・ルート統合・投影スタンプ・所有権ベースラインを作る。
ワークフローの intent は作らない。

セクション指定でピンポイントに設定もできる:
`aidlc config models` / `runtime` / `providers` / `trust` / `flags` / `project`。

> **⚠ 名前が衝突している。** セッション内のスラッシュコマンド `/aidlc config`（6 章 6.5 参照）と、
> ここで説明したネイティブ CLI の `aidlc config` は**別物**である。
> 前者はワークフローの depth / test-strategy / review を扱い、後者はプロジェクトの導入・設定を扱う。

---

## 17.6 Claude Code 出荷設定からモデル固定と無制限 `Bash` 許可が消えた

> **⚠ この変更が入ったのは 2.7.2（`12b8d6e0` / #756）であり、2.8.1 ではない。**
> `git log -S` で削除コミットを特定した結果である（`model: opus[1m]` / `effortLevel: xhigh` /
> 無制限 `Bash` / `templated` tier のいずれも同じコミット）。
> **2.7.2 は公開済みの v2.8.0 に含まれる。**
> したがって **2.8.0 を使っている時点で、この変更はすでに効いている。**
> 「2.8.1（未リリース）の変更」と読むと、まだ pin が効いていると誤解する。

`harness/claude/settings.json` の差分（逐語、呼び出し文字列の書き換えを除く）:

```diff
-      "Bash(bun \"$CLAUDE_PROJECT_DIR/.claude/tools/\"*)",
-      "Bash",
+      "Bash(bun .claude/tools/*)",
-  "model": "opus[1m]",
-  "effortLevel": "xhigh",
```

- **自動許可リストから無制限の `Bash` エントリが削除された。**
  これは「拒否リストが強化された」のではなく、**出荷設定で自動承認される範囲が狭まった**ということである。
  **ただし最終的にプロンプトが出るかどうかは、利用者側・組織側の設定にも依存する。**
  本章が実測したのは出荷 `settings.json` の内容までである。
- 出荷時のモデル固定（`opus[1m]`）と effort 固定（`xhigh`）が削除された。

あわせて `core/tools/aidlc-tiers.ts` の `templated` tier の投影が変わった:

```diff
   templated: {
-    claude: { model: "sonnet", effort: "medium" },
-    codex: { model: "openai.gpt-5.6-terra", effort: "medium" },
-    opencode: { model: "amazon-bedrock/global.anthropic.claude-sonnet-4-6", variant: "medium" },
+    claude: { model: "inherit", effort: null },
+    codex: { model: null, effort: null },
+    opencode: { model: null, variant: null },
```

`templated` は「方法論が知識側に既にあり、出力が定型に寄る作業」
（デリバリ計画・CI/CD 設定・runbook 等）に付く tier である。
これが**固定モデルからセッション継承へ変わった**。
`templated.kiro` は 2 版とも `{ model: null }` で、Kiro には元から投影されていない。

上流のコメントは、`balanced` と `templated` が同一投影だった状態を解消し、
`templated` を「別のダイヤルを持つグループのまま、出荷既定は継承」に置いた、と説明している。

> **⚠ 削除されたのはトップレベルの pin と `templated` tier だけである。**
> **`balanced` tier は 2.7.0 から変わっていない**（実測: 2 版でバイト同一）。
> いまも次を固定している。
>
> | ハーネス | `balanced` の投影 |
> |---|---|
> | Claude Code | `model: sonnet` / `effort: medium` |
> | Codex CLI | `openai.gpt-5.6-terra` / `medium` |
> | opencode | `amazon-bedrock/global.anthropic.claude-sonnet-4-6` / `medium` |
> | Cursor / Kiro / Copilot | 固定なし（セッション継承） |
>
> 上流のコメントは **`balanced` を「レビュア tier」**と説明し、
> **「レビュー専用エージェント 2 体だけが持ち、他は持たない」**と明記している。
> **したがって「モデル固定が全面的に消えた」わけではない。**
> **Claude Code / Codex CLI / opencode のレビュー工程は今も固定モデルで走り、
> Cursor / Kiro / Copilot はセッション設定を継承する。**


---

## 17.7 リリースの検証経路が作り直された

`.github/workflows/release.yml` が **58 行から 577 行へ全面的に書き換えられた**。
旧 v1 向けの dispatch 手順は新設の `dispatch-v1-release.yml`（53 行）へ退避している。

タグ push で起動し、`validate` ジョブが 4 条件を強制する。

1. タグが `v<major>.<minor>.<patch>` 形式であること
2. タグが チェックアウトした commit を指していること
3. その commit が **`main` の祖先**であること
4. **`AIDLC_VERSION` とタグが一致すること**

以降 `verify` → `native-smoke`（5 ランナーのマトリクス）→ `build`（7 ターゲット）→
`musl-smoke`（Alpine コンテナ）→ `stage-release` → Windows / POSIX のライフサイクル検証 →
`publish` → `release` と続く。
`publish` は `actions/attest-build-provenance@v2` で署名し、
オフライン検証用に `aidlc-release.intoto.jsonl` を資産へ同梱する。
最終の `release` ジョブは `environment: release` ゲート付きである。

クライアント側（`scripts/install.sh`）は、**`gh` CLI があれば** attestation を検証し、
無ければ SHA-256 チェックサム検証のみで続行する（`gh` は必須ではない）。
ダウンロード URL は HTTPS 必須で、資格情報・クエリ・フラグメントを含む URL は拒否される。

### ⚠ リリース時のフルテストスイートが無効化されたまま

`.github/workflows/release.yml:61-63`（逐語）:

```yaml
      # TEMPORARY: the release tag already passed the PR CI suite. Keep this
      # duplicate full-suite run disabled while the first native release ships.
      # - run: bash tests/run-tests.sh --ci --parallel 8
```

初回ネイティブリリースの出荷中に限った暫定措置と明記されているが、
**HEAD `c03f9e28` の時点でも解除されていない**。
現状、タグ push 時のフルスイート再実行は行われていない
（PR 時の CI は動いているので、テストが一切走っていないわけではない）。

---

## 17.8 roadmap は更新されていない

`docs/roadmap.md` は 2 版の間で**変更が無い**（`roadmap.html` は `docs/roadmap.html` への
リダイレクトスタブで、こちらも変更なし）。

その結果、上流 roadmap の記述が現状と食い違っている。

| roadmap の記述 | 実際（HEAD） |
|---|---|
| 「The current v2 version is **2.6.124**」 | `AIDLC_VERSION` = **2.8.1** |
| 「`origin/main` tip `82d2e304`」 | `c03f9e28` |
| 「native distribution … **remains under review in #756**」 | **#756 はマージ済み**（`12b8d6e0` として 2.7.2 に着地） |
| 「**no public v2 native release exists yet**」 | **v2.8.0 が資産 13 件付きで公開済み** |

16 章では「上流 roadmap の記載が 1 版古い」と記録した。**今回は 5 版古い**
（2.6.124 の後に 2.7.0 / 2.7.1 / 2.7.2 / 2.8.0 / 2.8.1 が出ている）。
上流の roadmap を現在値の出典に使わないこと。

進行中 PR のうち、本ノートの追跡対象は変わっていない。

- **#968**（Devin CLI / Desktop ハーネス）は**依然 OPEN**（最終更新 2026-09-02）。
  マージされればハーネス 8 種目になる。
- あわせて **#996「feat: Devin Harness」も別に OPEN** である。
  Devin ハーネスの PR が 2 本並存している状態で、どちらが採られるかは上流の判断待ちである。
- #1025「chore: upgrade bun to 1.4.0」は OPEN。`main` は全ワークフローで bun 1.3.14 に固定されている。

> 上流の open PR は `gh pr list --limit 40` で 40 件返ったため、
> 41 件目以降の有無は確認していない。

---

## 17.9 リポジトリ運用ファイルの変化

| 変更 | 内容 |
|---|---|
| `docs/rfcs/` の削除 | 4 ファイル −1,117 行。`.gitignore` に `/docs/rfcs/` を追加 |
| 新規ガイド | `docs/guide/18-install-and-lifecycle.md`（+944 行）— インストーラのライフサイクル全書 |
| 新規リファレンス | `docs/reference/19-supply-chain-security.md`（+101 行）— リリースのサプライチェーン |
| 新規スクリプト | `install.sh` / `install.ps1` / `package-release.ts` / `verify-release.ts` |
| `package.json` | `check` が `package.ts && package.ts --check` の 2 段に |
| `ci.yml` | `dist/` が gitignore されたため、各ジョブで `bun scripts/package.ts` を先に実行 |
| `CONTRIBUTING.md` / `AGENTS.md` | `dist/` の位置づけを「コミットする生成物」→「**コミットしないローカル生成物**」へ |

`docs/rfcs/` 削除の理由は上流のコミットメッセージに明記されている（逐語）:

> RFC drafts and implementation plans are working notes, not documentation.
> They have no permanent home under `docs/` … Design proposals belong in GitHub issues
> via the RFC issue template; local drafts live in the gitignored `tmp/`.

`package.json` の `check` が 2 段になったのは、検査の性格が変わったためである。
従来は「コミット済み `dist/` と再生成結果のバイト一致（drift guard）」を見ていたが、
`dist/` がコミットされなくなったので、
**独立した一時ルートで 2 回ビルドして結果が一致するか（determinism guard）**を見る形になった。

> **⚠ 本ノートの 12 章が参照している `docs/rfcs/` は、もう取得できない。**
> 12.9 節が逐語引用している Markdown 2 本は、当時の clone にしか存在しない。
> 引用そのものは当時の記録として有効だが、**読者が同じ URL で追検証することはできない**。

---

## 17.10 全 4 版の一覧

| 版 | CHANGELOG 日付 | CHANGELOG 投入コミット | 要旨 | 本章での扱い |
|---|---|---|---|---|
| 2.8.1 | 2026-09-08 | `c03f9e28`（#1054） | ウィザードの Enter 受理、umask 非依存の same-version update。**未リリース** | [17.3](#173--281-は-changelog-にあるがリリースされていない) |
| 2.8.0 | 2026-09-08 | `1c64d9a6`（#1046） | 2.7.x のロールアップ。**CHANGELOG エントリとしては** 2.7.2 から挙動変更なし | [17.3](#173--281-は-changelog-にあるがリリースされていない) |
| 2.7.2 | 2026-09-07 | `12b8d6e0`（#756） | ネイティブ配布・config ポリシー・リリース基盤。**本章の主題** | [17.1](#171-いちばん大きい変更は-dist-の消滅) / [17.5](#175-設定に階層ができた--ガードレールを恒久的に記録できる) / [17.7](#177-リリースの検証経路が作り直された) |
| 2.7.1 | 2026-09-01 | `a277af21`（#997） | solo ワークフローの Plan Approval デッドロック修正 | [17.4](#174-271--solo-ワークフローで-plan-approval-がデッドロックしていた) |

> **⚠ 「CHANGELOG 投入コミット」と「タグが指すコミット」は別である。**
> 2.8.0 の CHANGELOG を入れたのは `1c64d9a6` だが、**`v2.8.0` タグが指すのは `0d399dd8`** である
> （その間に #1048 / #1049 / #1050 の 3 件が入っている。いずれも `AIDLC_VERSION` は 2.8.0 のまま）。
> **リリースされた 2.8.0 の実体は `0d399dd8` 時点のツリーである。**
> **したがって「2.8.0 は 2.7.2 から挙動変更なし」は CHANGELOG エントリについての話であり、
> リリースされたタグには #1049（glibc 判定修正）と #1050（provenance 受理条件）の挙動変更が含まれる。**

> **⚠ CHANGELOG の日付と実際のマージ日は一致しない。**
> 2.7.1 は CHANGELOG で 2026-09-01 だが、`a277af21` のコミット日は 2026-09-02 である。
> 2.7.2 も CHANGELOG は 2026-09-07、`12b8d6e0` は 2026-09-08 である。
> 「いつから使えたか」を判断するときは CHANGELOG の日付ではなくタグを見ること。

11 コミットのうち、**CHANGELOG.md を変更したのは 4 件だけ**である
（`a277af21` / `12b8d6e0` / `1c64d9a6` / `c03f9e28`）。
残る **7 件は CHANGELOG に対応するエントリを持たない**。

| コミット | 内容 |
|---|---|
| `c46f500a`（#1029） | `docs/rfcs` の削除 |
| `22ed2d10`（#1015） | プラグイン compose アサーションの Windows 移植性 |
| `e7689885`（#1014） | Windows で `safe.directory` を保持するテスト |
| `f9f48945`（#1048） | Windows リリーススモークを通すための暫定措置。**これが 17.7 のコメントアウトを入れたコミットである**（解除したのではない） |
| `eb4cfcfc`（#1049） | コンパイル済み Linux バイナリでの glibc 検出修正 |
| `0d399dd8`（#1050） | provenance 検証で source-ref のみの形式も受理。**`v2.8.0` タグはこのコミットを指す** |
| `78a6879e`（#1053） | README / getting-started の簡素化 |

`eb4cfcfc` は、Bun でコンパイルしたバイナリが `process.report` を持たない場合があるのに
その不在だけで musl と判定していた不具合の修正である。
musl ローダの実在確認を AND 条件に加えて直している。

---

## 17.11 過去章への影響 — 手順の全面改訂

**今回、過去章の事実誤りは見つからなかった。** 16 章までの測定値は、**その時点の測定記録としてすべて有効**である。

> **⚠ ただし「記録として有効」と「現在値として引用してよい」は別である。**
> **`core/tools/*.ts` の 51 は 2.8.1 では 69 になっている。**
> 16 章までが記録する 51 を**現在値として引用してはならない**。
> それ以外の中核メトリクスは 2.8.1 でも同値である。

ただし今回は 16 章のときより影響が広い。
**`dist/` を前提にした「実行される手順」が、公開ノート全体に散らばっていたためである。**

| 区分 | 扱い | 対象 |
|---|---|---|
| 日付付きの測定記録 | **書き換えない** | 10 章の `対象:` 行、11〜16 章の `基準:` 行と本文、`CONVERSATION_LOG.md` と `REVIEW-8AI.md` の過去追記 |
| 実行される手順 | **書き換えた** | README のクイックスタート、6 章のインストール手順全編、9 章・SOURCES の clone コマンド |
| 現在の参照先の宣言 | **書き換えた** | README 冒頭、2 章のゾーン表と構成図、SOURCES の版チェーン、`docs/TRACEABILITY.md`、`docs/DEVELOPMENT_RULES.md` |
| 測定方法の宣言 | **書き換えた** | ハーネス数の測り方（`dist/` 直下 → `harness/` 直下） |

過去章の `基準:` 行の HEAD SHA は、今回も全件が `main` から到達可能である。

| 章 | `基準:` 行の HEAD | 現状 |
|---|---|---|
| 11 章 | `2ce654d1` | `main` から到達可 |
| 12 章 | `4569754e` | 同上 |
| 13 章 | `71d9a9e0` | 同上 |
| 14 章 | `840ba653` | 同上 |
| 15 章 | `2fbee12f` | 同上 |
| 16 章 | `96b11d39` | 同上 |

6 件とも `git merge-base --is-ancestor <SHA> origin/main` で確認済みである。

> **⚠ 過去章の `dist/` 記述を「誤り」として直さないこと。**
> 11 章から 16 章が記録している `dist/<harness>/` の再コピー手順は、
> **その時点では上流が公式に指示していた手順**である。記録として正しい。
> 変わったのは上流の配布方式であって、当時の測定ではない。

同じ理由で、**`dist/` のファイル数を使った測定**（6 章の
「`dist/` の変更ファイル数を『開発量』と読まないこと」の節など）は、
測定当時の記録としては有効だが、**今後は同じ手法で再現できない**。
その旨を各所に注記した。

---

## 17.12 本ノートの限界

- 本章は**上流リポジトリの差分の静的読解**に基づく。実機での再現は行っていない。
- **ネイティブインストーラを実際に走らせていない。** `install.sh` / `install.ps1` の
  記述はスクリプトとドキュメントの読解によるもので、導入結果を確認したものではない。
- 上流のテストスイートは実行していない。
- `dist/` / `dist-release/` はワーキングツリーに存在しないため、
  生成物の実バイトを検証していない。パッケージャのコードからの読解である。
- `aidlc config` がプラグインの compose フックを自動実行するかは**未確認**である。
  上流ドキュメントは「プラグイン合成ファイルと記録済みステージ寄与を保持してグラフを再生成する」
  と書くが、compose フックの自動実行は明記していない。
  Claude / Codex / Cursor / Kiro IDE は SessionStart フックで自己修復し、
  **Kiro CLI は明示的な `/aidlc plugin sync` が要る**、という記述は維持されている。
- Windows ARM64 でのネイティブバイナリの挙動は上流に記述が無く、**未確認**である。
- **v2.8.0 の Copilot / Cursor フック不具合（17.3）は、上流のコミット本文と CHANGELOG の読解による。**
  **実機で再現していない。** 該当ハーネスを使う場合は自環境で確認すること。
- 2.8.1 の修正は未リリースのため、**公開されているネイティブ資産では確認できない**。
  SHA `c03f9e28` を checkout してソースから生成すれば検証自体は可能だが、**本調査では行っていない**。
- 「2.8.1 調査で残った未確認事項」は
  [docs/REMAINING_TASKS.md](./docs/REMAINING_TASKS.md) に列挙した。
