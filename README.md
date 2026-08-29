# AWS AI-DLC Workflows 2.0 整理ノート（非公式）

> **免責（必読）**  
> - 本リポジトリは **非公式の二次整理** であり、AWS / awslabs / Amazon の公式文書・公式サポートではない  
> - 生成 AI による要約・再構成を含み、**誤りがあり得る**  
> - 実装・仕様の正は常に上流 [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows)（v2）のソースと `docs/` を参照すること  
> - 本リポジトリの文章のライセンスは **MIT**（`LICENSE`）。上流実装のライセンスは **MIT-0**（別物）

初回調査日: 2026-07-28（実装バージョン 2.5.11）  
最終同期日: 2026-08-23  
対象実装: [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows) **v2 ブランチ**（実装バージョン **2.6.123**。上流 HEAD `2fbee12f` / 取得日 2026-08-23）

> 上流はマイナーリリースが頻繁である。本ノートは特定時点のスナップショットであり、
> 数値・仕様は参照時に、**上流リポジトリ**（`awslabs/aidlc-workflows` v2 のローカル clone）の `core/tools/aidlc-version.ts` と CHANGELOG で必ず照合すること。本ノートのリポジトリには `core/` は存在しない。

---

## このノートの目的

インターネット上の公式・解説情報と、`v2` ブランチのソース／ドキュメントを突き合わせ、**AI-DLC（AI-Driven Development Life Cycle）Workflows 2.0** を日本語で俯瞰できるようにしたものです。

- **方法論（methodology）**: AWS が定義した AI 中心の開発ライフサイクル
- **実装（implementation）**: `aidlc-workflows` の v2 ブランチが提供する、複数 CLI ハーネス向けのネイティブ実行エンジン

---

## 一言で言うと

AI-DLC 2.0 は、**「プロンプトを投げて祈る」アドホックな AI コーディングを、検証可能で自己修正するエンジニアリング・ワークフローに変える**枠組みです。

- **承認ゲートでは人が最終判断**する（レビュアは最終拒否権を持たない）
  - 例外: Initialization はゲートなし／フェーズ境界の Verification Gate は自動／Construction で ladder 後に **autonomous** を選ぶと以降の Construction 反復のゲートは省略可（失敗時は halt-and-ask）。**「Bolt」は 2.6.86 で上流が「計画上のスプリント様スライス」へ再定義した** → [01.8.1](./01-overview.md)
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
| 監査イベント種別 | **91**（22 分類）※ |
| 対応ハーネス | Claude Code, Kiro IDE, Kiro CLI, Codex CLI, **Cursor**, opencode, GitHub Copilot（計 7 種） |
| 実装バージョン | **2.6.123**（上流 HEAD `2fbee12f`。取得日 2026-08-23） |
| 上流実装のライセンス | MIT-0（`aidlc-workflows`） |
| 本ノートのライセンス | MIT（本リポジトリ `LICENSE`） |

※ 監査カテゴリ数は正典レジストリ `core/knowledge/aidlc-shared/audit-format.md` の Event Registry 見出し基準（22）。`docs/reference/12-state-machine.md` 基準では 19 分類（イベント種別の集合自体は両出典で同一）。

---

## クイック開始（概念）

```bash
# 1. 全ハーネス共通: bun を PATH に（非対話シェルからも見えること）
curl -fsSL https://bun.sh/install | bash

# 2. ソース取得
git clone https://github.com/awslabs/aidlc-workflows.git
cd aidlc-workflows && git checkout v2

# 3. 使うハーネスの dist をプロジェクトへコピー
# 例: Claude Code
cp -R dist/claude/.claude/. your-project/.claude/
cp -R dist/claude/aidlc/.   your-project/aidlc/

# 4. セッション内 — まずヘルスチェック、問題なければワークフロー開始
/aidlc --doctor
/aidlc Build a task management API with user authentication
```

**モデル／認証の前提（ハーネス別）**: 出荷設定は多くの場合 **AWS Bedrock** を想定するが、**全ハーネス共通の必須ではない**。Claude Code / Codex の出荷設定は Bedrock 寄り、Kiro はサインインとセッションモデル、opencode はグローバル設定のプロバイダ、に依存する。詳細は [06-harnesses-install.md](./06-harnesses-install.md)。

---

## 注意

- 生成 AI の出力は誤りを含み得る。公式も **生成物とコストのレビュー**を求めている
- 推奨モデルは公式 README 時点で **Claude Opus 4.8**（特に Kiro では有料プランが必要な場合あり）
- 本ノートは **二次整理**である。数値・手順の正本は、精読した **v2 ブランチの `docs/`・ソース・CHANGELOG** を優先する
- [2.0 Specification PDF](https://github.com/awslabs/aidlc-workflows/blob/v2/assets/AI-DLC-Workflows-2.0-Specification.pdf)（リポジトリ上は主に `assets/`）は公式白書だが、**本ノート作成時は全文未精読**。PDF 固有の細部は PDF 本体を確認すること
- 公式 README の NOTE どおり、インタフェースは安定しつつ継続改善されるため、依存する場合は **既知良版の pin** を推奨
