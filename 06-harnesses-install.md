# 06. ハーネスと導入

## 6.1 対応ハーネス（v2）

| Harness | 最低バージョン目安 | 導入元 | 起動 |
|---------|-------------------|--------|------|
| **Claude Code** | 最新推奨 | `dist/claude/` をコピー | `/aidlc` |
| **Kiro IDE** | hooks v2 対応含む | `dist/kiro-ide/` をコピー | `/aidlc` |
| **Kiro CLI** | ≥ 2.6 | `dist/kiro/` をコピー | `/aidlc` |
| **Codex CLI** | ≥ **0.145.0** | `dist/codex/` をコピー | `$aidlc` |
| **Cursor** | 明示の最低版なし（上流は cursor-agent **2026.07** で検証） | **コピーではなく同梱インストーラ実行**: `bun dist/cursor/install.ts <project>` | `/aidlc` |
| **opencode** | ≥ 1.17 | `dist/opencode/` をコピー | `/aidlc` |
| **GitHub Copilot** | CLI ≥ **1.0.74** / VS Code ≥ **1.130**（2026-07-22 リリース） | `dist/copilot/` をコピー | `/aidlc` |

**計 7 種**（`ls harness/` = claude / codex / copilot / cursor / kiro / kiro-ide / opencode）。
決定論エンジン（state machine・audit・並列の審判）はハーネス横断で同一。違うのはシェル（skills/hooks の載せ方）。

> **Cursor は 2.5.63 で追加**（IDE と CLI `agent` の両方を 1 つの `.cursor/` で兼ねる）。
> **7 種のうち Cursor だけ導入形態が違う**。他の 6 種は `dist/<harness>/` を `cp` するのに対し、
> Cursor は配布物に同梱された**インストーラ `dist/cursor/install.ts` を bun で実行**する
> （`bun dist/cursor/install.ts <project>`）。
> インストーラはプロジェクト所有ファイルとの衝突を拒否し、`.cursor/.gitignore` と既存の method memory を保全、
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

1. **bun** を PATH に入れる（非対話シェルからも見えること）  
   zsh なら `~/.zshenv` にも `BUN_INSTALL`/`PATH` を書く必要がある場合あり
2. 推奨モデル: **Claude Opus 4.8**（公式 README。Kiro では有料プランが必要な場合あり）

```bash
curl -fsSL https://bun.sh/install | bash
git clone https://github.com/awslabs/aidlc-workflows.git
cd aidlc-workflows && git checkout v2
```

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

### Claude Code

```bash
cp -R dist/claude/.claude/. your-project/.claude/
cp -R dist/claude/aidlc/.   your-project/aidlc/
cd your-project && claude
# /aidlc --doctor
# /aidlc <description>
```

### Kiro IDE / CLI

**重要**: 既存 `.kiro` がある場合は **中身コピー**しないとネストする。

```bash
mkdir -p your-project/.kiro your-project/aidlc
cp -R dist/kiro-ide/.kiro/. your-project/.kiro/     # IDE
cp -R dist/kiro-ide/aidlc/. your-project/aidlc/
cp dist/kiro-ide/AGENTS.md your-project/AGENTS.md
```

CLI は `dist/kiro/` を同様に。  
`aidlc/` は `.kiro/` の**兄弟**（内側ではない）。

### GitHub Copilot（2.5.60 で追加）

**他ハーネスと違い、コピーだけでは動かない。folder trust の設定が要る。**

```bash
mkdir -p your-project/.aidlc your-project/aidlc your-project/.github
cp -R dist/copilot/.aidlc/.  your-project/.aidlc/
cp -R dist/copilot/aidlc/.   your-project/aidlc/    # .aidlc/ の兄弟（内側ではない）
cp -R dist/copilot/.github/. your-project/.github/  # マージ。すべて aidlc- 接頭辞なので既存は上書きされない
cp dist/copilot/AGENTS.md    your-project/AGENTS.md # 既存があればマージ（@-import ブロックは残す）
```

