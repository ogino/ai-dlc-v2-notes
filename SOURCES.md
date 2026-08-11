# 調査ソース一覧

調査日: 2026-07-28  
整理: コーディングエージェントによる二次整理（モデル帰属は会話ログを参照）

## 一次情報（優先）

1. GitHub `awslabs/aidlc-workflows` tree/v2 README  
   https://github.com/awslabs/aidlc-workflows/tree/v2
2. 公式 v2 ブランチのローカル clone（`git clone --depth 1 --branch v2`）
3. 同 clone 内（精読・突合済み）:
   - `README.md`（GA、ハーネス、バージョン要件）
   - `CHANGELOG.md`（**2.5.x を中心**に参照。2.4 / 2.3 も一部）
   - `core/tools/aidlc-version.ts` → `2.5.62`（2026-08-10 再取得。初回調査時 `2.5.11` → 2026-08-05 時点 `2.5.37`）
   - `docs/guide/00-introduction.md`
   - `docs/guide/03-spaces-and-intents.md`
   - `docs/guide/04-phases-and-stages.md`
   - `docs/guide/05-scopes-and-depth.md`
   - `docs/guide/06-agents.md`
   - `docs/guide/09-rules-and-the-learning-loop.md`
   - `core/knowledge/aidlc-shared/audit-format.md`（監査イベントの実レジストリ。2026-08-11 に独立集計）
   - `core/hooks/aidlc-fold-usage.ts` / `core/tools/aidlc-metrics.ts`（2026-08-11）
   - `docs/guide/harnesses/copilot.md` / `dist/copilot/AGENTS.md`（2026-08-11）
   - `docs/guide/glossary.md`
   - `core/` ディレクトリ一覧（stages/tools/hooks/scopes/sensors）

## 二次情報（裏取りに使用）

**本リポジトリの記述は原則として上流クローンの実ファイル計測に基づく。**
以下は例外的に外部ページを根拠にした箇所であり、区別できるよう分けて記載する。

- VS Code 更新履歴
  - https://code.visualstudio.com/updates/v1_130 — 見出し「Visual Studio Code 1.130」、
    本文に `Release date: July 22, 2026`
  - https://code.visualstudio.com/updates/v1_125 — 同 `Release date: June 17, 2026`
  - いずれも 2026-08-11 参照。上流が要求する `VS Code >= 1.130` が実在する版であることの確認と、
    リリース間隔の裏取りに使用（**VS Code は月刊ではなく、1.125 → 1.130 が約 5 週間**）

---

## Web / AWS

4. AI-Driven Development Life Cycle ブログ  
   https://aws.amazon.com/blogs/devops/ai-driven-development-life-cycle/
5. Open-Sourcing Adaptive Workflows for AI-DLC  
   https://aws.amazon.com/blogs/devops/open-sourcing-adaptive-workflows-for-ai-driven-development-life-cycle-ai-dlc/
6. Building with AI-DLC using Amazon Q Developer  
   https://aws.amazon.com/blogs/devops/building-with-ai-dlc-using-amazon-q-developer/

## 二次解説（補足・対比）

7. ELEKS: AI-DLC Explained  
   https://eleks.com/blog/aws-ai-dlc-explained/
8. specs.md AI-DLC Flow Overview  
   https://specs.md/aidlc/overview
9. Zenn: A Deep Dive into the Rules of AI-DLC  
   https://zenn.dev/aecomet/articles/ai-dlc-workflows-deep-dive
10. Reddit: Workflows 2.0 preview 言及（**非公式・参考程度**）  
    https://www.reddit.com/r/AIDeveloperNews/comments/1v2zlz0/aws_previews_aidlc_workflows_20_a_fully/
11. Builder.aws: AI-DLC v2 Core Concepts（公開コミュニティ記事）  
    https://builder.aws.com/content/3GnQs6zaYp4mAx9FWBulDkhWnBo/ai-dlc-v2-core-concepts

## 未精読（必要なら次）

- `assets/AI-DLC-Workflows-2.0-Specification.pdf` **全文**（パス上の正は `assets/`。数値の多くは `docs/` と README で突合済み）
- Method Definition Paper 全文（Amplify ホスト）
- `docs/reference/*` 各章（エンジン内部）
- `docs/harness-engineering/*`
- 全 stage ファイルの frontmatter マトリクス

## 正本の優先順位（本ノート内）

1. **精読済み**: v2 の `docs/guide/*`・`README`・`CHANGELOG`・`core/` の実体  
2. **未精読だが公式**: Spec PDF 全文、Method Paper 全文 → 細部は必ず原典を確認  
3. **二次解説**: 対比・理解の補助のみ。1.x 一般化などは [08](./08-v1-vs-v2.md) の留保に従う  

## 免責

本ノートは上記の要約・再構成であり、AWS または awslabs の公式文書ではない。  
**実装・運用の正は常に v2 ブランチのソースと `docs/`（および必要なら Spec PDF）である。**  
本ノートが「正とする」と書く箇所も、精読範囲外の細部については原典が優先する。
