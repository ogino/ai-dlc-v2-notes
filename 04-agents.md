# 04. エージェント体制（14）

## 4.1 構成

| 区分 | 数 | 役割 |
|------|-----|------|
| ドメイン専門家 | 11 | ステージ実行（lead / support） |
| レビュア | 2 | 成果物への敵対的レビューのみ |
| Composer | 1 | 適応的ステージ計画の提案 |
| **合計** | **14** | |

Conductor（`/aidlc`）はロスタ外の「セッション本体」。

---

## 4.2 11 ドメインエージェント

| Agent | ドメイン | Lead 例 |
|-------|----------|---------|
| **product** | 要件・ストーリー・スコープ・市場 | 1.1, 1.2, 1.4, 2.3, 2.4 |
| **design** | UX/UI・モック | 1.6, 2.5 |
| **delivery** | チーム・納品計画・ハンドオフ | 1.5, 1.7, 2.9 |
| **architect** | 設計・NFR・Unit 分解・契約定義 | 1.3, 2.6–2.8, 3.1–3.3 |
| **aws-platform** | AWS インフラ・CDK | 3.4, 4.2 |
| **compliance** | 規制・データ分類 | サポート専用 |
| **devsecops** | 脅威モデル・セキュリティ | サポート専用 |
| **developer** | 実装・コード分析 | 2.1 scan, 3.5 |
| **quality** | テスト・性能 | 3.6, 4.6 |
| **pipeline-deploy** | CI/CD・デプロイ | 2.2, 3.7, 4.1, 4.3 |
| **operations** | 観測・インシデント・フィードバック | 4.4, 4.5, 4.7 |

### モデル tier（概念）

| Tier | 対象 | 意図 |
|------|------|------|
| **judgment** | architect, product, design, developer, quality, devsecops, compliance, aws-platform 等 | セッションのモデル／effort を継承（格下げしない） |
| **balanced** | レビュア等 | 中間 |
| **templated** | delivery, pipeline-deploy, operations | 計画書・YAML・ランブック寄り |

**Kiro 注意**: 2.5.6 以降、Kiro ではモデル pin を外し、**セッションモデル継承**（利用可能モデルの不一致で spawn 失敗する問題の修正）。

---

## 4.3 レビュア（2）

| Reviewer | 対象 | 契約 |
|----------|------|------|
| **product-lead** | 要件・ストーリー・モック | ビジネス整合・検証可能性 |
| **architecture-reviewer** | 技術設計・NFR・コード生成 | 健全性・実装可能性・参照整合 |

動作:

1. ステージ本体が成果物を生成
2. 別 subagent としてレビュア起動（builder の memory は見ない）
3. `## Review` に **READY / NOT-READY**
4. NOT-READY なら builder 再実行（最大 `reviewer_max_iterations` 既定 2）
5. 上限後も人間ゲートへ（**レビュアは最終拒否権を持たない**）

2.4.0 以降: **adversarial review contract**（欠陥がある前提で反証する。意見だけの NOT-READY は不可）。

**エンジンによる強制（2.5.5 以降）**: `reviewer` を宣言したステージは、当該レビュアの新鮮な `REVIEW_COMPLETED` 監査行が無いと `approve` / `advance` / `finalize` / `complete-workflow` のいずれの完了経路でも**拒否される**。導体の判断でレビュアを省略することはできない。

> ただしカバレッジは全ステージではない。`reviewer:` を宣言するのは **33 中 13 ステージ**で、そのうち 9 は `execution: CONDITIONAL`。CONDITIONAL ステージは条件不成立としてスキップ報告できるため、**強制が回避不能なのは ALWAYS の 4 ステージ**（intent-capture / requirements-analysis / units-generation / code-generation）である。

**チェックリストの所在（2.5.33 以降）**: レビュアの `knowledge/<agent>/reviewing.md` は、パッケージング時に**レビュア agent 本体へ埋め込まれる**。実行時に知識 glob を読みに行く設計から、ビルド時埋め込みへ変わった。Kiro のレビュア JSON からは冗長な per-agent 知識 glob が削除されている。