その後:

1. **プロジェクトを信頼する**（必須）。`copilot` を対話起動して trust プロンプトを承認するか、
   `~/.copilot/config.json` の `trustedFolders` にプロジェクトの絶対パスを追加する。
   **未信頼だとリポジトリフックが 1 本も動かない**
2. ヘッドレス（`copilot -p`）で使う場合は `GITHUB_COPILOT_PROMPT_MODE_REPO_HOOKS=1`
3. `AGENTS.md` の「Git Integration」節の `.gitignore` 記述を適用してからワークフローを開始する

配置先: `/aidlc` とステージ／スコープランナーは `.github/skills/`、
14 ペルソナは `.github/agents/`、フック定義は `.github/hooks/aidlc.json`。

### Codex CLI

```bash
cp -R dist/codex/.codex/.  your-project/.codex/
cp -R dist/codex/.agents/. your-project/.agents/
cp -R dist/codex/aidlc/.   your-project/aidlc/
cp dist/codex/AGENTS.md   your-project/AGENTS.md
# プロジェクトは git repo であること（Codex が .codex/hooks.json を発見する条件）
```

**hooks の trust（未信頼だと hooks が動かない）** — どちらか:

1. **TUI で "Trust all"**（初回セッション）
2. **生成した trust エントリを `$CODEX_HOME/config.toml` に反映**:
   ```bash
   # AI-DLC ソース checkout 側（dist を出したリポジトリ）で
   bun install --frozen-lockfile
   bun scripts/package.ts codex trust --project <プロジェクトの絶対パス>
   # 出力の [hooks...] TOML を $CODEX_HOME/config.toml にマージ
   # （同一 hook path の既存エントリは置換。二重追記で TOML が壊れる）
   ```

詳細は公式 `docs/guide/harnesses/codex-cli.md`。

> **trust テーブルは「初回だけ」ではない（2.6.44）。** 上の手順は初回導入時のものだが、
> 2.6.44 で `request_user_input` を拾う新しい PostToolUse 登録が
> **`.codex/hooks.json` の PostToolUse 配列の先頭に挿入された**。
> trust エントリは `post_tool_use:N` という**位置インデックス**で hook を指すため、
> 先頭挿入によって**既存エントリのインデックスが全部 1 つずつズレる**。
> `dist/codex/` を再コピーしただけでは、古い trust テーブルが別の hook を指したままになる。
> アップグレード時は次を行うこと:
>
> ```bash
> bun scripts/package.ts codex trust --project "<プロジェクトの絶対パス>"
> # 出力の TOML で $CODEX_HOME/config.toml の既存 AI-DLC trust エントリを「差し替える」
> # （追記ではない。同一 hook path の古いエントリは消す）
> ```
>
> そのうえで**新しい Codex セッションを開始する**。

### Cursor（2.5.63 で追加）

**他ハーネスと違い、`cp` ではない。同梱インストーラを bun で実行する。**

```bash
bun dist/cursor/install.ts your-project
# 検証（プロジェクトルートに入ってから相対パスで実行）
cd your-project
bun .cursor/tools/aidlc-utility.ts doctor
```

その後、`your-project/` を Cursor IDE で開く（または中で `agent` を起動する）と `/aidlc` が使える。

- IDE と CLI（`agent`）は**同じ `.cursor/` を読む**ので、インストールは 1 回でよい
- `aidlc/` は `.cursor/` の**兄弟**。エンジンが読む `aidlc/spaces/default/memory/` のメソッドツリーが同梱されており、
  これが無いと `/aidlc --doctor` の "workspace shell ready" チェックが落ちる
