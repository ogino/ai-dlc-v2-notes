# 06. ハーネスと導入

## 6.1 対応ハーネス（v2）

| Harness | 最低バージョン目安 | コピー元 | 起動 |
|---------|-------------------|----------|------|
| **Claude Code** | 最新推奨 | `dist/claude/` | `/aidlc` |
| **Kiro IDE** | hooks v2 対応含む | `dist/kiro-ide/` | `/aidlc` |
| **Kiro CLI** | ≥ 2.6 | `dist/kiro/` | `/aidlc` |
| **Codex CLI** | ≥ **0.145.0** | `dist/codex/` | `$aidlc` |
| **opencode** | ≥ 1.17 | `dist/opencode/` | `/aidlc` |

決定論エンジン（state machine・audit・並列の審判）はハーネス横断で同一。違うのはシェル（skills/hooks の載せ方）。

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

### モデル／認証（出荷既定 ≠ 全ハーネス必須）

公式 README は Bedrock の準備に触れるが、**「どのハーネスでも Bedrock アカウント必須」ではない**。

| ハーネス | 目安 |
|----------|------|
| **Claude Code** | 出荷設定は Bedrock。モデルアクセス有効化 + AWS SDK 資格情報が実質必要 |
| **Codex CLI** | 出荷 `config.toml` は Bedrock ブロック。OpenAI 認証等への代替はガイド参照 |
| **Kiro IDE / CLI** | Kiro サインイン + セッションで選ぶモデル（≥2.6 等は公式 README の Kiro CLI 要件） |
| **opencode** | グローバル opencode 設定のプロバイダ／モデル |

---

## 6.3 インストール要点

### Claude Code

```bash
cp -r dist/claude/.claude/ your-project/.claude/
cp -r dist/claude/aidlc/   your-project/aidlc/
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

### Codex CLI

```bash
cp -r dist/codex/.codex/  your-project/.codex/
cp -r dist/codex/.agents/ your-project/.agents/
cp -r dist/codex/aidlc/   your-project/aidlc/
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

### opencode

```bash
cp -r dist/opencode/.aidlc/    your-project/.aidlc/     # エンジン（.opencode 外）
cp -r dist/opencode/.opencode/ your-project/.opencode/  # ネイティブ殻
cp -r dist/opencode/aidlc/     your-project/aidlc/
cp dist/opencode/opencode.json your-project/opencode.json
cp dist/opencode/AGENTS.md     your-project/AGENTS.md
```

理由: opencode は `.opencode/tools/*.ts` をカスタムツールとして自動 import するため、エンジンをそこへ置くとクラッシュする。

---

## 6.4 よく使うコマンド

| コマンド | 意味 |
|----------|------|
| `/aidlc --doctor` | 環境・設定ヘルスチェック |
| `/aidlc --doctor --export` | 秘匿化済み診断レポート（2.5.2+） |
| `/aidlc --status` | 進捗 |
| `/aidlc <description>` | ワークフロー開始（scope 自動） |
| `/aidlc bugfix ...` | 明示 scope |
| `/aidlc compose "..."` | カスタム計画 |
| `/aidlc --scope X --stage Y` | ジャンプ |
| `/aidlc config list` | depth / test-strategy |
| `/aidlc space` / `intent` | ワークスペース操作 |
| `/aidlc plugin list|sync` | プラグイン |

Codex は `$aidlc` 表記。

### 直接ツール呼び出し（`/aidlc` サブコマンドではない）

次の 2 つは `/aidlc <x>` の形を取らず、ツールを直接起動する。

| コマンド | 意味 |
|----------|------|
| `bun <harness-dir>/tools/aidlc-utility.ts codekb-scope-diff --repo <repo>` | Reverse Engineering 再実行前に codekb ストアの鮮度を確認（`NO_STORE` / `CURRENT` / `STALE` / `UNVERIFIED` / `UNKNOWN_SCOPE`）。2.5.35+ |
| `bun <harness-dir>/tools/aidlc-workspace-sync.ts [--force]` | 任意の `repos.json` に基づき不足リポジトリを clone、管理対象 `.gitignore` を更新、VSCode マルチルート生成。2.5.36+ |

> `--doctor` は 2.5.36 で advisory 行が 3 つ増えた（`aidlc/` 配下の未コミット変更、`repos.json` とディスク上 sibling の drift、管理対象 `.gitignore` ブロックの陳腐化）。後者 2 つは `repos.json` が存在する場合のみ表示される。

---

## 6.5 トラブルシュート（頻出）

| 症状 | 対処 |
|------|------|
| 端末では bun が見えるがハーネスが見えない | 非対話 PATH（`~/.zshenv` 等） |
| Codex doctor が version 不足 | ≥ 0.145.0 |
| Bedrock AccessDenied（Claude/Codex 出荷設定） | モデル有効化 + 資格情報 + region |
| Codex hooks が動かない | §6.3 の trust（TUI または config.toml へ TOML 反映） |
| 新 dist をコピーしたが反映されない | **新セッション**起動 |
| Kiro IDE hooks 無反応 | v2 schema hooks の正しい中身コピー（2.5.10） |

---

## 6.6 ソースの確認方法

```bash
git clone --depth 1 --branch v2 https://github.com/awslabs/aidlc-workflows.git
cd aidlc-workflows

ls dist/     # claude  codex  kiro  kiro-ide  opencode  plugins
ls assets/   # AI-DLC-Workflows-2.0-Specification.pdf（公式パス）
```

> **Spec PDF**: リポジトリ上の正は `assets/AI-DLC-Workflows-2.0-Specification.pdf`。`dist/` 配下に同名が同梱される場合もあるが、リンク・引用は `assets/` を使う。
