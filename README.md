# AWS AI-DLC Workflows 2.0 整理ノート（非公式）

> **免責（必読）**  
> - 本リポジトリは **非公式の二次整理** であり、AWS / awslabs / Amazon の公式文書・公式サポートではない  
> - 生成 AI による要約・再構成を含み、**誤りがあり得る**  
> - 実装・仕様の正は常に上流 [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows)（`main` ブランチ）のソースと `docs/` を参照すること  
> - 本リポジトリの文章のライセンスは **MIT**（`LICENSE`）。上流実装のライセンスは **MIT-0**（別物）

初回調査日: 2026-07-28（実装バージョン 2.5.11）  
最終同期日: 2026-09-24（タグ `v2.10.0` で測定）  
対象実装: [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows) **リリース `v2.10.0`**（`2a883858`。取得日 2026-09-24。同日の `main` は `20008a5d` でタグから +1、テストのみ）

> **⚠ 上流の `dist/` ディレクトリは 2026-09-08 に削除された。**
> 導入はネイティブインストーラ（`install.sh` / `install.ps1`）と `aidlc config` に変わった。
> **`cp -R dist/<harness>/` は上流の公式手順ではなくなった。**
> 経緯と影響は [17-release-impact-2801.md](./17-release-impact-2801.md) を参照。
>
> **📌 続報（2026-09-17）—— 上記の 🔴 は解消済み。**
> `52da70ad` を含む **`v2.8.1` が 2026-09-09 20:12 UTC にリリース**された（本ノートの測定の数時間後）。
> その後 **v2.8.2（09-11）・v2.9.0（09-15）・v2.10.0（09-24、現 Latest）** が公開されている。
> **Copilot / Cursor を使う場合も、v2.8.1 以降を導入すれば不具合は起きない。**
> **17 章が案内するソース生成の暫定回避策は不要である。**
> 2.8.1 → 2.9.0 の差分は [18 章](./18-release-impact-290.md)、2.9.0 → 2.10.0 は [19 章](./19-release-impact-2100.md) で扱う。

> **⚠ 版を固定するなら `v2.10.0`（現 Latest）を使う。**
> **`v2.8.1` / `v2.8.2` / `v2.9.0` / `v2.10.0` はいずれも実在するタグである**
> （旧記述の「`v2.8.1` タグは存在しない」は 2026-09-09 時点の話で、現在は誤り）。
>
> **⚠ 17 章の基準 `c03f9e28` は、リリース版 `v2.8.1`（＝ `215afe1a`）ではない。**
> どちらも `AIDLC_VERSION` は `2.8.1` だが、タグは 5 コミット後を指す。
> ソースを照合する目的なら SHA を、導入するならタグを使うこと（→ [18.1](./18-release-impact-290.md)）。

> **✅ v2.9.0 に残っていたセンサー・フック系の不具合 3 件（#1070 / #1166 / #1249）は、すべて `v2.10.0` に収録された（2026-09-24）。**
> **v2.10.0 に上げれば、18.6 が案内していた回避策（手動コピー経路で導入し、フックの PATH に `bun` を置く）は不要になる。**
> v2.9.0 に留まる場合は [18.6](./18-release-impact-290.md) の警告が引き続き当てはまる（→ [19.4](./19-release-impact-2100.md)）。
>
> **⚠ ただし `v2.10.0` にも別の既知の不具合がある（2026-09-25 追記）** —— **ネイティブ導入でチーム担当（`Unit Ownership: team`）の Construction が始まらない**（#1286。修正 `057b13be` は未リリース）。単独運用と手動コピー（Bun）経路は影響しない（→ [19.4](./19-release-impact-2100.md)）。