- 再実行は**アップグレード**として働き、フレームワーク管理ファイルだけを更新して active-space や
  プラグイン選択状態は保つ。管理下のどのパスを保全したかはインストーラが出力する
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
> 自分の版を照合するときは `core/tools/aidlc-version.ts` を見ること。）
> `failClosed: true` の下では空 stdout が不正 JSON と扱われるため、**Cursor IDE ではあらゆるツール呼び出しが
> ブロックされた**（CLI は沈黙を allow と解釈したため無症状）。**2.5.69 で修正済み**。
> 該当版を入れている場合は `dist/cursor/` を更新して `bun dist/cursor/install.ts <project>` を再実行する。

### opencode

```bash
cp -R dist/opencode/.aidlc/.    your-project/.aidlc/     # エンジン（.opencode 外）
cp -R dist/opencode/.opencode/. your-project/.opencode/  # ネイティブ殻
cp -R dist/opencode/aidlc/.     your-project/aidlc/
cp dist/opencode/opencode.json your-project/opencode.json
cp dist/opencode/AGENTS.md     your-project/AGENTS.md
```

理由: opencode は `.opencode/tools/*.ts` をカスタムツールとして自動 import するため、エンジンをそこへ置くとクラッシュする。

---

## 6.4 アップグレードでハーネスごとにすること（2.6.2 → 2.6.55）

