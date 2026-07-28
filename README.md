# AWS AI-DLC Workflows 2.0 整理ノート（非公式）

> **免責（必読）**  
> - 本リポジトリは **非公式の二次整理** であり、AWS / awslabs / Amazon の公式文書・公式サポートではない  
> - 生成 AI による要約・再構成を含み、**誤りがあり得る**  
> - 実装・仕様の正は常に上流 [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows)（v2）のソースと `docs/` を参照すること  
> - 本リポジトリの文章のライセンスは **MIT**（`LICENSE`）。上流実装のライセンスは **MIT-0**（別物）

調査日: 2026-07-28  
対象実装: [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows) **v2 ブランチ**（実装バージョン **2.5.11**）

---

## このノートの目的

インターネット上の公式・解説情報と、`v2` ブランチのソース／ドキュメントを突き合わせ、**AI-DLC（AI-Driven Development Life Cycle）Workflows 2.0** を日本語で俯瞰できるようにしたものです。

- **方法論（methodology）**: AWS が定義した AI 中心の開発ライフサイクル
- **実装（implementation）**: `aidlc-workflows` の v2 ブランチが提供する、複数 CLI ハーネス向けのネイティブ実行エンジン

---

## 一言で言うと

AI-DLC 2.0 は、**「プロンプトを投げて祈る」アドホックな AI コーディングを、検証可能で自己修正するエンジニアリング・ワークフローに変える**枠組みです。

- **承認ゲートでは人が最終判断**する（レビュアは最終拒否権を持たない）
  - 例外: Initialization はゲートなし／フェーズ境界の Verification Gate は自動／Construction で ladder 後に **autonomous** を選ぶと残 Bolt のゲートは省略可（失敗時は halt-and-ask）
- 決定論的エンジンがルーティングし、LLM 導体（conductor）が実行品質を担う
- 1 つの `core/` から Claude Code / Kiro IDE / Kiro CLI / Codex / opencode 向け配布物を生成する

---

## ドキュメント一覧

### 公開向け（読む順番の目安）

| ファイル | 内容 |
|----------|------|
| [01-overview.md](./01-overview.md) | 背景・方法論・2.0 GA の位置づけ |
| [02-architecture.md](./02-architecture.md) | リポジトリ構成・Engine/Conductor・平面モデル |
| [03-phases-and-stages.md](./03-phases-and-stages.md) | 5 フェーズ / 32 ステージの全体像 |
| [04-agents.md](./04-agents.md) | 14 エージェント体制 |
| [05-scopes-depth-test.md](./05-scopes-depth-test.md) | 9 スコープ・深度・テスト戦略・Composer |
| [06-harnesses-install.md](./06-harnesses-install.md) | 対応ハーネスと導入手順の要点 |
| [07-learning-loop-state.md](./07-learning-loop-state.md) | Space/Intent・Rules・Sensors・監査 |
| [08-v1-vs-v2.md](./08-v1-vs-v2.md) | 1.x 系と 2.0 の差分 |
| [09-references.md](./09-references.md) | 参照リンク・上流リポジトリ内パス |
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
| ステージ | 32 |
| エージェント | 14（ドメイン 11 + レビュア 2 + Composer 1） |
| スコープ | 9 + 自動検出 + カスタム compose |
| 深度 / テスト戦略 | 各 3 段階（独立） |
| 監査イベント種別 | 74 |
| 対応ハーネス | Claude Code, Kiro IDE, Kiro CLI, Codex CLI, opencode |
| 実装バージョン | 2.5.11（2026-07-24 時点 CHANGELOG） |
| 上流実装のライセンス | MIT-0（`aidlc-workflows`） |
| 本ノートのライセンス | MIT（本リポジトリ `LICENSE`） |

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
cp -r dist/claude/.claude/ your-project/.claude/
cp -r dist/claude/aidlc/   your-project/aidlc/

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
