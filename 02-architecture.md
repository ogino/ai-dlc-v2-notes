# 02. アーキテクチャ

## 2.1 設計原則: One Core, Many Harnesses

```
  手書きソース（編集する）          生成物（手編集禁止・CI でドリフト検出）
  ─────────────────────          ────────────────────────────────
  core/          方法論の正本  ──►  dist/claude/
  harness/<name>/ 薄い表面     ──►  dist/kiro-ide/
  plugins/       拡張         ──►  dist/kiro/
  scripts/package.ts ビルド    ──►  dist/codex/
                                  dist/opencode/
                                  dist/copilot/   ← 2.5.60 で追加
```

| ゾーン | 役割 |
|--------|------|
| **core/** | エンジン・32 ステージ・14 エージェント・スコープ・センサー・知識・hooks |
| **harness/** | 各 CLI への薄いアダプタ（manifest, settings, orchestrator skill） |
| **dist/** | ユーザーがプロジェクトへコピーする配布物 |
| **docs/** | User Guide / Harness Engineer Guide / Developer Reference |
| **plugins/** | 任意の追加ステージ等（例: `test-pro`） |

ビルド:

```bash
bun scripts/package.ts            # 全 dist 再生成
bun scripts/package.ts claude     # 1 ハーネスのみ
bun scripts/package.ts --check    # バイト一致ドリフトガード（CI）
```

**ルール**: 方法論の変更は `core/`（と必要なら `harness/`）のみ。`dist/` の手編集は CI 失敗。

---

## 2.2 Engine と Conductor

| 役割 | 実体 | 責務 |
|------|------|------|
| **Engine** | `aidlc-orchestrate.ts` 等の決定論的ツール | 次に何をするか（ルーティング・スコープ・ジャンプ・ゲート状態） |
| **Conductor** | `/aidlc` セッション（`SKILL.md`） | ステージをよく実行し、良い質問をし、判断を人間に提示 |

ループ:

```
Engine: next → Directive
  → Conductor: 実行（run-stage / ask / swarm / ...）
  → Conductor: report
  → Engine: 次の Directive
  → ...
  → done
```

Engine の主要サブコマンド概念: `next` / `report` / `park`  
ライフサイクル遷移は Engine 所有。導体やスクリプトが state を直接書き換えない設計。

---

## 2.3 平面モデル（Planes）

ネットワーキングの三平面にならった分離:

| 平面 | 内容 | 比喩 |
|------|------|------|
| **Control plane** | ステージ定義・Rules・Sensors のスキーマ。ワークフロー開始時にコンパイル | 経路計算（ゆっくり・賢い） |
| **Data plane** | 実際のステージ実行・Bolt・監査テレメトリ | パケット転送（高速・ルックアップ） |
| **Management plane** | `/aidlc --doctor`・監査照会・オンボーディング文書 | 運用・観測 |

含意:

- 学習ループで確定した Rule は **次ワークフローから**有効（実行中に再コンパイルしない）
- 再現性と再開時の健全性を優先

---

## 2.4 core/ の中身（ローカル調査）

```
core/
├── agents/           # 14 ペルソナ markdown
├── aidlc-common/     # conductor + stages/ + protocols/
│   ├── stages/
│   │   ├── initialization/   # 0.1–0.3
│   │   ├── ideation/         # 1.1–1.7
│   │   ├── inception/        # 2.1–2.8
│   │   ├── construction/     # 3.1–3.7
│   │   └── operation/        # 4.1–4.7
│   └── protocols/            # stage-protocol 等
├── tools/            # TypeScript CLI（エンジン本体）。出典により本数が揺れる → 下表注記
├── hooks/            # session-start, stop, sensors, reviewer-scope 等
├── scopes/           # 9 スコープ定義
├── sensors/          # claim-sources, required-sections, upstream-coverage, linter, type-check
├── knowledge/        # 方法論ナレッジ（エージェント別）
├── memory/           # フレームワーク既定の method ツリー
├── skills/           # セッション用スキル（session-cost, replay, outcomes-pack 等）
└── templates/        # AGENTS.md / CLAUDE.md 系スケルトン
```

**tools 本数（2.5.37 時点で出典差は解消。2.5.62 では `aidlc-*.ts` が 36 本＋ディスパッチャで計 37）**: 公式 README は *34 aidlc-\*.ts engine tools* と表現し、`core/tools/aidlc-*.ts` の実数も **34 本**で一致する。`.ts` 全体では 35 本だが、35 本目の `aidlc.ts` はディスパッチャ本体で `aidlc-*` パターンに含まれない。

> 2.5.11 時点では README が「25」と表記しており実数と食い違っていたが、2.5.36 で README 側が更新され解消した。本数はリリースで増減するため、参照時は `ls core/tools/aidlc-*.ts | wc -l` で実測し README 記載と照合すること。

主要ツール例:

| ツール | 役割 |
|--------|------|
| `aidlc-orchestrate.ts` | ルーティング中枢 |
| `aidlc-state.ts` | 状態ファイル |
| `aidlc-log.ts` | 監査・質問・レビュー記録 |
| `aidlc-graph.ts` | ステージグラフ compile |
| `aidlc-utility.ts` | intent-create / space / doctor 等（**`intent-birth` は 2.5.57 で `intent-create` に改名。旧名は no-op**） |
| `aidlc-bolt.ts` / `aidlc-swarm.ts` | Construction 並列 |
| `aidlc-learnings.ts` | 学習ループ |
| `aidlc-sensor-*.ts` | 決定論的検証 |
| `aidlc-steering.ts` | ルール束の配信（2.5.33 の `load-steering`） |
| `aidlc-workspace-sync.ts` | 複数リポジトリ manifest の同期（2.5.36） |

---

## 2.5 ステージ実行トポロジ（2.5.0 以降）

ステージ frontmatter の **`mode` は次の 4 値のみ**（2.5.0 以降）:

| Mode（正式値） | 通信トポロジの説明 | 代表ステージ |
|----------------|-------------------|--------------|
| **inline** | 導体がペルソナを採用し会話内で実行 | 大半（28。Init の決定論 3 段含む） |
| **subagent** | **hub-and-spoke 形状**: リード草稿 → 支援が独立 contribution → リード統合。Code Generation は支援なしの focused subagent | Practices Discovery, Code Generation |
| **pipeline** | 鎖。順に成果物を完成 | Reverse Engineering（2-link） |
| **mob** | メッシュ。リード草稿 + 並行ブラインド貢献 | User Stories |

> **用語注意**: *hub-and-spoke* は `mode` の値ではない。`mode: subagent` かつ support agents があるときの形状の説明語。CHANGELOG の要約で modes と並記されることがあるが、スキーマ上の mode は上表の 4 つ。

共通ルール:

- **委任するのは Conductor のみ**（エージェント同士は呼び出さない）
- 全員が自分の contribution を書き、**最終 `produces[]` はリードのみ**編集
- レビュアは別経路（`reviewer:` フィールド）

---

## 2.6 プロジェクト内レイアウト（実行時）

インストール後の典型:

```
your-project/
├── .claude/   または .kiro/ または .codex/ または .aidlc/ + .opencode/
│              ← エンジン（ハーネス固有）
├── aidlc/     ← ワークスペース殻（中立・ほぼ共通）
│   ├── active-space          # カーソル（gitignore）
│   └── spaces/
│       └── default/
│           ├── memory/       # Rules（org/team/project/phases）
│           ├── knowledge/    # チーム知識
│           ├── codekb/       # RE 成果のコード知識
│           └── intents/
│               ├── active-intent
│               ├── intents.json
│               └── <YYMMDD>-<label>/
│                   ├── aidlc-state.md
│                   ├── audit/
│                   └── <phase>/<stage>/...
└── （あなたのコードリポジトリ）
```

ポイント:

- **エンジン**と**成果物・方法**を分離
- 複数 Intent を並行保持可能（1.x の単一上書きモデルからの進化）
- multi-repo: ワークスペースの兄弟ディレクトリを Intent 誕生時に記録

---

## 2.7 品質制御ループ

```
  Rules（フィードフォワード）          Sensors（フィードバック）
  エージェントが読む永続指示            書き込み時の決定論チェック
         │                                    │
         └──────── control loop ──────────────┘
```

| 側 | 例 |
|----|-----|
| Rules | org.md / team.md / project.md / phases/*.md |
| Sensors | claim-sources, required-sections, upstream-coverage, linter, type-check |

センサーは現状 **advisory**（監査に記録するがゲートを自動ブロックしない）ものが中心。  
レビュア（Product Lead / Architecture Reviewer）は敵対的レビュー契約で READY/NOT-READY を付与。

---

## 2.8 技術スタック

| 項目 | 内容 |
|------|------|
| ランタイム | **bun**（全ハーネス共通の実行前提） |
| 言語 | TypeScript（core tools / hooks / tests） |
| リント | Biome |
| モデル実行 | **出荷既定は多くのハーネスで AWS Bedrock 寄り**。必須ではない（下表） |
| 推奨モデル | Claude Opus 4.8（公式 README） |
| バージョン定数 | `core/tools/aidlc-version.ts` → `AIDLC_VERSION = "2.5.62"`（本ノート整理時点。2.5.37 → 2.5.62 の差分は [11-release-impact-2562.md](./11-release-impact-2562.md)） |

| ハーネス | モデル／認証の目安 |
|----------|-------------------|
| Claude Code | 出荷 `settings.json` は Bedrock（region・モデル pin）。AWS 資格情報とモデル有効化が実質必要 |
| Codex CLI | 出荷 `config.toml` は Bedrock ブロック。OpenAI 認証等への差し替え余地あり（ガイド参照） |
| Kiro IDE / CLI | **Kiro サインイン + セッションモデル**が中心。2.5.6 以降エージェントはセッションモデル継承 |
| opencode | プロジェクト `opencode.json` はセッションモデルを固定しない。**グローバル opencode 設定のプロバイダ** |
| GitHub Copilot | GitHub Copilot の認証を使用。**folder trust が前提**（`~/.copilot/config.json` の `trustedFolders`）。2.5.60 で追加 |
