# 06. ハーネスと導入

> **⚠ 導入方法が根本的に変わった（2026-09-08）。**
> **実装版としては 2.7.2、利用者が入手できるリリースとしては v2.8.0 が最初である**
> （`v2.7.1` / `v2.7.2` のタグは存在しない。→ [17.1](./17-release-impact-2801.md#171-いちばん大きい変更は-dist-の消滅)）。
> **本章で単に「2.8.x」と書いている箇所はリリース観点である。**
> 上流のコミットや CHANGELOG を追う場合は **2.7.2** を境界として見ること。
>
> **📌 続報（2026-09-17）—— 上記の 🔴 は解消済み。**
> `52da70ad` を含む **`v2.8.1` が 2026-09-09 20:12 UTC にリリース**された（本ノートの測定の数時間後）。
> その後 **v2.8.2（09-11）と v2.9.0（09-15、現 Latest）** が公開されている。
> **Copilot / Cursor を使う場合も、v2.8.1 以降を導入すれば不具合は起きない。**
> **17 章が案内するソース生成の暫定回避策は不要である。**
> 2.8.1 → 2.9.0 の差分は [18 章](./18-release-impact-290.md) で扱う。

> **🔴 v2.9.0 へ更新するときは、プロジェクトごとに `aidlc config --yes` が要る。**
> CHANGELOG 逐語:
> ```
> run `aidlc update`, then run `aidlc config --yes` in each project to refresh its harness runtime.
> ```
> 2.8.x までの「何もしなくてよい」とは違う（→ [18.5](./18-release-impact-290.md)）。

> **🔴 現 Latest の v2.9.0 に、センサー・フック系の不具合が 3 件残っている（①②はネイティブ導入限定、③は Bun 経路でもフックの PATH 次第で起こる）（2026-09-23 時点）。**
> **①② は `bun` 実行では再現しない。安定版（Latest `v2.9.0`）には未収録で、preview `v2.9.1-preview.20260920.1` 以降には収録済み**（2026-09-23 実測）。
> **③ は `bun` 実行でも起こりうる（フックの PATH 上に `bun` が無い場合）。修正はどの版にも未収録**（preview を含む）。
>
> | 症状 | 影響 |
> |---|---|
> | **ゲートの Review brief が動かない**（#1070） | `loadDelegate` に `review-brief` の分岐が無く `does not export main(argv)` で終わる。**2.8.0 以降の全ネイティブリリースが該当** |
> | **ゲートのセンサーが発火しない**（#1166） | `aidlc sensor …` が `unknown command 'sensor'` になる。**`blocking` はゲートを拒否し、`advisory` は黙って捨てられる** |
> | **3 つのコアフックが `bun` を直接名指し**（#1249） | ネイティブには Bun が無いのに spawn が試みられる。**Stop フック・runtime-graph 再構築・Write 契機センサーが無言で死ぬ。doctor も警告できない** |
>
> **センサーを使う予定があるなら、ネイティブ v2.9.0 では期待どおりに動かない。**
> **✅ 回避策: そのプロジェクトだけ手動コピー経路（`aidlc-copy-runtime-2.9.0.tar.gz`、Bun 前提）で導入する。**
> **①② は回避できる。③ は `bun` が非対話フックの PATH 上にある場合に限り回避できる。**
> **⚠ preview に切り替えても ③ は直らない。**
> 詳細と根拠は [18.6](./18-release-impact-290.md)。
>
> **⚠ v2.8.0 に限り、GitHub Copilot と Cursor のフックが動作しない。**
> Copilot は全イベントでクラッシュ、Cursor は**全ツール呼び出しがブロックされる**。
> **修正 `52da70ad` は v2.8.1 に含まれるため、v2.8.1 以降では起きない。**
> **v2.8.0 を使っている場合のみ、v2.8.1 以降（推奨は現 Latest の v2.9.0）へ更新する。**
> **根拠は同コミットの本文とタグ包含判定（実機再現はしていない）。**
> 上流リポジトリから **`dist/` ディレクトリが削除された**。
> 導入はネイティブインストーラ（`install.sh` / `install.ps1`）で `aidlc` コマンドを入れ、
> プロジェクトごとに `aidlc config --harness <name>` を実行する形になった。
> **この経路（ネイティブ導入）なら Bun / Node.js は不要である。**
> **⚠ 手動コピー経路は別で、v2.9.0 以降は逆に Bun が必須・ネイティブ `aidlc` が不要になった**（→ 6.3）。
> 本章の版ごとのアップグレード記録に出てくる「`dist/<harness>/` の再コピー」は、
> **その版の時点で上流が指示していた操作の記録**であり、現在の手順ではない。
> 経緯は [17-release-impact-2801.md](./17-release-impact-2801.md) を参照。

## 6.1 対応ハーネス（2.x）

| Harness | 最低バージョン目安 | 導入コマンド（v2.8.0 以降。v2.9.0 でも同じ） | 起動 |
|---------|-------------------|--------|------|
| **Claude Code** | 最新推奨 | `aidlc config --harness claude` | `/aidlc` |
| **Kiro IDE** | hooks v2 対応含む | `aidlc config --harness kiro-ide` | `/aidlc` |
| **Kiro CLI** | ≥ 2.6 | `aidlc config --harness kiro` | `/aidlc` |
| **Codex CLI** | ≥ **0.145.0** | `aidlc config --harness codex` | `$aidlc` |
| **Cursor** | 明示の最低版なし（上流は cursor-agent **2026.07** で検証） | `aidlc config --harness cursor` | `/aidlc` |
| **opencode** | ≥ 1.17 | `aidlc config --harness opencode` | `/aidlc` |
| **GitHub Copilot** | CLI ≥ **1.0.74** / VS Code ≥ **1.130**（2026-07-22 リリース） | `aidlc config --harness copilot` | `/aidlc` |

**計 7 種**（`ls harness/` = claude / codex / copilot / cursor / kiro / kiro-ide / opencode）。
決定論エンジン（state machine・audit・並列の審判）はハーネス横断で同一。違うのはシェル（skills/hooks の載せ方）。

> **Cursor は 2.5.63 で追加**（IDE と CLI `agent` の両方を 1 つの `.cursor/` で兼ねる）。
> **2.7.1 以前は 7 種のうち Cursor だけ導入形態が違い**、他の 6 種が `dist/<harness>/` を `cp` するのに対し
> Cursor は同梱インストーラ `bun dist/cursor/install.ts <project>` を実行する形だった。
> **2.7.2 以降（リリースとしては v2.8.0 以降）は 7 種すべてが `aidlc config --harness <name>` に統一され、この非対称は解消した。**
> 導入処理の性質は引き継がれている —— プロジェクト所有ファイルとの衝突を拒否し、
> `.cursor/.gitignore` と既存の method memory を保全、
> `.cursor/hooks.json` と `.cursor/cli.json` は配列を構造マージ、`AGENTS.md` と `.gitignore` には
> AI-DLC 用のマーク付き区画を追記する。再実行はアップグレードとして働き、active-space ポインタは保たれる。

> **GitHub Copilot は 2.5.60 で追加**（CLI と VS Code agent mode の両方を 1 つの dist でカバー）。
> 他ハーネスと違い **folder trust が前提**で、プロジェクトの絶対パスが
> `~/.copilot/config.json` の `trustedFolders` に無いと**リポジトリフックが 1 本も動かない**。
> ヘッドレス（`copilot -p`）では `GITHUB_COPILOT_PROMPT_MODE_REPO_HOOKS=1` も要る。
> VS Code は `SessionEnd` を持たないため、次回 `SessionStart` で前セッションを事後整合する。

---

## 6.2 前提（共通とハーネス別）

### 全ハーネス共通

1. **ネイティブ導入なら bun は要らない**（2.7.2 以降。リリースは v2.8.0 以降）。配布されるのは単一バイナリで、
   Bun / Node.js のいずれも前提にしない。
   **⚠ ただし v2.9.0 以降、手動コピー経路を選ぶ場合は逆に Bun が必須でネイティブ `aidlc` が不要になった**（下記 4.）
2. 推奨モデル: **Claude Opus 4.8**（公式 README。Kiro では有料プランが必要な場合あり）

```bash
curl -fsSL https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.sh | sh
```

> **bun が要るのは、本章に出てくる範囲では次の 4 つの場合である。**
> 1. 上流リポジトリを clone してソースから生成する場合（`bun scripts/package.ts`。開発者向け経路。→ [6.3](#63-インストール要点)）
> 2. **`codekb-scope-diff` を使う場合** —— 上流はこの診断コマンドを今も
>    `bun <harness-dir>/tools/aidlc-utility.ts codekb-scope-diff` の形でしか案内しておらず、
>    **ネイティブ導入だけの環境には公式な実行手段が無い**（→ [6.5](#65-よく使うコマンド)）。
>    **Reverse Engineering の再実行前チェックを使うなら bun が要る。**
> 3. **Codex の trust エントリをチェックアウトから生成する場合**
>    （`bun install --frozen-lockfile` → `bun scripts/package.ts codex trust --project <path>`。
>    → [6.3 の Codex CLI 節](#codex-cli)）。**TUI の "Trust all" を使うなら bun は要らない。**
> 4. **手動コピー経路で導入・更新する場合（v2.9.0 以降）** —— `aidlc-copy-runtime-X.Y.Z.tar.gz` は
>    **Bun 前提の投影**であり、**Bun が無いとフックもツールも起動しない**。
>    上流ガイド逐語: `Bun is the runtime prerequisite; the native aidlc executable is not required`
>    （→ [6.3 の手動コピー節](#手動でファイルを置きたい場合)）。
>    **2.8.x までは逆に、手動コピーでもネイティブ `aidlc` が必須だった。**
> その場合は非対話シェルからも見える PATH に入れること
> （zsh なら `~/.zshenv` にも `BUN_INSTALL` / `PATH` を書く必要がある場合あり）。
>
> ```bash
> curl -fsSL https://bun.sh/install | bash
> git clone --branch main https://github.com/awslabs/aidlc-workflows.git
> cd aidlc-workflows
> ```

### 6.2.1 実行環境の前提: 単一ローカルファイルシステム（2.6.51 以降）

2.6.51 の継続カーソル（→ [6.4](#64-アップグレードでハーネスごとにすること262--2655)）は
「同時に 1 プロセスだけが勝つ（exactly one winner）」ことを、**単一のローカルファイルシステム**が備える
プリミティブに依存して実現している。上流が挙げる要件は、プロセス間で一貫した可視性・排他的なディレクトリ作成・
安定した通常ファイル読み取り・**マーカーとロックのパスに対する同一 FS 内のアトミックな `rename`** の 4 つで、
上流はこれを `docs/reference/06-hooks-and-tools.md` に明記したうえで、
**これらのプリミティブを尊重しない NFS / SMB / FUSE / オブジェクト同期フォルダを unsupported と宣言している**。

**したがって、記録ディレクトリの置き場所が初めて可用性の条件になった。**
ただし「ネットワーク共有や同期フォルダなら必ず駄目」ではない。上流が unsupported としているのは
**上記のプリミティブを尊重しない**ファイルシステムであり、実装や構成によっては満たす場合もある。
**満たさない構成では、従来 fail-open で動いていたものが「作業ディレクティブが 1 つも出ない」状態になりうる。**
自分の構成が該当するかは実際に確認すること。

> The directive could not be published, so **no work directive was issued**.
> Retry the command; if coordination remains busy, run `/aidlc --doctor`.

なお実装は候補ファイルにも親ディレクトリにも **`fsync` を行わない**。
保証されるのは「プロセス間で勝者がちょうど 1 つ」であって、**突然の電源断に対する耐久性は保証の外**である。

> ネットワーク FS 上で実際にどのタイミングでどのエラーが出るかは**本ノートでは未検証**
> （上流の unsupported 宣言を確認したのみ）。

### モデル／認証（出荷既定 ≠ 全ハーネス必須）

公式 README は Bedrock の準備に触れるが、**「どのハーネスでも Bedrock アカウント必須」ではない**。

| ハーネス | 目安 |
|----------|------|
| **Claude Code** | 出荷設定は Bedrock。モデルアクセス有効化 + AWS SDK 資格情報が実質必要 |
| **Codex CLI** | 出荷 `config.toml` は Bedrock ブロック。OpenAI 認証等への代替はガイド参照 |
| **GitHub Copilot** | GitHub Copilot の認証をそのまま使う。**加えて folder trust が必須**（`~/.copilot/config.json` の `trustedFolders`）。ヘッドレスは `GITHUB_COPILOT_PROMPT_MODE_REPO_HOOKS=1` |
| **Cursor** | Cursor 自身のサインイン。**出荷ペルソナはモデルを一切 pin していない**ため、全エージェントがセッションのモデルを継承する。名前付きモデル（`--model` 等）は**有料プランが必要**で、Free は `Auto` のみ |
| **Kiro IDE / CLI** | Kiro サインイン + セッションで選ぶモデル（≥2.6 等は公式 README の Kiro CLI 要件） |
| **opencode** | グローバル opencode 設定のプロバイダ／モデル |

---

## 6.3 インストール要点

> **⚠ 2.7.2 以降（リリースは v2.8.0 以降）で導入方法が変わった。** `cp -R dist/<harness>/` はもう上流の公式手順ではない。
> 詳細は [17.1](./17-release-impact-2801.md#171-いちばん大きい変更は-dist-の消滅)。

### 全ハーネス共通の導入手順

**ネイティブ `aidlc` を 1 回入れれば、7 ハーネス分のランタイムがすべて含まれる。**
プロジェクトごとに `--harness` でどの面を作るかを選ぶ。

```bash
# 1. ネイティブ aidlc を導入（bun / Node.js は不要）
curl -fsSL https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.sh | sh
#    Windows PowerShell:
#    irm https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.ps1 | iex

# 2. PATH を通す（インストーラは子シェルで走るため、親シェルには反映されない）
#    `aidlc: command not found` になる場合はこれ。インストーラが表示する手順が正
export PATH="$HOME/.local/bin:$PATH"   # $AIDLC_BIN_DIR を変えた場合はそのパス
aidlc version                          # 版が出れば導入成功

# 3. プロジェクトに面を作る
cd your-project
aidlc config --harness claude     # claude / codex / copilot / cursor / kiro / kiro-ide / opencode
aidlc doctor
```

導入先は Unix が `~/.local/bin`（`$AIDLC_BIN_DIR`）と
`${XDG_DATA_HOME:-~/.local/share}/aidlc`（`$AIDLC_INSTALL_ROOT`）、
Windows が `%LOCALAPPDATA%\aidlc`（コマンド名は `aidlc.cmd`）。
root での実行は拒否される。Homebrew / Nix 管理の既存 `aidlc` があれば置き換えず譲る。

`aidlc config` が作るもの: 選んだハーネスのツリー、`aidlc/` ワークスペースシェル、
ルート統合、投影スタンプ、所有権ベースライン。**ワークフローの intent は作らない。**

> **⚠ ワンライナーはダウンロードしたスクリプトを直接パイプ実行する。**
> 実行前に内容を確認したい場合は 2 段階の手順を使う。上流にも併存している。
>
> ```bash
> tmp="$(mktemp -d)"
> curl -fsSL https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.sh -o "$tmp/install.sh"
> less "$tmp/install.sh"          # ← ここで内容を確認する（この 1 行が 2 段階にする理由）
> sh "$tmp/install.sh"
> rm -rf "$tmp"
> ```
>
> **確認を挟まずに `sh` するなら、パイプ直結版と実質同じである。**

`install.sh` のオプション（**v2.9.0 実測・逐語**）:

```
Usage: install.sh [--version <x.y.z|x.y.z-preview.YYYYMMDD.N>] [--from <dir>] [--offline] [--profile <startup-file>] [--json|--quiet] [--no-color] [--yes]
```

> **基準 `c03f9e28` では `--version <x.y.z>` のみだった。**
> v2.9.0 で **preview id の構文が追加**され、`--version 2.9.1-preview.20260915.1` のような
> 指定が通るようになっている（→ [18.13](./18-release-impact-290.md)）。

**Windows（`install.ps1`）も同じ版指定ができる。** パラメータ名は `-Version` で、
文法は `install.sh --version` と同じ（stable の `x.y.z` と preview id を受理する）:

```powershell
# 版を固定して導入する場合（既定は latest）
.\install.ps1 -Version 2.9.0
```

> **⚠ 版を固定するなら `--version 2.9.0`（現 Latest）を使う。**
> `v2.8.1` / `v2.8.2` / `v2.9.0` はいずれも実在するタグである
> （「`v2.8.1` タグは存在しない」という旧記述は 2026-09-09 時点の話で、現在は誤り）。

### 手動でファイルを置きたい場合

> **🔴 v2.9.0 で手動コピー経路の前提が逆転した。ネイティブ `aidlc` バイナリは不要になり、代わりに Bun が要る。**
>
> | | 2.8.x まで | **v2.9.0 以降** |
> |---|---|---|
> | 取得するアセット | `aidlc-runtime-X.Y.Z.tar.gz` | **`aidlc-copy-runtime-X.Y.Z.tar.gz`** |
> | 前提 | **ネイティブ `aidlc` が必須** | **Bun が必要。ネイティブ `aidlc` は不要** |
> | 中身の出どころ | `dist-release/<harness>`（ネイティブ投影） | **`dist/<harness>`（Bun 前提の投影）** |
>
> 上流 `README.md` 逐語（2.9.0）:
> ```
> Cannot install a native executable, or prefer to manage the project files
> manually? Install Bun, download `aidlc-copy-runtime-X.Y.Z.tar.gz` from the
> release, and copy the complete `runtime/<harness>/` directory into your
> project. This path does not require the native `aidlc` command.
> ```
> 2.8.x までは逆に「**Install the matching native `aidlc` command**」と書かれていた。
>
> `scripts/package-release.ts` でも、copy 用アーカイブは `dist/<harness>`、
> ネイティブ用は `dist-release/<harness>` から作られている。
> **つまり手動コピー経路は「ネイティブ導入の代替」になった。**
> **これは上流 README と `package-release.ts` の読解であり、本調査では実機で確かめていない。**
> **📌 かつてここに「Copilot / Cursor のフック不具合を避けるソース生成経路」を記していたが、
> その回避策はもう不要である。** 不具合は `52da70ad` で修正され、**v2.8.1 以降に含まれる**。
> **素直に v2.8.1 以降（推奨は現 Latest の v2.9.0）を導入すればよい。**

**v2.9.0 以降**は、Bun を導入したうえで `aidlc-copy-runtime-X.Y.Z.tar.gz` を展開し、
**`runtime/<harness>/` をディレクトリごと**プロジェクトへコピーする。
これがプロジェクト内ファイルを手で管理する場合の正規経路である。
**2.8.x までは、ネイティブ `aidlc` を導入したうえで `aidlc-runtime-X.Y.Z.tar.gz` を使う手順だった。**

#### 上流が案内している手順（逐語ベース）

上流ガイド `docs/guide/18-install-and-lifecycle.md` の Copy Channel 節は、
**署名検証とチェックサム照合を手順に含めている**。社内導入で供給元検証が要件なら、ここを落とさないこと。

```bash
set -eu   # 検証に失敗したら展開へ進まない（上流の原文には無いので足している）
tag=vX.Y.Z
tmp="$(mktemp -d)"
runtime_asset="aidlc-copy-runtime-${tag#v}.tar.gz"
runtime_checksum="${runtime_asset}.sha256"
source_repo="${AIDLC_RELEASE_REPOSITORY:-awslabs/aidlc-workflows}"
release_workflow="${AIDLC_RELEASE_WORKFLOW:-$source_repo/.github/workflows/release.yml}"

gh release download "$tag" --repo "$source_repo" --dir "$tmp" \
  --pattern "$runtime_asset" \
  --pattern "$runtime_checksum" \
  --pattern aidlc-release.intoto.jsonl

gh attestation verify "$tmp/$runtime_asset" \
  --bundle "$tmp/aidlc-release.intoto.jsonl" \
  --repo "$source_repo" \
  --signer-workflow "$release_workflow" \
  --source-ref "refs/tags/$tag"

# 古い macOS には sha256sum が無い。その場合は同梱の shasum を使う
if command -v sha256sum >/dev/null 2>&1; then
  (cd "$tmp" && sha256sum -c "$runtime_checksum")
else
  (cd "$tmp" && shasum -a 256 -c "$runtime_checksum")
fi
tar -xzf "$tmp/$runtime_asset" -C "$tmp"
RUNTIME_ROOT="$tmp/runtime"
```

> **⚠ 上流の原文には `set -e` が無い。** そのまま貼ると、`gh attestation verify` や
> チェックサム照合が失敗しても `tar -xzf` まで進んでしまう。上のブロックでは冒頭に `set -eu` を足した。
> **また上流は `sha256sum` を前提にしているが、古い macOS には無い**ため、`shasum -a 256 -c` への分岐を足した。

> **⚠ copy 用アーカイブは `version.json` / `checksums.txt` の外にある。**
> 上流逐語: `The copy archive stays outside version.json and checksums.txt so existing
> 2.8.x native clients can continue to parse release metadata and self-update.`
> **検証は専用の `.sha256` サイドカーと provenance で行う。**

#### 🔴 既存プロジェクトへの適用は、そのままコピーしてはいけない

上流の最後の 1 行は次の形である:

```bash
cp -R "$RUNTIME_ROOT/claude/." your-project/
```

**`runtime/<harness>/` には、ハーネスのツリーだけでなく `aidlc/` ワークスペースの殻と
プロジェクトルートのファイル（`AGENTS.md` 等）が同梱されている**（上流逐語:
`so the harness tree, aidlc/ workspace shell, and project-root files stay together`）。

**既存プロジェクトにそのまま `cp -R` すると、次を潰しうる。**

- `aidlc/spaces/<space>/memory/` —— **利用者が積み上げた記憶**
- `aidlc/knowledge/` —— **取り込んだ知識**
- ルートの `AGENTS.md` —— **プロジェクト固有の指示**

新規プロジェクトなら上流の手順でよい。**既存プロジェクトでは、中身の種類ごとに扱いを変えること。**

`aidlc-copy-runtime-2.9.0.tar.gz` の `runtime/<harness>/` を実測すると、中身は 3 種類に分かれる。

| 種類 | 中身（実測） | 既存プロジェクトでの扱い |
|---|---|---|
| **ハーネス管理ディレクトリ** | 下表 | **上書きで更新する**（ここを据え置くと旧版の不具合が残る） |
| **ルートのファイル** | `.gitignore`、`AGENTS.md`（claude 以外）、`.mcp.json`（claude）、`opencode.json`（opencode） | **上書きしない。差分を見て手でマージする** |
| **`aidlc/` ワークスペースの殻** | `active-space`、**`spaces/default/memory/{org,team,project}.md`**、`phases/*.md` | **既存ファイルは絶対に上書きしない。足りないものだけ補う** |

> **⚠ `aidlc/spaces/<space>/memory/org.md` などは利用者が書く記憶ファイルである。**
> **[18.3](./18-release-impact-290.md) の Change Control の `strict` 宣言もここに書く。** 上書きすると統制設定ごと消える。

ハーネス管理ディレクトリ（実測）:

| ハーネス | 管理ディレクトリ |
|---|---|
| claude | `.claude/` |
| codex | `.codex/`、`.agents/` |
| copilot | `.github/`、`.aidlc/` |
| cursor | `.cursor/` |
| kiro / kiro-ide | `.kiro/` |
| opencode | `.opencode/`、`.aidlc/` |

```bash
dest=$(cd your-project && pwd) || exit 1   # 絶対パスにする（下の検査で cd するため）
h=claude                         # ハーネス名
managed=".claude"                # 上表の管理ディレクトリ（複数ならスペース区切り）
R="$RUNTIME_ROOT/$h"

# 1) aidlc/ に中身があれば、まず退避する（mkdir より前に判定する）
if [ -d "$dest/aidlc" ]; then
  if ! contents=$(ls -A "$dest/aidlc"); then
    echo "aidlc/ を読めません。中止します。" >&2; exit 1
  fi
  if [ -n "$contents" ]; then
    cp -R "$dest/aidlc" "$dest/aidlc.bak-$(date +%Y%m%d%H%M%S)" || exit 1
  fi
fi

ts=$(date +%Y%m%d%H%M%S)
work=$(mktemp -d) || exit 1          # 差分や一覧はここに書く（既存のファイルを消さずに済む）

# 2) ハーネス管理ディレクトリは「退避して新しいものを丸ごと置く」
#    上書きだけだと、上流が新版で消したファイルが残り、新旧が混ざる。
#    ただしアーカイブにはファイル単位の所有台帳が無く、上流が消したものと利用者が足したものを
#    機械的に区別できない。そこで旧版にだけあったファイルは消さずに一覧にし、人が判断する
keep="settings.local.json"           # 旧ディレクトリから自動で戻す利用者ファイル（管理ディレクトリ直下からの相対）
for d in $managed; do
  if [ -e "$dest/$d" ]; then
    mv "$dest/$d" "$dest/$d.bak-$ts" || exit 1
  fi
  cp -R "$R/$d" "$dest/$d" || exit 1
  if [ -d "$dest/$d.bak-$ts" ]; then
    for k in $keep; do
      if [ -f "$dest/$d.bak-$ts/$k" ] || [ -L "$dest/$d.bak-$ts/$k" ]; then
        cp -pRP "$dest/$d.bak-$ts/$k" "$dest/$d/$k" || exit 1   # -P: シンボリックリンクはリンクのまま戻す
      fi
    done
    old=$(cd "$dest/$d.bak-$ts" && find . \( -type f -o -type l \)) || exit 1   # シンボリックリンクも含める
    printf '%s\n' "$old" | while IFS= read -r f; do
      [ -n "$f" ] || continue
      if [ ! -e "$dest/$d/$f" ]; then
        printf '%s/%s\n' "$d" "${f#./}" >> "$work/restore-candidates.txt"   # 旧版にだけある
      elif ! cmp -s "$dest/$d.bak-$ts/$f" "$dest/$d/$f"; then
        printf '%s/%s\n' "$d" "${f#./}" >> "$work/changed-files.txt"        # 両方にあり内容が違う
      fi
    done
  fi
done
if [ -s "$work/restore-candidates.txt" ]; then
  echo "旧版にだけあったファイルを $work/restore-candidates.txt に列挙した。"
  echo "利用者が足したもの（独自のエージェント等）は *.bak-$ts から戻し、上流が削除したものは戻さないこと"
fi
if [ -s "$work/changed-files.txt" ]; then
  echo "新旧で内容が違う同名ファイルを $work/changed-files.txt に列挙した（上流の更新も含む）。"
  echo "自分で編集したファイル（プロバイダ設定など）は *.bak-$ts の内容を見て手でマージすること"
fi

# 3) ルートのファイルは上書きしない。既存があれば差分を出し、無ければ置く
for f in .gitignore AGENTS.md .mcp.json opencode.json; do
  [ -e "$R/$f" ] || continue
  if [ -e "$dest/$f" ]; then
    out="$work/upgrade-$(echo "$f" | tr -d .).diff"
    # diff の終了コードは 0 = 同一 / 1 = 差分あり / 2 = 読めない等のエラー。2 とリダイレクト失敗だけを止める
    #   リダイレクトに失敗した場合も 1 になるので、差分ファイルが空でないことも確かめる
    if diff -u "$dest/$f" "$R/$f" > "$out"; then rm -f "$out"
    else
      rc=$?
      { [ "$rc" -eq 1 ] && [ -s "$out" ]; } || { echo "$f の差分を作れませんでした" >&2; exit 1; }
      echo "$out を確認し、$f を手でマージすること"
    fi
  else
    cp "$R/$f" "$dest/$f" || exit 1
  fi
done

# 4) aidlc/ は足りないファイルだけ補う。既存ファイル（利用者の記憶）には一切触れない。
#    cp -n の終了コードは「既存を飛ばした」と「実エラー」を区別できず、存在確認だけでは
#    途中で切れたファイルを見逃すため、欠けているファイルを 1 本ずつコピーし、
#    その都度「cp の終了コード」と「内容の一致（cmp）」の両方を確かめる
list=$(cd "$R/aidlc" && find . -type f) || { echo "展開したアーカイブの aidlc/ を読めません" >&2; exit 1; }
[ -n "$list" ] || { echo "展開したアーカイブの aidlc/ が空です" >&2; exit 1; }
printf '%s\n' "$list" | while IFS= read -r f; do
  { [ -e "$dest/aidlc/$f" ] || [ -L "$dest/aidlc/$f" ]; } && continue   # 壊れたリンクも「既存」として触らない
  mkdir -p "$dest/aidlc/$(dirname "$f")" &&
    cp "$R/aidlc/$f" "$dest/aidlc/$f" &&
    cmp -s "$R/aidlc/$f" "$dest/aidlc/$f" ||
    { rm -f "$dest/aidlc/$f"   # ここで新規に作ったファイルだけを消す（途中で切れている場合がある）
      echo "aidlc/ の補完に失敗しました: $f" >&2; exit 1; }
done || exit 1
```

> **⚠ 手順 2 は管理ディレクトリを `*.bak-<時刻>` に退避してから新版を丸ごと置く。**上流が新版で消したファイルは残らないので新旧は混ざらない。
> **代わりに、利用者が管理ディレクトリ内に足したファイル（独自のエージェントなど）は新しい側に無くなる。**`settings.local.json` だけは自動で戻し、それ以外は `restore-candidates.txt` を見て人が戻すこと。
> **利用者が編集した出荷ファイル（例: `.codex/config.toml` のプロバイダ設定、`opencode.json`）も新版で置き換わる。**
> 新旧で内容が違う同名ファイルは `changed-files.txt` に出るが、**上流の更新と利用者の編集は区別できない**ため
> ほとんどの管理ファイルが載りうる。自分で編集した覚えのあるファイルを中心に、退避側と見比べてマージすること（アーカイブにファイル単位の所有台帳が無いため、機械的には区別できない）。
>
> **検証の範囲（2026-09-23）**: 実物の `aidlc-copy-runtime-2.9.0.tar.gz` を展開し、
> **claude ハーネスについて、使い捨ての既存プロジェクトに対して macOS で流した。**
> `org.md`（`strict` 宣言入り）・`.claude/settings.local.json`・既存 `.gitignore` が保持され、
> `.claude/settings.json` が更新され、不足していた記憶ファイルが補われ、`aidlc.bak-*` が作られることを確認した。
> **管理ディレクトリの入れ替えも確かめた** —— 旧版にだけあったフックは新側に残らず、`settings.local.json` は自動で戻り、
> 利用者の独自エージェントは `restore-candidates.txt` に挙がって `*.bak-*` 側に保全された。
> **失敗側も確かめた** —— 書き込み不能なディレクトリと、ファイルサイズ制限による途中切れ（1024 バイトで切れた `project.md`）の
> どちらでも手順 4 が exit 1 とファイル名を出して止まり、既存の記憶ファイルは無傷だった。
> **他ハーネス・Linux・Windows では流していない。** 社内で 1 度確かめてから手順書に採ること。

> **🔴 v2.9.0 で手動コピー用のアセットが分離された。取得するファイル名が変わっている。**
>
> | 版 | ネイティブ用 | **手動コピー用** |
> |---|---|---|
> | v2.8.x | `aidlc-runtime-X.Y.Z.tar.gz` | 同じもの |
> | **v2.9.0 以降** | `aidlc-runtime-X.Y.Z.tar.gz` | **`aidlc-copy-runtime-X.Y.Z.tar.gz`** |
>
> **v2.9.0 で `aidlc-runtime-2.9.0.tar.gz` を取っても手動コピー用ではない。**
> CHANGELOG 逐語:
> ```
> Manual-copy users must replace the complete `runtime/<harness>/` tree from
> `aidlc-copy-runtime-2.9.0.tar.gz`.
> ```
> （→ [18.4](./18-release-impact-290.md)）

> **⚠ 版の固定は、経路ごとに対象が違う。**
>
> | 経路 | 固定するもの |
> |---|---|
> | **手動コピーのみ**（v2.9.0 以降の既定） | **`aidlc-copy-runtime-X.Y.Z.tar.gz` だけ**。ネイティブ導入は不要 |
> | **ネイティブ導入のみ** | `install.sh --version X.Y.Z`（資産は `aidlc-runtime-X.Y.Z.tar.gz`） |
> | 両方を併用する場合 | **両方に同じ `X.Y.Z` を指定する。** `releases/latest` で入れたバイナリと別リリースのアーカイブを混ぜない |
>
> 導入済みの版は `aidlc version` で確認できる（ネイティブ導入時のみ）。

**チェックアウトから `bun scripts/package.ts <harness>` で `dist/<harness>/` を生成することも今も可能**だが、
上流はこれを利用者向けとは認めていない（`docs/guide/12-cli-commands.md:243-246` 逐語）:

> Framework developers may generate the ignored Bun-shaped `dist/` projection locally with
> `bun scripts/package.ts`; **release users should not copy from a checkout.**

なお `bun scripts/package.ts <harness>` は `dist/<harness>/` と `dist-release/<harness>/` の
**両方**を生成する。前者は `bun …` を呼ぶ Bun 前提の投影、後者はネイティブ `aidlc` を呼ぶ形である。
**v2.9.0 以降はどちらもリリース資産になる** —— `dist/<harness>/` は
`aidlc-copy-runtime-X.Y.Z.tar.gz`、`dist-release/<harness>/` は `aidlc-runtime-X.Y.Z.tar.gz`
の中身である（`scripts/package-release.ts` 実測）。
**2.8.x までは後者だけがリリース資産だった。**

### 初回実行ウィザード

未設定プロジェクトで TTY から引数なしに `aidlc config` を実行すると、初回ウィザードが動く。

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

| # | 内容 | 選択肢 |
|---|---|---|
| 1 | Harness | 7 種から |
| 2 | Model provider | `amazon-bedrock` / `other`（bedrock なら region と profile） |
| 3 | Model effort preset | `balanced` / `thorough` / `minimal` |
| 4 | Plugins | `all installed` / `none optional` / `choose` |
| 5 | MCP servers | `on` / `off` |
| 6 | 記録先 | project 共有 / project 個人 / machine |

最後に `Apply? [Y/n]` のゲートがあり、**それより前にはファイルを 1 つも書かない**。

> **⚠ このゲートで Enter を押すとキャンセル扱いになる不具合がある。**
> **`c03f9e28` で修正され、リリース版 `v2.8.1` 以降に含まれる（解消済み）。**
> **v2.8.0 を使い続ける場合のみ**、既定を受け入れるときも明示的に `y` を入力すること。

セクションを指定してピンポイントに設定することもできる:
`aidlc config models` / `runtime` / `providers` / `trust` / `flags` / `project`。

> **⚠ 名前が衝突している。** ここで説明したネイティブ CLI の `aidlc config` と、
> セッション内のスラッシュコマンド `/aidlc config`（[6.5](#65-よく使うコマンド)。depth / test-strategy / review）は**別物**である。

### ハーネス別の追加要件

導入コマンド自体は共通だが、**ホスト側の前提はハーネスごとに残る**。

#### Claude Code

`aidlc config --harness claude` の後、`/hooks` からプロジェクトフックを承認して**完全に再起動**する。
`disableAllHooks` が報告される場合は編集可能なレイヤで削除・上書きが要る。
`allowManagedHooksOnly` は管理者ポリシーの変更が要る。

#### Kiro IDE / CLI

`aidlc/` は `.kiro/` の**兄弟**（内側ではない）。IDE と CLI は別のハーネスとして扱う
（`--harness kiro-ide` / `--harness kiro`）。読み分けは [6.4](#kiro-cli-と-kiro-ide-は分けて読むこと) を参照。

#### GitHub Copilot

**導入しただけでは動かない。folder trust が要る。**

1. **プロジェクトを信頼する**（必須）。`copilot` を対話起動して trust プロンプトを承認するか、
   `~/.copilot/config.json` の `trustedFolders` にプロジェクトの絶対パスを追加する。
   **未信頼だとリポジトリフックが 1 本も動かない**
2. ヘッドレス（`copilot -p`）で使う場合は `GITHUB_COPILOT_PROMPT_MODE_REPO_HOOKS=1`
3. `AGENTS.md` の「Git Integration」節の `.gitignore` 記述を適用してからワークフローを開始する

配置先: `/aidlc` とステージ／スコープランナーは `.github/skills/`、
14 ペルソナは `.github/agents/`、フック定義は `.github/hooks/aidlc.json`。
`aidlc/` は `.aidlc/` の**兄弟**（内側ではない）。

#### Codex CLI

**プロジェクトが git repo であること**が前提（Codex が `.codex/hooks.json` を発見する条件）。
上流のハーネス別ガイドも「Codex requires the target project to be a Git repository for project hook discovery」と明記している。

**hooks の trust（未信頼だと hooks が動かない）** — どちらか:

1. **TUI で "Trust all"**（初回セッション）
2. **生成した trust エントリを `$CODEX_HOME/config.toml` に反映**する。
   現在の trust 状態は `aidlc config trust --show` / `--check` で確認できる。
   上流リポジトリのチェックアウトから生成する場合は従来どおり:

   ```bash
   bun install --frozen-lockfile
   bun scripts/package.ts codex trust --project <プロジェクトの絶対パス>
   # 出力の [hooks...] TOML を $CODEX_HOME/config.toml にマージ
   # （同一 hook path の既存エントリは置換。二重追記で TOML が壊れる）
   ```

   > **⚠ ネイティブ導入時に同等の TOML を生成する上流コマンドは未確認である。**
   > `aidlc config trust` は検証・確認系のサブコマンドとして実装されているが、
   > 生成手段が同じかは本調査では確かめていない。

> **trust テーブルは「初回だけ」ではない（2.6.44）。**
> 2.6.44 で `request_user_input` を拾う新しい PostToolUse 登録が
> **`.codex/hooks.json` の PostToolUse 配列の先頭に挿入された**。
> trust エントリは `post_tool_use:N` という**位置インデックス**で hook を指すため、
> 先頭挿入によって**既存エントリのインデックスが全部 1 つずつズレる**。
> **エンジンを更新しただけでは、古い trust テーブルが別の hook を指したままになる。**
> 更新時は trust エントリを**差し替え**（追記ではない。同一 hook path の古いエントリは消す）、
> そのうえで**新しい Codex セッションを開始する**。
>
> なお **2.7.2 以降**は hook の起動コマンド自体が `{{INVOKE}} engine hook <name>` 形式に変わったため、
> **trust のハッシュ対象文字列も変わっている**。更新後の trust 再登録は必須である。

#### Cursor

2.7.1 以前は「他ハーネスと違い、同梱インストーラ `bun dist/cursor/install.ts <project>` を実行する」
という**Cursor だけ別扱い**の導入形態だった。
**2.7.2 以降（リリースとしては v2.8.0 以降）は 7 ハーネスすべてが `aidlc config --harness <name>` に統一され、この非対称は解消した。**
（`harness/cursor/install.ts` はソース側に残っているが、利用者が直接叩く経路ではない。）

- IDE と CLI（`agent`）は**同じ `.cursor/` を読む**ので、導入は 1 回でよい
- `aidlc/` は `.cursor/` の**兄弟**
- Cursor ネイティブのショートカットとして `/aidlc-status`・`/aidlc-jump --stage <slug>`（または `--phase <name>`）・
  `/aidlc-scope <name>` が入る（同じ決定論エンジンを叩く）

> **このハーネスで違うこと**（公式 `docs/guide/harnesses/cursor.md`）:
> 質問は構造化ウィジェットではなく**番号付きの散文**で描画される（正は `[Answer]:` タグ付きの質問ファイル）。
> フックは `.cursor/hooks.json` から**アダプタ 1 本**（`.cursor/hooks/aidlc-cursor-adapter.ts`）を経由し、
> Cursor の camelCase イベントを core のフック本体に写す。PreToolUse ガードは
> `{"permission":"allow"|"deny"}` を stdout に返し、`failClosed: true` で登録されている
> （＝入力不正・ガード欠落・ガードのクラッシュはいずれも deny）。
> 一方 `stop` フックは停止を拒否できないため、**転送ループの強制は advisory**（opencode と同じ姿勢）。
> Cursor 固有の `Delete` ツールは reviewer スコープガードに「書き込み」として提示される。

> **既知の不具合と修正**: 2.5.63〜2.5.68 の Cursor 配布物は、allow 経路で stdout に何も書かずに終了していた。
> （この範囲のうち **2.5.65 / 2.5.66 は欠番**で存在しない。実在するのは 2.5.63 / 64 / 67 / 68。
> 自分の版を照合するときは `aidlc version`、またはソースの `core/tools/aidlc-version.ts` を見ること。）
> `failClosed: true` の下では空 stdout が不正 JSON と扱われるため、**Cursor IDE ではあらゆるツール呼び出しが
> ブロックされた**（CLI は沈黙を allow と解釈したため無症状）。**2.5.69 で修正済み**。
> **⚠ ただし同じ失敗様式が v2.8.0 で再発している**（ネイティブ化でアダプタ経路が外れたため。→ 下記の表と 17.3）。
> **したがって「2.5.69 以降なら安全」ではない。**
> - **2.5.63〜2.5.68 に当たっている場合**: エンジンを **2.5.69 以上 2.7.x 以下**へ更新して再導入する。
> - **v2.8.0 に当たっている場合**: **v2.8.1 以降へ更新すれば直る**（推奨は現 Latest の v2.9.0）。
>   修正 `52da70ad` は **v2.8.1 に含まれる**（`git fetch origin --tags` してから
>   `git merge-base --is-ancestor 52da70ad v2.8.1` で確認済み）。
>   **かつてここに記していたソース生成による暫定回避は、もう不要である。**
> 切り分けは [6.6 のトラブルシュート表](#66-トラブルシュート頻出)を参照。

#### opencode

`aidlc/` と `.aidlc/`（エンジン）は `.opencode/` の**外**に置かれる。
理由: opencode は `.opencode/tools/*.ts` をカスタムツールとして自動 import するため、
エンジンをそこへ置くとクラッシュする。

---

## 6.4 アップグレードでハーネスごとにすること（2.6.2 → 2.6.55）

> **⚠ 2.6.55 → 2.6.123 では、再コピーだけでは済まない作業が過去最多になった。**
> 本節の表は **2.6.55 までの手順**である。2.6.123 へ上げるときは、**全 62 版の Upgrade 文を精査した結果、
> 下の 20 版について追加作業が要る**（根拠と詳細は
> [15-release-impact-26123.md](./15-release-impact-26123.md)）。
>
> **2.6.123 から 2.7.0 へ上げるだけなら、下の表の作業は増えない。** 2.6.124 に移行処理は無く、2.7.0 が変えたのは**バージョン定数だけ**でロジックの変更は無い。
> **ただしプラグインを入れているなら、再コピー後に `/aidlc plugin sync` が要る**
> （エンジンを入れ替えるとコンパイル済みグラフが素に戻り、合成が失われる）。これは版によらず毎回必要である。
> ただし **配布物の入手方法そのものが 2.7.2 で変わった**（リリースとしては v2.8.0 以降）—— `dist/` は上流から消え、
> エンジンの入手は `install.sh` / `install.ps1`、プロジェクトへの適用は `aidlc config` になった
> （→ [17.1](./17-release-impact-2801.md#171-いちばん大きい変更は-dist-の消滅)）。
> 本節以下に並ぶ版ごとの手順は、**その版の時点で上流が指示していた操作**の記録である。
> 「`dist/` の再コピー」と書かれている箇所の読み替えは、**現在の導入状態で変わる**。
>
> | 現在の状態 | 読み替え先 |
> |---|---|
> | **2.7.1 以前（`dist/` をコピーした状態）** | **`install.sh` / `install.ps1` → `aidlc config`**。`aidlc update` は使えない |
> | ネイティブ `aidlc` を導入済み | `aidlc update` → 各プロジェクトで `aidlc config` |
>
> **⚠ 初回の移行で `aidlc update` から始めてはいけない。**
> 2.7.1 以前の導入にはネイティブ `aidlc` 実行ファイルが存在せず、最初のコマンドで失敗する。
>
> **⚠ 出荷ファイルを手編集している場合は、`aidlc config` の前に片付ける。**
> 手編集したファイルがあると `aidlc config` は `conflict` を出して止まる。**対処は対象で違う。**
>
> | 対象 | 対処 |
> |---|---|
> | **管理下のハーネスファイル**（`.claude/` `.github/` などツリー内。Composer 定義もここ） | **編集内容を退避してから `--force`**。`--force` で置換される（**編集は失われる**） |
> | **ルート統合の未マーク AI-DLC 内容**（`AGENTS.md` の区画など） | **`--force` は効かない。** 移動または削除してから作り直させる |
>
> （詳細は [4.6 カスタマイズの正しい場所](./04-agents.md#46-カスタマイズの正しい場所)）。
>
> **⚠ 「再コピー後に `/aidlc plugin sync`」という指示の扱いは、2.8.x では断定できない。**
> 上流の記述が 2 つに割れているためである（HEAD `c03f9e28` 実測）——
> `docs/guide/12-cli-commands.md` は「engine の再インストール・アップグレードのたびに再実行せよ」と書くが、
> **その理由は「新しい `dist/<harness>/` をコピーすると出荷グラフに戻るため」**であり `dist/` コピー前提である。
> 一方 `docs/guide/18-install-and-lifecycle.md:818-820` は
> 「プラグイン変更はプロジェクト設定であり `aidlc config` に収束する。独立した公開プラグインコマンドは無い」
> 「**`aidlc doctor` が installed 対 composed のプラグイン状態を報告する**」と書く。
>
> **実務手順**: 更新後に `aidlc doctor` を実行し、**ずれが報告された場合に `/aidlc plugin sync`** を打つ。
> 手動コピー導入・明示的なプラグイン変更・構成破損の場合は従来どおり明示同期が要る。
> **Kiro CLI は SessionStart の自己修復が効かない**ため、明示実行の必要性が最も高い。
> また 2.6.124 は**既存の `aidlc-state.md` を書き換えないし、コミット済みの履歴も変えない**。
> **2.7.0 の CHANGELOG は 2.6.x 全体のロールアップ再掲なので、それを読んだだけでは
> 下の 20 版の一度きりの作業は済まない**（→ [16.4](./16-release-impact-2700.md#164-270-の-changelog-はロールアップであって新機能一覧ではない)）。
> エントリが挙げる「2.6.1 より前に作成したワークフローは完了・アーカイブしておくこと」も、
> **2.7.0 の新制約ではなく 2.6.1 の state スキーマ v8 の再掲**である。
> 自分の構成に該当する行だけ拾えばよい。
>
> **(a) 自作ステージ／プラグインを持っている場合**
>
> | 版 | 追加で要ること |
> |---|---|
> | **2.6.121** | **`reviewer:` を宣言する自作・プラグインステージすべてに `review_artifact:` を追加**し、`produces[]` の必須 Markdown を 1 つ選ぶ。per-Unit ステージでは関連する全 Unit 種別に適用。**未移行のステージはコンポーズとグラフコンパイルが拒否する**（発火点は compose / compile。既存のコンパイル済みグラフを再コンパイルしないなら動き続けるが、**プラグイン利用者は復旧の `plugin sync` で compose が走るため上げた時点で踏む**） |
> | **2.6.65** | 同じ成果物を `produces` / `optional_produces` に重複宣言していると**コンパイルが通らない**。片方を改名するか consumer を直す |
> | **2.6.94** | composer proposal を読む統合は `birthDescription` → **`creationDescription`** に改名。改名された内部ヘルパを import しているカスタムツールは import を更新 |
>
> **(b) プラグインを使っている場合**
>
> | 版 | 追加で要ること |
> |---|---|
> | **2.6.110** | `dist/` 上書きでプラグイン合成が黙って消える。**`/aidlc plugin sync` を実行**（`--doctor` の `Composed plugin surface` が exit 1 で検出）。Claude / Codex / Cursor / Kiro IDE は次セッションで自己修復するが **Kiro CLI は明示実行が必須** |
> | **2.6.111** | 各プラグインの projection も refresh して **re-compose**。新しい advisory が名指しするインストール済みファイルを**削除**してから compose し直す |
> | **2.6.61** | 各プラグインの `dist/plugins/<name>/<harness>/` projection を refresh・re-compose して doctor スクリプトを導入する |
> | **2.6.78** | プラグインルートが設定済みで使える compose フックが 1 つも無いと `plugin sync` が **exit 1** になる。**自動化を「不完全なプラグイン導入」として扱うよう修正**（CI が落ちるようになる） |
>
> **(c) Kiro を使っている場合**
>
> | 版 | 追加で要ること |
> |---|---|
> | **2.6.64** | **手動削除**（`rm -f <proj>/.kiro/agents/aidlc.json` / `.kiro/agents/aidlc-*-agent.json` / `.kiro/settings/cli.json`）。上流「An overlay copy cannot delete retired files.」 |
> | **2.6.85** | **Kiro IDE の overlay インストールに新しい `aidlc-terminal-command*` フック登録を含める** |
> | **2.6.60** | インストール済み Kiro プラグインの **compose を再実行**。doctor が「composed 済みの非対応値」を報告したら、プラグインソースを直し、名指しされたペルソナを削除して compose し直す |
>
> **(d) 進行中のワークフロー／レビューがある場合**
>
> | 版 | 追加で要ること |
> |---|---|
> | **2.6.122** | 進行中の `workspace_requires` レビューは、**完了前にもう一度レビューし直す**（source identity の形式が変わったため） |
> | **2.6.114** | 本版より前に打刻された **no-DAG の stage-level レビュー請求**は、Bolt がマージされると経路が変わって一致しなくなりうる。verdict を記録する前に `--retry-pending` で**同じ pending 序数を再配車**する |
> | **2.6.69** | per-unit の workspace レビュー請求のたびに妥当な **`source-manifest.json` を書く**。verdict 前にソースが変わったら `--retry-pending` で再配車 |
>
> **(e) その他（該当者のみ）**
>
> | 版 | 追加で要ること | 対象 |
> |---|---|---|
> | **2.6.93** | `/hooks` で Claude プロジェクトフックを承認し、**Claude Code を完全に再起動** | Claude Code |
> | **2.6.108** | **人が居ないまま prompt を投げるプロセスに `AIDLC_UNATTENDED=1` を設定**（→ [15.4](./15-release-impact-26123.md#154-人間の関与--無人ドライバの抑止は-opt-in26108)） | cron / CI / 夜間ランナー |
> | **2.6.71** | **`aidlc/spaces/<space>/memory/org.md` を手編集**して新規セッションを開始（既存ワークスペースの `org.md` は再コピーで上書きされない） | 非英語で使う場合 |
> | **2.6.80** | `bun scripts/package.ts codex trust --project <絶対パス>` の**再実行** | Codex |
> | **2.6.91** | 空木ハッシュ `4b825dc642cb6eb9a060e54bf8d69288fbee4904` で保存された CodeKB scope タイムスタンプの**再採番**（または Reverse Engineering の再実行） | CodeKB 利用者 |
> | **2.6.84** | コンパイル済みバイナリで入れている場合は**実行ファイルと隣接する `runtime/` ディレクトリを置換**する（ソース導入なら通常の `dist/` 再コピーでよい） | バイナリ導入 |
> | **2.6.96** | センサーキャッシュに対する**利用者側の回避用 ignore ルールを削除**する | 回避策を入れていた場合 |
>
> **移行しないと止まるのは 4 つ —— 2.6.121・2.6.65・2.6.94・2.6.78 である。止まる契機はそれぞれ違う。**
>
> | 版 | いつ止まるか |
> |---|---|
> | **2.6.121** | **compose / graph compile を走らせたとき**（上流: "Composition and graph compilation reject reviewer stages that are not migrated."） |
> | **2.6.65** | **graph compile を走らせたとき**（authored `.md` を読む `aidlc-graph compile` が `throw`） |
> | **2.6.94** | **改名された内部ヘルパを import しているカスタムツールを起動したとき**（import が解決しない） |
> | **2.6.78** | **CI から `plugin sync` を呼んでいて、設定済みのプラグインルートに使える compose フックが 1 つも無いとき**（exit 0 → **exit 1**。パイプラインが落ちる） |
>
> **2.6.121 と 2.6.65 は「上げた瞬間」ではない。** 既にコンパイル済みのグラフを持っていて
> 再コンパイルしないなら、そのまま動き続ける（2.6.95 の doctor チェックは既存グラフ向けだが
> **advisory で exit code を変えない**）。
> ——ただし**該当条件を持つプラグイン**（未移行の `reviewer:` ステージ、または重複プロデューサ）を
> 使っている場合は**上げた時点で踏む**。`dist/` の入れ替えでコンパイル済みグラフが素に戻り（2.6.110）、
> 復旧に要る `/aidlc plugin sync` が compose を走らせるためである。
> **どちらの条件も持たないプラグインは普通に compose される。**
> 残り 16 版は**ワークフローの実行そのものは止めない**（不便・危険ではある）。
> なお 2.6.102（lifecycle フィールド追加）は「file 単位のドリフト診断が欲しい場合のみ」と
> 上流が明記しているので、必須作業には数えていない。

### 全ハーネス共通のスワップ手順（2.6.51 以降の前提）

2.6.51 で継続カーソルが 7 ハーネスに一般化されたことにより、`dist/` の入れ替えは
**「ファイルを上書きするだけ」の操作ではなくなった**。以下の 4 点はハーネスによらず一律に掛かり、
下の表の **○** も「これを満たしたうえで」の意味になる。

1. **静止状態で行う。** AI-DLC のコマンドが 1 つも走っておらず、フックも発火していない瞬間に
   交換を完了させる（上流原文: `in one quiescent swap (no AI-DLC command or hook running)`）。
2. **部分適用せず、ツリー全体を一度に入れ替える。** 新旧混在は非サポート
   （2.7.1 以前は `dist/<harness>/` の全ツリーコピー。2.7.2 以降は `aidlc config` が
   トランザクションとして同じ保証を担う）
   （`mixed old/new tool files are unsupported`）。旧 `aidlc-orchestrate` は 2.6.51 で削除された
   シンボルを named import するため、混在させると**挙動が混ざるのではなく AI-DLC のコマンドが全部落ちる**。
   同種の制約は 2.6.50 にもある（旧散文の `Accept as-is after N cycles` や
   `--result rejected --user-input "<feedback>"` は 2.6.50 の新ガードで拒否される）。
   **ツールと散文は同時に入れ替える**こと。
3. **交換後に `next` を打つ。** ステージ進捗は巻き戻らない。失うのは
   **ステージ規則の分割配信の途中位置**だけである。進行中トークンは
   project / intent / state / トークンダイジェストの **4 点が一致する場合にのみアトミックに移行する**。
4. **ロールバックはセキュリティ後退である。** 旧版へ戻すと sessionless の同一トークン再生
   （上流 issue #762）の抜け穴が復活する。動くが弱くなる、という後退であることを理解して選ぶこと。

> **前提となる実行環境**: 2.6.51 の exactly-one-winner は**単一ローカルファイルシステム**を前提とし、
> 上流が unsupported と明記しているのは、**必要なプリミティブ（同一 FS 内のアトミック rename・
> 排他ディレクトリ作成・プロセス間の一貫した可視性）を尊重しない**ファイルシステムである。
> NFS / SMB / FUSE / オブジェクト同期フォルダが**常にそうだとは限らない** → 6.2.1
> → [6.2 実行環境の前提](#621-実行環境の前提-単一ローカルファイルシステム2651-以降)

> 「新旧混在ではロードエラーになる」は ESM の named-import 契約からの帰結であり、
> **実際に半々のツリーを動かして確かめたものではない**（本ノート未検証）。

### ハーネス固有の追加操作

凡例（**上記の共通手順を満たしたうえで**、さらにハーネス固有の操作が要るか）:
**○** = 共通手順だけで完了し、ハーネス固有の追加操作は不要 /
**✗** = 共通手順に加えて**必ず**ハーネス固有の追加操作が要る /
**△** = 通常は共通手順だけで済むが、**特定の条件下でのみ**ハーネス固有の追加操作が要る
（Kiro CLI / Kiro IDE はプラグイン利用時）。

> **2.6.36 の learning selections 非互換は、どのハーネスでも起こりうる。**
> 進行中ワークフローが 2.6.36 より前に生成された selections ファイルを持っていれば、
> ○ の行のハーネスでも該当ステージの `surface` 再実行が要る（→ 本節末）。

> **`cp` の書式に注意。** 本ノートのコピー例は `cp -R <src>/. <dst>/` の形（末尾が `/.`）で統一している。
> `cp -R <src>/ <dst>/`（末尾がスラッシュのみ）は、**GNU cp（Linux）では宛先が既存だと
> `<dst>/<src名>/` に入れ子で置かれる**とされ、既存のエンジンが更新されないまま残る。
> 一方 macOS の BSD cp は同じ書き方でも中身をマージする（本ノートの環境で実測）。
> **GNU cp 側の挙動は本ノートでは実機確認していない**（調査環境が macOS のため）が、
> `/.` 形式は両方で「中身をマージ」になるので、書式を統一しておけばこの差に依存しない。

| ハーネス | 共通スワップ手順のほかに**ハーネス固有の**操作が要るか | 追加で必要な操作 |
|----------|------------------------------|------------------|
| **Claude Code** | ○ | なし |
| **Cursor** | ○ | なし（`bun dist/cursor/install.ts <project>` の再実行が「再コピー」に相当。**2.6.48 は対象外** — 元から具体パスを出荷していた） |
| **Codex CLI** | ✗ | **2.6.44**: `bun scripts/package.ts codex trust --project "<絶対パス>"` を再実行し、既存 trust テーブルを**差し替え**、**新セッションを開始**（§6.3 の囲み参照） |
| **GitHub Copilot** | ○ | **2.6.48** の対処は再コピーそのもの。**2.6.12** 由来の「進行中ワークフローがある場合は新しい会話を開始」は、**2.6.51 で共通手順の `next` に一般化された**ため、Copilot 固有の追加操作としては残らない（→ [6.6](#github-copilot-アップグレード後は進行中ワークフローを新しい会話で継続する2612) と上記の共通手順） |
| **opencode** | ○ | **2.6.48** の対処は再コピーそのもの（プレースホルダ持ちペルソナが具体パス版に置き換わる）。追加操作は無い |
| **Kiro CLI** | △ | **2.6.46**: 再コピーで verb interceptor 修正が入る（**CLI のみ**）。**2.6.47**: プラグイン利用時は projection を再ビルド／再コピーしたうえで `aidlc plugin sync` か `hooks/compose.ts` を**明示実行** |
| **Kiro IDE** | △ | **2.6.47**: プラグイン利用時は projection を再ビルド／再コピー。新規 `.kiro/hooks/aidlc-<plugin>-compose.json`（SessionStart 登録）が自動で効くので**明示実行は不要**（CLI と対処が違う） |

> **⚠ 廃止された旧フックは再コピーでは消えない。** 2.6.47 で
> `hooks/aidlc-plugin-compose.kiro.hook` は Kiro CLI / Kiro IDE **双方の projection から削除された**が、
> `cp -R` は削除を反映しない（上書きとコピーのみ）。既存インストールを更新した場合、
> **旧フックがそのまま残る**ので手で消す。12 章で 2.6.1 の残骸検出をしたのと同じ種類の作業である。
>
> ```bash
> # 残骸の検出（プラグインを使っている場合のみ該当）
> find your-project -name "aidlc-plugin-compose.kiro.hook"
> # 見つかったら削除
> ```

### Kiro CLI と Kiro IDE は分けて読むこと

同じ「Kiro」でも今回の 2 件は影響範囲が違う。

- **2.6.46 の verb interceptor 修正は Kiro CLI のみ。** 生の `/aidlc --status` / `space` / `space-create` / `intent` 等が silent no-op になっていた不具合の修正で、**Kiro IDE の dist には同ファイルの変更が無い**。
- **2.6.47 のプラグイン compose 配線は両方が影響を受けるが、対処が違う。**
  旧 `hooks/aidlc-plugin-compose.kiro.hook` は **CLI / IDE 双方の projection から削除された**。
  - **CLI**: hook 登録をもう出さない。フォルダを置いたあと `aidlc plugin sync` か `hooks/compose.ts` を**自分で実行**する。
  - **IDE**: 代わりに `.kiro/hooks/aidlc-<plugin>-compose.json`（v2 `SessionStart` 登録）が出る。ワークスペースルートから Bun ランチャを起動するので、**明示実行は要らない**。
- **旧ファイル名に依存したスクリプトがあれば外すこと。** `aidlc-plugin-compose.kiro.hook` はもう出荷されない。

### ペルソナ記憶パスの固定（2.6.48）

opencode と GitHub Copilot の出荷ペルソナは、記憶参照を `aidlc/spaces/<active-space>/memory/...` という**可変プレースホルダ**で持っていた。2.6.48 でこれが `aidlc/spaces/default/memory/...` という**具体パスの default-space シード**に変わり、`/aidlc space default` が出荷ファイルを書き換えなくなった（byte-identical のまま）。`/aidlc space <name>` で別 Space に切り替えれば、従来どおり全ペルソナが張り替えられる。

**Cursor は元からこの形だったので対象外。** 影響を受けるのは opencode と Copilot の 2 つだけで、対処は `dist/<harness>/` の再コピー（プレースホルダ持ちの古いペルソナを置き換える）。

### 学習 selections ファイルの非互換（2.6.36）

`aidlc-learnings.ts persist` の冪等キーが positional candidate id（`c1`）から**学習内容自体の SHA-256** に変わり、`surface()` の出力スキーマに `space` / `intent` が追加された。このため:

- **アップグレード前に生成した selections ファイルは失敗する。** エラーは `selections-json is malformed: missing or non-string space`。
- **対処は該当ステージの `surface` を再実行して selections を作り直すこと。** `persist` を単純にリトライしても直らない（ファイル自体に新フィールドが無いため）。

### `dist/` の変更ファイル数を「開発量」と読まないこと

> **⚠ この測定手法は 2.7.2 以降では再現できない。** `dist/` が上流リポジトリから消えたためである。
> 以下は測定当時（2.6.x 期）の記録として残す。

2.6.2 → 2.6.49 で `dist/` の変更ファイル数はハーネス間でほぼ同数になる。

| ハーネス | 変更ファイル数 |
|----------|----------------|
| opencode | 139 |
| copilot | 139 |
| codex | 123 |
| claude | 116 |
| kiro | 115 |
| kiro-ide | 114 |
| cursor | 114 |

**これは「全ハーネスに等量の固有開発があった」という意味ではない。** 共有コア（`aidlc-lib.ts`・`aidlc-state.ts`・プロトコル・14 ペルソナ・stage-runner の SKILL.md 等）が機械的に全ハーネスへ投影される構造の効果である。opencode と copilot が突出するのも固有開発量ではなく、**同一内容を中立エンジン木（`.aidlc/`）とネイティブ殻（`.opencode/` / `.github/`）の 2 箇所に投影する**ためで、basename のユニーク数で比べると差は縮む（claude 116/78・copilot 139/87・cursor 114/76・kiro-ide 114/76）。ハーネス固有の実質差分は数ファイル単位で、上の 6.4 の表（CHANGELOG の Upgrade 文が名指しした版）と一致する。

---

## 6.5 よく使うコマンド

| コマンド | 意味 |
|----------|------|
| `/aidlc --doctor` | 環境・設定ヘルスチェック |
| `/aidlc --doctor --export` | 秘匿化済み診断レポート（2.5.2+） |
| `/aidlc --status` | 進捗 |
| `/aidlc <description>` | ワークフロー開始（scope 自動） |
| `/aidlc bugfix ...` | 明示 scope |
| `/aidlc compose "..."` | カスタム計画 |
| `/aidlc --scope X --stage Y` | ジャンプ |
| `/aidlc --review <class>` | この実行中のレビュークラス上限（`adversarial` / `advisory` / `none`）。2.5.54+ |
| `/aidlc config list` | depth / test-strategy / review |
| `/aidlc space [name]` / `space-create <name>` | Space の一覧・切替 ／ 新規作成 |
| `/aidlc intent [name]` | Intent の一覧・切替 |
| `/aidlc plugin select [names]` | このインストールで有効なプラグイン一覧の表示・設定 |
| `/aidlc plugin list|sync` | プラグイン |

Codex は `$aidlc` 表記。Cursor には加えてネイティブの `/aidlc-status`・`/aidlc-jump`・`/aidlc-scope` がある。

### 直接ツール呼び出し（`/aidlc` サブコマンドではない）

次の 2 つは `/aidlc <x>` の形を取らず、ツールを直接起動する。

| コマンド | 意味 |
|----------|------|
| `bun <harness-dir>/tools/aidlc-utility.ts codekb-scope-diff --repo <repo>` | Reverse Engineering 再実行前に codekb ストアの鮮度を確認（`NO_STORE` / `CURRENT` / `STALE` / `UNVERIFIED` / `UNKNOWN_SCOPE`）。2.5.35+ |
| **`aidlc system workspace-sync [--force]`** | 任意の `repos.json` に基づき不足リポジトリを clone、管理対象 `.gitignore` を更新、VSCode マルチルート生成。2.5.36+。**2.7.2 以降、上流はこのネイティブ形式を案内している**（従来は `bun <harness-dir>/tools/aidlc-workspace-sync.ts`） |

> **⚠ この 2 つは 2.8.x で扱いが分かれた。**
> - `workspace-sync` は上流ドキュメントが **`aidlc system workspace-sync`** を案内するようになった。
>   ネイティブ導入だけの利用者もそのまま実行できる。
> - **`codekb-scope-diff` は上流ドキュメントが今も `bun …/aidlc-utility.ts` 形式のままである**
>   （`docs/guide/12-cli-commands.md:1013-1015`、HEAD `c03f9e28` 実測）。
>   ディスパッチャ上は `aidlc engine workspace codekb-scope-diff` というルートが存在するが、
>   **`engine` 名前空間は上流自身が
>   「Engine machinery - generated harness surfaces only; not for human scripts」と明記した hidden ルート**であり、
>   利用者が直接叩く経路として案内されていない。
>   **したがってネイティブ導入だけの環境では、この診断コマンドの公式な実行手段が現時点で無い**
>   （`bun` を別途入れるか、上流の案内が更新されるのを待つことになる）。**未確認事項として記録した。**

> `--doctor` は 2.5.36 で advisory 行が 3 つ増えた（`aidlc/` 配下の未コミット変更、`repos.json` とディスク上 sibling の drift、管理対象 `.gitignore` ブロックの陳腐化）。後者 2 つは `repos.json` が存在する場合のみ表示される。

---

## 6.6 トラブルシュート（頻出）

| 症状 | 対処 |
|------|------|
| 端末では bun が見えるがハーネスが見えない | 非対話 PATH（`~/.zshenv` 等） |
| Codex doctor が version 不足 | ≥ 0.145.0 |
| Bedrock AccessDenied（Claude/Codex 出荷設定） | モデル有効化 + 資格情報 + region |
| Codex hooks が動かない | §6.3 の trust（TUI または config.toml へ TOML 反映） |
| Codex: アップグレード後に hooks が誤動作／効かない | **trust テーブルの再生成**（2.6.44 で PostToolUse 配列の先頭に新フックが入りインデックスがズレる。§6.3 の囲み） |
| エンジンを更新したが反映されない | **新セッション**起動 |
| Copilot: アップグレード後に進行中ワークフローが進まない／古い挙動をする | **`next` を打つ**（2.6.51 以降。7 ハーネス共通の手順で、Copilot 固有ではない。2.6.12 当時の手当ては「新しい会話を開始」だった。下記） |
| Kiro CLI で `/aidlc --status` 等が無反応（silent no-op） | 2.6.46 の verb interceptor 修正。エンジンを更新して `aidlc config --harness kiro` を再実行（**Kiro CLI のみの修正**） |
| Kiro: プラグインの compose がアップグレード後に走らない | 2.6.47。projection を再ビルド／再コピーし、**CLI は** `aidlc plugin sync` か `hooks/compose.ts` を明示実行（**IDE は不要**）。§6.4 |
| Kiro IDE hooks 無反応 | v2 schema hooks の正しい中身コピー（2.5.10） |
| GitHub Copilot でフックが全イベントでクラッシュする（`undefined is not an object (evaluating 'input.length')`） | **v2.8.0 の既知不具合**（ネイティブ化で Copilot アダプタが引数 1 個のフック経路に落ち、対象が捨てられる）。**v2.8.1 以降へ更新すれば直る**（修正 `52da70ad` は v2.8.1 に含まれる。→ [18 章](./18-release-impact-290.md)） |
| Cursor IDE で全ツール呼び出しがブロックされる | **原因が 2 つある。どちらかを切り分けること。**<br>**(a) 2.5.63〜2.5.68 の既知不具合**（allow JSON 未出力 × `failClosed`）→ **2.5.69 以上 2.7.x 以下**へ更新して再導入。<br>**(b) v2.8.0 の再発**（ネイティブ化で Cursor アダプタが引数 1 個のフック経路に落ちた。→ [17.3](./17-release-impact-2801.md#173--281-は-changelog-にあるがリリースされていない)）→ **v2.8.1 以降へ更新すれば直る**（修正 `52da70ad` は v2.8.1 に含まれる。推奨は現 Latest の v2.9.0） |
| 学習 persist が `selections-json is malformed: missing or non-string space` で落ちる | 2.6.36 の非互換。該当ステージの **`surface` を再実行**して selections を作り直す（`persist` のリトライでは直らない）。§6.4 |

### GitHub Copilot: アップグレード後は進行中ワークフローを新しい会話で継続する（2.6.12）

2.6.12 で Copilot は、直近に配信した AI-DLC ディレクティブを **Copilot 所有のアトミックなエンジンカーソル**で保持し、Stop からの継続とリプレイ拒否をそこで判定する設計になった。カーソルは提示トークンのダイジェストを丸ごと比較してから後続トークンを公開する仕組みで、**アップグレード前の転送マーカーは形式が合わず再利用できない**（欠落・不正・v1・陳腐化したコンテキストは従来のステートレス経路に落ちる）。

そのため、**進行中のワークフローを抱えたまま Copilot 面を更新した場合は、新しい Copilot の会話を開始してから続きを進めること。** 同じ会話を続けると、アップグレード前のマーカーを引きずった状態で再開しようとすることになる。

> **追記: 2.6.51 で状況が変わった。上の 2.6.12 の記述は、当時の事実としては正しい。**
> 2.6.12 の CHANGELOG 自身が「`sessionless:` and **non-Copilot continuation remain stateless in this release**」と
> 明記していたとおり、当時のカーソルは意図的に **Copilot 所有**だった。
> 2.6.51 はこれを **7 ハーネス共通の仕組みに一般化**し、2.6.12 が残していた
> 「sessionless の同一トークン再生」の抜け穴（上流 issue #762）を塞いだ。
> マーカーの保存先もハーネスディレクトリではなく**記録ディレクトリ側**
> （`aidlc/spaces/<space>/intents/<record>/.aidlc-active-directive.json`。intent 生成前は
> `intents/.aidlc-active-directive.json`）に移っている。
>
> **その結果、推奨される手当ては「新しい会話を開始」ではなく `next` を打つことに変わり、
> それは Copilot 固有ではなく 7 ハーネス共通の手順になった**
> （→ [6.4 全ハーネス共通のスワップ手順](#全ハーネス共通のスワップ手順2651-以降の前提)）。
> Copilot 固有として残るのは、セッション所有権と delivery evidence による**マーカーの enrichment のみ**である
> （上流散文: 「Copilot's session ownership and delivery evidence **enrich** that marker but **do not own replay**」）。

---

## 6.7 ソースの確認方法

導入済みなら版はコマンドで確認できる。

```bash
aidlc version
```

ソースを直接見る場合:

```bash
# 17 章が対象とする 2.8.1 のソースを照合する場合は SHA を指定する
# ⚠ c03f9e28 はリリース版 v2.8.1（= 215afe1a）ではない。タグは 5 コミット後を指す
git clone https://github.com/awslabs/aidlc-workflows.git
cd aidlc-workflows && git checkout c03f9e28

# リリース済みの版を見るならタグで固定する（現 Latest は v2.9.0）
# git clone --depth 1 --branch v2.9.0 https://github.com/awslabs/aidlc-workflows.git

# 上流の現在を追うなら main（動くブランチなので、本ノートの数値と食い違いうる）
# git clone --depth 1 --branch main https://github.com/awslabs/aidlc-workflows.git

ls harness/  # claude  codex  copilot  cursor  kiro  kiro-ide  opencode  ← ハーネスは 7 種
ls assets/   # AI-DLC-Workflows-2.0-Specification.pdf（ハイフン区切りの名前）
```

> **Spec PDF**: 2.6.2 時点で PDF は 2 箇所にあり、**ファイル名が異なる**。
> `assets/AI-DLC-Workflows-2.0-Specification.pdf`（ハイフン区切り）と
> `dist/AI-DLC Workflows 2.0 Specification.pdf`（空白区切り）。
> **2.7.2 で `dist/` 側は消えたため、現在の所在は `assets/` のみである。**
> なお PDF の内容が 33 ステージ構成に更新されているかは**未確認**。

---

> **上流 2.6.2 → 2.6.49 の差分**: [13-release-impact-2649.md](./13-release-impact-2649.md)
> **上流 2.6.49 → 2.6.55 の差分**: [14-release-impact-2655.md](./14-release-impact-2655.md)