> **⚠ 2.6.55 → 2.6.123 では、再コピーだけでは済まない作業が過去最多になった。**
> 本節の表は **2.6.55 までの手順**である。2.6.123 へ上げるときは、下の 5 点を追加で行うこと
> （根拠と詳細は [15-release-impact-26123.md](./15-release-impact-26123.md)）。
>
> | # | 版 | 追加で要る作業 | 対象 |
> |:-:|---|---|---|
> | 1 | **2.6.64** | **古いファイルの手動削除**（上流「An overlay copy cannot delete retired files.」）<br>`rm -f <proj>/.kiro/agents/aidlc.json`<br>`rm -f <proj>/.kiro/agents/aidlc-*-agent.json`<br>`rm -f <proj>/.kiro/settings/cli.json` | **Kiro IDE** |
> | 2 | **2.6.110** | `dist/` 上書きでプラグイン合成が黙って消える。**`/aidlc plugin sync` を実行**（`--doctor` の `Composed plugin surface` が exit 1 で検出）。Claude / Codex / Cursor / Kiro IDE は次セッションで自己修復するが **Kiro CLI は明示実行が必須** | プラグイン利用者 |
> | 3 | **2.6.71** | 非英語で使う場合、**`aidlc/spaces/<space>/memory/org.md` を手編集**して新規セッションを開始する。既存ワークスペースの `org.md` は再コピーで上書きされない | 日本語利用者 |
> | 4 | **2.6.82** | Claude プロジェクトフックを `/hooks` で承認し、**Claude Code を完全に再起動**する | Claude Code |
> | 5 | **2.6.91** | CodeKB scope fingerprint の修正。**空木ハッシュで保存されたタイムスタンプは再採番が要る** | CodeKB 利用者 |
>
> 2.6.64 は 13 章で扱った 2.6.47 と**同種の作業**である（→ [15.8](./15-release-impact-26123.md#158-その他の主題)）。


**「`dist/` を再コピーすれば済む」ハーネスと、追加の手作業が要るハーネスがある。**
ただし **2.6.51 以降は「再コピー」そのものに全ハーネス共通の作法が付いた**ので、
ハーネス別の表を読む前に次を満たすこと。

### 全ハーネス共通のスワップ手順（2.6.51 以降の前提）

2.6.51 で継続カーソルが 7 ハーネスに一般化されたことにより、`dist/` の入れ替えは
**「ファイルを上書きするだけ」の操作ではなくなった**。以下の 4 点はハーネスによらず一律に掛かり、
下の表の **○** も「これを満たしたうえで」の意味になる。

1. **静止状態で行う。** AI-DLC のコマンドが 1 つも走っておらず、フックも発火していない瞬間に
   交換を完了させる（上流原文: `in one quiescent swap (no AI-DLC command or hook running)`）。
2. **部分コピーせず、`dist/<harness>/` を全ツリーで入れ替える。** 新旧混在は非サポート
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
| `bun <harness-dir>/tools/aidlc-workspace-sync.ts [--force]` | 任意の `repos.json` に基づき不足リポジトリを clone、管理対象 `.gitignore` を更新、VSCode マルチルート生成。2.5.36+ |

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
| 新 dist をコピーしたが反映されない | **新セッション**起動 |
| Copilot: アップグレード後に進行中ワークフローが進まない／古い挙動をする | **`next` を打つ**（2.6.51 以降。7 ハーネス共通の手順で、Copilot 固有ではない。2.6.12 当時の手当ては「新しい会話を開始」だった。下記） |
| Kiro CLI で `/aidlc --status` 等が無反応（silent no-op） | 2.6.46 の verb interceptor 修正。`dist/kiro/` を再コピー（**Kiro CLI のみの修正**） |
| Kiro: プラグインの compose がアップグレード後に走らない | 2.6.47。projection を再ビルド／再コピーし、**CLI は** `aidlc plugin sync` か `hooks/compose.ts` を明示実行（**IDE は不要**）。§6.4 |
| Kiro IDE hooks 無反応 | v2 schema hooks の正しい中身コピー（2.5.10） |
| Cursor IDE で全ツール呼び出しがブロックされる | 2.5.63〜2.5.68 の既知不具合（allow JSON 未出力 × `failClosed`）。**2.5.69 以降**へ更新し `bun dist/cursor/install.ts <project>` を再実行 |
| 学習 persist が `selections-json is malformed: missing or non-string space` で落ちる | 2.6.36 の非互換。該当ステージの **`surface` を再実行**して selections を作り直す（`persist` のリトライでは直らない）。§6.4 |

### GitHub Copilot: アップグレード後は進行中ワークフローを新しい会話で継続する（2.6.12）

2.6.12 で Copilot は、直近に配信した AI-DLC ディレクティブを **Copilot 所有のアトミックなエンジンカーソル**で保持し、Stop からの継続とリプレイ拒否をそこで判定する設計になった。カーソルは提示トークンのダイジェストを丸ごと比較してから後続トークンを公開する仕組みで、**アップグレード前の転送マーカーは形式が合わず再利用できない**（欠落・不正・v1・陳腐化したコンテキストは従来のステートレス経路に落ちる）。

そのため、**進行中のワークフローを抱えたまま `dist/copilot/` を更新した場合は、新しい Copilot の会話を開始してから続きを進めること。** 同じ会話を続けると、アップグレード前のマーカーを引きずった状態で再開しようとすることになる。

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

```bash
git clone --depth 1 --branch v2 https://github.com/awslabs/aidlc-workflows.git
cd aidlc-workflows

ls dist/     # claude  codex  copilot  cursor  kiro  kiro-ide  opencode  plugins
             # ＋ "AI-DLC Workflows 2.0 Specification.pdf"（空白区切りの名前）
ls harness/  # claude  codex  copilot  cursor  kiro  kiro-ide  opencode  ← ハーネスは 7 種
ls assets/   # AI-DLC-Workflows-2.0-Specification.pdf（ハイフン区切りの名前）
```

> **Spec PDF**: 2.6.2 時点で PDF は 2 箇所にあり、**ファイル名が異なる**。
> `assets/AI-DLC-Workflows-2.0-Specification.pdf`（ハイフン区切り）と
> `dist/AI-DLC Workflows 2.0 Specification.pdf`（空白区切り）。**リンク・引用は `assets/` を正とする。**
> なお PDF の内容が 33 ステージ構成に更新されているかは**未確認**。

---

> **上流 2.6.2 → 2.6.49 の差分**: [13-release-impact-2649.md](./13-release-impact-2649.md)
> **上流 2.6.49 → 2.6.55 の差分**: [14-release-impact-2655.md](./14-release-impact-2655.md)