---

## 4.4 Composer（1）

`aidlc-composer-agent` — 適応的ワークフロー設計。

起動例:

```
/aidlc compose "harden the deployment pipeline and add observability"
/aidlc compose --report sonar.json
/aidlc --new-scope "..."
```

処理:

1. 実装エントロピー 5 成分を推定  
   （意図曖昧さ / 構造不確実性 / 検証エントロピー / リスク / 未解決仮定）
2. 最小十分の EXECUTE/SKIP グリッドを提案
3. 人間が承認するまで何も書かない
4. 承認後: 既存スコープ一致ならそのまま birth / カスタムなら scope ファイル生成

オプションで **CodeKB MCP**（外部）があれば call-graph 等で構造推定を強化。未設定時は workspace scan。

**ハーネス差（重要）**:

| ハーネス | CodeKB MCP |
|----------|------------|
| Claude Code / Codex / opencode | 設定すれば Composer が構造推定に利用可能（接続方法は各 harness ガイド） |
| **Kiro CLI / Kiro IDE** | 出荷の Composer 設定に **CodeKB は付与されない**。常に **workspace-scan フォールバック** |

CodeKB は AI-DLC 同梱ではない外部 MCP。フレームワーク内の `aidlc/spaces/<space>/codekb/`（Reverse Engineering 成果のローカル store）とは別物。

> **2.5.74 の MCP レジストリはこの表を変えない。** Kiro CLI に同梱されたのは
> `@context7` / `@aws-mcp` / `@aws-pricing` / `@aws-iac` / `@aws-serverless` の 5 本で、
> **CodeKB はこの中に無い**。したがって 5 本の `disabled` を外しても、
> Composer の構造推定に CodeKB が使えるようにはならない。
> レジストリ自体の仕組み（Kiro CLI のみ・per-agent 付与・既定無効）は 4.5 を参照。

実行中の reshape は `recompose`（完了済みステージは凍結）。

---

## 4.5 ツールアクセス方針

- 既定: セッションの全ツール + MCP を継承
- 唯一の出荷制限: **`Task` はエージェント禁止**（サブエージェント spawn は Conductor のみ）
- MCP はプロジェクト `.mcp.json` 等で共有（per-agent 付与なし）——
  **ただし Kiro CLI だけは 2.5.74 で例外になった**。`.kiro/settings/mcp.json` のレジストリを持ち、
  14 ペルソナの `tools` に `@server` が個別付与される（既定は全サーバ `disabled: true`）。
  `@server` はどのペルソナの `allowedTools` にも入っていないため、有効化しても呼び出しごとに承認が要る。
  他 6 ハーネス（Kiro IDE を含む）のエージェント定義に `@server` は 1 件も無い

---

## 4.6 カスタマイズの正しい場所

| やりたいこと | 場所 |
|--------------|------|
| 会社標準・規約をエージェントに読ませる | `aidlc/spaces/<space>/knowledge/<agent>/` ※1 |
| 行動ルールを永続化 | `aidlc/spaces/<space>/memory/` ※2 |
| 新エージェントを追加 | ユーザー所有の agents ファイル（アップグレードで上書きされない場所） |
| 出荷 14 体の本文を直接編集 | **非推奨**（アップグレードで上書き） |

※1 レビュア 2 体はチェックリストがビルド時に本体へ埋め込まれるため（4.3 参照）、`knowledge/<reviewer-agent>/` への配置がドメインエージェントと同じ経路で効くかは要確認。

※2 ルールは 2.5.33 以降、導体がパスを読むかどうかに依存せず、エンジンが `run-stage` 前に `load-steering` ディレクティブとして**内容を確定配信**する（7.2 参照）。知識ファイルは引き続きパス参照（path-loaded）。
