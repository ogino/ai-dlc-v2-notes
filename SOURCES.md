# 調査ソース一覧

調査日: 2026-07-28  
整理: コーディングエージェントによる二次整理（モデル帰属は会話ログを参照）

## 一次情報（優先）

1. GitHub `awslabs/aidlc-workflows` tree/main README  
   https://github.com/awslabs/aidlc-workflows/tree/main
2. 公式 `main` ブランチのローカル clone  
   `git clone --depth 1 --branch main https://github.com/awslabs/aidlc-workflows.git`  
   **2026-09-01 の再編以前は `v2` ブランチだった。`v2` は削除済みで、`main` がその線形な継続である**
   （11〜15 章の `基準:` 行が記録している HEAD SHA は、いずれも `main` から到達できる。
   10 章は `対象:` 行にブランチ名のみで SHA を持たない）
3. 同 clone 内（精読・突合済み）:
   - `README.md`（GA、ハーネス、バージョン要件）
   - `CHANGELOG.md`（**2.5.x を中心**に参照。2.4 / 2.3 も一部）
   - `core/tools/aidlc-version.ts` → `2.7.0`（2026-09-01 再取得。branch `main` HEAD `96b11d39`。初回調査時 `2.5.11` → 2026-08-05 時点 `2.5.37` → 2026-08-10 時点 `2.5.62` → 2026-08-14 時点 `2.6.2` → 2026-08-22 時点 `2.6.49`（HEAD `71d9a9e0`）→ 2026-08-22 時点 `2.6.55`（HEAD `840ba653`）→ 2026-08-29 時点 `2.6.123`（branch `v2` HEAD `2fbee12f`））
   - `docs/guide/00-introduction.md`
   - `docs/guide/03-spaces-and-intents.md`
   - `docs/guide/04-phases-and-stages.md`
   - `docs/guide/05-scopes-and-depth.md`
   - `docs/guide/06-agents.md`
   - `docs/guide/09-rules-and-the-learning-loop.md`
   - `core/knowledge/aidlc-shared/audit-format.md`（監査イベントの実レジストリ。2026-08-11 に独立集計）
   - `core/hooks/aidlc-fold-usage.ts` / `core/tools/aidlc-metrics.ts`（2026-08-11）
   - `docs/guide/harnesses/copilot.md` / `dist/copilot/AGENTS.md`（2026-08-11）
   - `docs/guide/harnesses/cursor.md`（2026-08-14。2.5.63 で追加された Cursor ハーネス）
   - `core/sensors/aidlc-traceability.md`（2026-08-14。2.5.71 で追加された 6 本目のセンサー）
   - `core/aidlc-common/stages/inception/domain-design.md` / `contract-design.md`（2026-08-14。2.6.1 で `application-design` を置き換え）
   - `dist/kiro/.kiro/settings/mcp.json` と `dist/kiro/.kiro/agents/*.json`（14 ペルソナ＋指揮役 `aidlc.json`）（2026-08-14。2.5.74 の MCP レジストリ）
   - `dist/kiro-ide/.kiro/agents/aidlc-composer-agent.json` と `dist/claude/.mcp.json`（2026-08-14。MCP 同梱の有無をハーネス間で突き合わせるため）
   - 全 7 ハーネスの `dist/*/…/agents/` 配下（2026-08-14。`@server` 付与の有無を機械的に集計）
   - `docs/guide/glossary.md`（2026-08-29 に再読。**2.6.86 で Bolt の定義が書き換わっている**）
   - `core/` ディレクトリ一覧（stages/tools/hooks/scopes/sensors）
   - 2.6.55 → 2.6.123 区間（2026-08-29 実測。HEAD `2fbee12f`）:
     `core/tools/*.ts`（`import.meta.main` の有無で CLI とライブラリを判別）/
     `core/hooks/*.ts` / `core/scopes/*.md` と `scope-grid.json` /
     `core/sensors/*.md`（`fire_on` と `default_severity`）/
     `core/tools/aidlc-audit.ts` の `VALID_EVENT_TYPES` — 詳細は
     [15-release-impact-26123.md](./15-release-impact-26123.md)
   - 2.6.123 → 2.7.0 区間（2026-09-01 実測。branch `main` HEAD `96b11d39`）:
     上記と同じ測定を再実行し**全項目不変**を確認 / `git diff 2fbee12f..origin/main`（`core/` は 4 ファイル 4 行）/
     `docs/roadmap.md` / `.github/workflows/` — 詳細は
     [16-release-impact-2700.md](./16-release-impact-2700.md)

## 二次情報（裏取りに使用）

**本リポジトリの記述は原則として上流クローンの実ファイル計測に基づく。**
以下は例外的に外部ページを根拠にした箇所であり、区別できるよう分けて記載する。

- VS Code 更新履歴（いずれも 2026-08-11 参照）
  - https://code.visualstudio.com/updates/v1_130 — 見出し「Visual Studio Code 1.130」、`Release date: July 22, 2026`
  - https://code.visualstudio.com/updates/v1_125 — `Release date: June 17, 2026`
  - https://code.visualstudio.com/updates/v1_115 — `Release date: April 8, 2026`
  - https://code.visualstudio.com/updates/v1_111 — **`the first of our weekly Stable releases`**、`Release date: March 9, 2026`
  - https://code.visualstudio.com/updates/v1_110 — 見出し「February 2026 (version 1.110)」、`Release date: March 4, 2026`
  - https://code.visualstudio.com/updates/v1_103 — 見出し「July 2025 (version 1.103)」、`Release date: August 7, 2025`
  - 用途: 上流が要求する `VS Code >= 1.130` が実在することの確認。
    **リリース運用は 1.111 で月刊から週次へ切り替わっている**（上流が `v1_111` で明記）ため、
    その境をまたぐ区間からの外挿は成り立たない。境の前後を押さえるために複数点を取った。
    表に出している値は**平均間隔**であり、各版の実間隔ではない（中間版は個別に確認していない）

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
- `docs/rfcs/` のうち **HTML 版 2 本**（`IMPLEMENTATION-PLAN.html` / `kiro-ide-hooks-fix-plan.html`）。
  Markdown 版 2 本（`IMPLEMENTATION-PLAN.md` / `reviewer-reliability-and-stage-decomposition.md`）は
  **精読済み**で、第12.9節で逐語引用している（ただし仕様ではなく上流の作業用メモとして扱う）
- 全 stage ファイルの frontmatter マトリクス

## 正本の優先順位（本ノート内）

1. **精読済み**: `main` の `docs/guide/*`・`README`・`CHANGELOG`・`core/` の実体  
2. **未精読だが公式**: Spec PDF 全文、Method Paper 全文 → 細部は必ず原典を確認  
3. **二次解説**: 対比・理解の補助のみ。1.x 一般化などは [08](./08-v1-vs-v2.md) の留保に従う  

## 免責

本ノートは上記の要約・再構成であり、AWS または awslabs の公式文書ではない。  
**実装・運用の正は常に `main` ブランチのソースと `docs/`（および必要なら Spec PDF）である。**  
本ノートが「正とする」と書く箇所も、精読範囲外の細部については原典が優先する。