> **🔴 2.10.0 への更新は作業が要る（2.9.0 と同じ形）。導入経路によって手順が違う。**
>
> | 導入経路 | 手順 |
> |---|---|
> | **ネイティブ導入** | `aidlc update` の後、**プロジェクトごとに `aidlc config --yes`** |
> | **手動コピー運用** | **`aidlc-copy-runtime-2.10.0.tar.gz`** を取得し、ハーネス管理ディレクトリを更新し、ルートのファイルと `aidlc/`（利用者の記憶）は保全する。**丸ごと上書きしてはいけない**（手順は [6 章の手動コピー節](./06-harnesses-install.md#手動でファイルを置きたい場合)）。**`aidlc` コマンドは使わない**（そもそも存在しない） |
>
> **手動コピー運用者は `aidlc update` も `aidlc config --yes` も実行できない。**
> ネイティブバイナリを入れていないためである（→ [18.4](./18-release-impact-290.md) / [18.5](./18-release-impact-290.md)）。
>
> **⚠ Bedrock を出荷既定のまま使っているプロジェクトは、`aidlc config --yes` で Bedrock 設定が外れうる**（→ [19.7](./19-release-impact-2100.md)）。
> **⚠ 1 つのプロジェクトに置けないハーネスの組み合わせがある**（Kiro CLI と Kiro IDE、OpenCode と GitHub Copilot。→ [19.6](./19-release-impact-2100.md)）。

> **🔴 Change Control は v2.10.0 で Guard Policy に改名され、`relaxed` の意味が広がった。**
> 既定スコープ `classic` を含む 8 スコープ（既定 `relaxed`）では、**承認後に計画を編集しても再承認を求められず、レビュー後の成果物の書き換えも拒否されない**（監査行 `GUARD_STOOD_ASIDE` が残る）。
> **初回の Plan Approval は引き続き必須**である。組織として締めたいならメモリの `## Guard Policy` に `Mode: strict` を書く（→ [19.3](./19-release-impact-2100.md)）。
>
> **🔴 既存ワークフローは Guard Policy（旧 Change Control）が暗黙に `strict` になる。**
> `Guard Policy` 行（旧 `Change Control` 行）を持たない既存の `aidlc-state.md` は、スコープ既定ではなく **`strict`** として扱われる。
> **doctor にも診断にも該当する finding が無く、気付く手段が用意されていない。**
> 既存 intent には一度 `/aidlc --guard-policy <strict|relaxed|off>`（v2.9.0 では `--change-control <strict|relaxed>`）を実行して意図を固定すること（→ [18.3.1](./18-release-impact-290.md)）。

> **🔴 既定スコープ `classic` が 26 → 18 ステージに縮小した（v2.9.0、破壊的）。**
> **効くのは、実際に `classic` に解決される intent である** —— `/aidlc-init` や `--scope` 無しの `intent-create` などのフォールバック経路と、`classic` を明示した場合。
> 対話的な `/aidlc <説明文>` でキーワードに当たらなければ compose 提案が先に出るので、黙って `classic` が始まるわけではない（→ [5.2.1](./05-scopes-depth-test.md)）。
> 旧 classic の形が要るなら **`workshop`** を使う（→ [18.2](./18-release-impact-290.md)）。
>
> **⚠ 上流の `v2` ブランチは 2026-09-01 に削除された。**`main` が 2.x の正本である（旧 1.x は新設の `v1` ブランチへ移動）。
> 経緯と影響は [16-release-impact-2700.md](./16-release-impact-2700.md) を参照。

> 上流はマイナーリリースが頻繁である。本ノートは特定時点のスナップショットであり、
> 数値・仕様は参照時に、**上流リポジトリ**（`awslabs/aidlc-workflows` `main` のローカル clone）の `core/tools/aidlc-version.ts` と CHANGELOG で必ず照合すること。本ノートのリポジトリには `core/` は存在しない。

---

## このノートの目的

インターネット上の公式・解説情報と、上流のソース／ドキュメントを突き合わせ、**AI-DLC（AI-Driven Development Life Cycle）Workflows 2.0** を日本語で俯瞰できるようにしたものです。

- **方法論（methodology）**: AWS が定義した AI 中心の開発ライフサイクル
- **実装（implementation）**: `aidlc-workflows` の `main` ブランチが提供する、複数 CLI ハーネス向けのネイティブ実行エンジン

---

## 一言で言うと

AI-DLC 2.0 は、**「プロンプトを投げて祈る」アドホックな AI コーディングを、検証可能で自己修正するエンジニアリング・ワークフローに変える**枠組みです。

- **承認ゲートでは人が最終判断**する（レビュアは最終拒否権を持たない）
  - 例外: Initialization はゲートなし／フェーズ境界の Verification Gate は自動／Construction で ladder 後に **autonomous** を選ぶと以降の Construction ステージの承認ゲートは省略可（失敗時は halt-and-ask）。**「Bolt」は 2.6.86 で上流が「計画上のスプリント様スライス」へ再定義した** → [01.8.1](./01-overview.md#181-bolt-の定義は-2686-で上流が書き換えた)
- 決定論的エンジンがルーティングし、LLM 導体（conductor）が実行品質を担う
- 1 つの `core/` から Claude Code / Kiro IDE / Kiro CLI / Codex / **Cursor** / opencode / **GitHub Copilot** 向け配布物を生成する

---

## ドキュメント一覧

### 公開向け（読む順番の目安）

| ファイル | 内容 |
|----------|------|
| [01-overview.md](./01-overview.md) | 背景・方法論・2.0 GA の位置づけ |
| [02-architecture.md](./02-architecture.md) | リポジトリ構成・Engine/Conductor・平面モデル |
| [03-phases-and-stages.md](./03-phases-and-stages.md) | 5 フェーズ / 33 ステージの全体像 |
| [04-agents.md](./04-agents.md) | 14 エージェント体制 |
| [05-scopes-depth-test.md](./05-scopes-depth-test.md) | 11 スコープ・深度・テスト戦略・Composer |
| [06-harnesses-install.md](./06-harnesses-install.md) | 対応ハーネスと導入手順の要点 |
| [07-learning-loop-state.md](./07-learning-loop-state.md) | Space/Intent・Rules・Sensors・監査 |
| [08-v1-vs-v2.md](./08-v1-vs-v2.md) | 1.x 系と 2.0 の差分 |
| [09-references.md](./09-references.md) | 参照リンク・上流リポジトリ内パス |
| [10-release-impact-2537.md](./10-release-impact-2537.md) | 2.5.11 → 2.5.37 の差分／ソース読解で分かった挙動 |
| [11-release-impact-2562.md](./11-release-impact-2562.md) | 2.5.37 → 2.5.62 の差分。**監査 74→82**、フック 7 本改名、GitHub Copilot ハーネス追加 |
| [12-release-impact-2602.md](./12-release-impact-2602.md) | 2.5.62 → 2.6.2 の差分。**ステージ 32→33**、`application-design` 廃止と `domain-design` / `contract-design`、成果物名の作り直し、state スキーマ v8、Cursor ハーネス追加 |
| [13-release-impact-2649.md](./13-release-impact-2649.md) | 2.6.2 → 2.6.49 のリリース差分 |
| [14-release-impact-2655.md](./14-release-impact-2655.md) | 2.6.49 → 2.6.55 のリリース差分。**中核メトリクスは全項目不変**で、変わったのは実行時のガード・継続トークン・監査の発火条件 |
| [15-release-impact-26123.md](./15-release-impact-26123.md) | 2.6.55 → 2.6.123 のリリース差分。**フック 17→18 / `core/tools/*.ts` 41→51 / 監査 86→91 / `bugfix` 7→9・`refactor` 8→10**、プラグイン作成ツールチェーン、Bolt 用語の再定義 |
| [16-release-impact-2700.md](./16-release-impact-2700.md) | 2.6.123 → 2.7.0 のリリース差分。**中核メトリクスは全項目不変**。上流の `v2` ブランチ削除と `main` への一本化、2.6.124 の状態ファイル相対パス化、**2.7.0 の CHANGELOG がロールアップ再掲である**こと |
| [17-release-impact-2801.md](./17-release-impact-2801.md) | 2.7.0 → 2.8.1 のリリース差分。**上流から `dist/` が消えネイティブ配布へ**、`aidlc` CLI と設定階層の新設、ガードレール 9 種の設定ファイル記録、2.7.1 の Plan Approval デッドロック修正、**測定時点で 2.8.1 が未リリースだった**こと（→ 18 章で解消） |
| [18-release-impact-290.md](./18-release-impact-290.md) | 2.8.1 → 2.9.0 のリリース差分。**既定スコープ `classic` が 26 → 18**（破壊的）、**Change Control**（v2.8.1 出荷済み）と既存レコードが暗黙に `strict` になる件、**手動コピー用アセットの分離**、監査イベント 91 → 99、**v2.9.0 に残っていたセンサー・フック系の不具合 3 件（①②はネイティブ導入限定、③は Bun 経路でもフックの PATH 次第で起こる）**（うち 1 件は当時 preview でも未修正だった。**3 件とも v2.10.0 で解消** → 19 章） |
| [19-release-impact-2100.md](./19-release-impact-2100.md) | 2.9.0 → 2.10.0 のリリース差分（**タグ間で測定**）。**Change Control → Guard Policy** と `relaxed` の意味の拡大（初回 Plan Approval は必須のまま）、**18 章の不具合 3 件の解消**、Construction の新既定、複数ハーネスの共存と組み合わせ制限、Bedrock 出荷既定の撤去、**リリース時フルテストの再撤去**、監査イベント 99 → 105 |
| [SOURCES.md](./SOURCES.md) | 調査ソース一覧・免責 |

### メンテナ向け（作業記録）

| ファイル | 内容 |
|----------|------|
| [REVIEW-8AI.md](./REVIEW-8AI.md) | 複数 AI レビュー結果 |
| [CONVERSATION_LOG.md](./CONVERSATION_LOG.md) | セッション要約 |
| [docs/](./docs/) | 開発ルール・残タスク・会話アーカイブ・追跡性 |

---

## 主要メトリクス（v2 実装）

| 項目 | 値 |
|------|-----|
| フェーズ | 5（Initialization / Ideation / Inception / Construction / Operation） |
| ステージ | 33 |
| エージェント | 14（ドメイン 11 + レビュア 2 + Composer 1） |
| スコープ | 11 + 自動検出 + カスタム compose |
| 深度 / テスト戦略 | 各 3 段階（独立） |
| 監査イベント種別 | **105**（タグ `v2.9.0` は 99、基準 `c03f9e28` は 91、タグ `v2.8.1` は 95）※ |
| 対応ハーネス | Claude Code, Kiro IDE, Kiro CLI, Codex CLI, **Cursor**, opencode, GitHub Copilot（計 7 種） |
| 実装バージョン | **2.10.0**（タグ `v2.10.0` = `2a883858`。取得日 2026-09-24）※現 Latest は `v2.10.0` |
| 上流実装のライセンス | MIT-0（`aidlc-workflows`） |
| 本ノートのライセンス | MIT（本リポジトリ `LICENSE`） |

※ 監査カテゴリ数は正典レジストリ `core/knowledge/aidlc-shared/audit-format.md` の Event Registry 見出し基準で **25**（形式見出し 3 本は分類に数えない）。**基準 `c03f9e28` は 22、タグ `v2.8.1` は 23**（Change Control が加わった）、**2.9.0 で 25**（Ceremony と Commit Provenance が加わった）。**2.10.0 も 25**（イベントは 6 種増えたが分類は増えていない）。
※ **`docs/reference/12-state-machine.md` 基準では 20 分類**（基準 `c03f9e28` では 19）。イベント種別の集合自体は両出典で同一で、**分類数が違うのはグルーピングの粒度の差である**。どちらを引用するかは出典を明記すること。
※ **`Interaction Events` は見出しが宣言する件数と表の行数が 1 件ずれている**（基準 `c03f9e28`: 宣言 10 / 行 9、**タグ `v2.8.1` 以降は宣言 11 / 行 10**）。**⚠ この齟齬は `v2.10.0` で解消された**（`261083ce` / #1150 で宣言 13 / 行 13 に是正。初出は preview `v2.9.1-preview.20260920.1`）。**`v2.9.0` は 11 / 10 のままである。**

---

## クイック開始（概念）

```bash
# 1. ネイティブ aidlc を導入（bun / Node.js は不要）
#    macOS / Linux / WSL
curl -fsSL https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.sh | sh
#    Windows PowerShell
#    irm https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.ps1 | iex
#    版を固定する場合は install.ps1 を保存してから: .\install.ps1 -Version 2.10.0

# 2. PATH を通す（インストーラは子シェルで走るため、親シェルには反映されない）
#    インストーラが表示する手順に従うか、新しいシェルを開く
export PATH="$HOME/.local/bin:$PATH"   # Unix の既定。$AIDLC_BIN_DIR を変えた場合はそのパス
aidlc version                          # ここで版が出れば導入成功

# 3. プロジェクトに導入（dist のコピーはもう要らない）
cd your-project
aidlc config --harness claude
aidlc doctor

# 4. セッション内 — ワークフロー開始
/aidlc Build a task management API with user authentication
```

> **⚠ 上のワンライナーはダウンロードしたスクリプトを直接パイプ実行する。**
> 実行前に内容を検証したい場合は、上流のハーネス別ガイドが載せている 2 段階の手順を使う。
> どちらも上流に併存している（[17.1](./17-release-impact-2801.md#171-いちばん大きい変更は-dist-の消滅)）。
>
> ```bash
> tmp="$(mktemp -d)"
> curl -fsSL https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.sh -o "$tmp/install.sh"
> # ここで "$tmp/install.sh" を確認してから実行する
> sh "$tmp/install.sh"
> rm -rf "$tmp"
> ```

**本ノートの数値を再現・照合する場合**は、上流リポジトリを clone してソースを直接測る。
版を固定するなら **`v2.10.0`**（現 Latest）か SHA を使う。

```bash
git clone --branch main https://github.com/awslabs/aidlc-workflows.git   # 最新を追う場合
# git clone --branch v2.10.0 https://github.com/awslabs/aidlc-workflows.git  # 版を固定する場合
cd aidlc-workflows
```

**モデル／認証の前提（ハーネス別）**: **`v2.10.0` の出荷設定は Bedrock を指定しない**（Claude Code / Codex は利用者のプロバイダ設定に従う。`v2.9.0` までは両者とも Bedrock 寄りだった）。Kiro はサインインとセッションモデル、opencode はグローバル設定のプロバイダ、に依存する。詳細は [06-harnesses-install.md](./06-harnesses-install.md)。

> **⚠ `v2.10.0` で、出荷既定から Bedrock 指定が撤去された**（`c8ad4116` / #1101。初出は preview `v2.9.1-preview.20260921.1`）。`harness/claude/settings.json` の `env` は `AWS_AIDLC_DEFAULT_SCOPE` のみになり、Codex / opencode の `balanced` 階層もモデル指定が `null`（ハーネス任せ）になった。**Claude の `balanced` は引き続き `sonnet` / `medium`** である。**出荷既定のまま Bedrock を使っていたプロジェクトは `aidlc config --yes` で設定が外れうる**（→ [19.7](./19-release-impact-2100.md)）。

---

## 注意

- 生成 AI の出力は誤りを含み得る。公式も **生成物とコストのレビュー**を求めている
- 推奨モデルは公式 README 時点で **Claude Opus 4.8**（特に Kiro では有料プランが必要な場合あり）
- 本ノートは **二次整理**である。数値・手順の正本は、精読した **`main` ブランチの `docs/`・ソース・CHANGELOG** を優先する
- [2.0 Specification PDF](https://github.com/awslabs/aidlc-workflows/blob/main/assets/AI-DLC-Workflows-2.0-Specification.pdf)（リポジトリ上は主に `assets/`）は公式白書だが、**本ノート作成時は全文未精読**。PDF 固有の細部は PDF 本体を確認すること
- 公式 README の NOTE どおり、インタフェースは安定しつつ継続改善されるため、依存する場合は **既知良版の pin** を推奨
