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

### レビュアの 60 ターン上限（2.6.8）— ハーネスで強制力がまったく違う

`maxTurns: 60` は**出荷 14 ペルソナのうち 2 つ**（`aidlc-architecture-reviewer-agent` / `aidlc-product-lead-agent`）にだけ付く。正典は `core/agents/aidlc-*-agent.md` の frontmatter 1 箇所で、そこから各ハーネスへ投影される。

**ただし「投影された」ことと「強制される」ことは別**である。上流 CHANGELOG 2.6.8 が自ら列挙している:

| ハーネス | 強制力 | 上限に達したときの挙動 |
|----------|--------|------------------------|
| **Claude Code** | **ネイティブ強制**（`maxTurns` をそのまま読む） | 上限でサブエージェントを停止。**final-message ターンも無い**。呼び出し側は出力を受け取らず、書かれていないレビューは丸ごと失われる |
| **opencode** | **ネイティブ強制**（emit が `steps: 60` に改名） | 最後に**テキストのみ 1 ターン**を許す。要約は返せるが**ツール呼び出しができないのでレビュー本体は書けない** |
| **Codex CLI** | **散文のみ** | TOML ペルソナに frontmatter が無いため、emit がペルソナ本文側の引用文を書き換える形になる |
| **Cursor** | **散文のみ** | per-agent cap キーが無い。寛容な `.md` 面には**無効な `maxTurns: 60` が出荷され続ける** |
| **GitHub Copilot** | **散文のみ** | 同上 |
| **Kiro CLI / Kiro IDE** | **散文のみ** | ネイティブ agent JSON は未知フィールドで fail-closed するためキーは入らず、`.md` 面の inert なキーと `## Turn Budget` 散文だけが届く |

→ **「レビュアは 60 ターンで必ず止まる」と言えるのは Claude Code と opencode だけ。** 残り 5 ハーネスは「エージェントが自分で守る」ことに依存し、決定的な保証は無い。しかも強制される 2 つでも挙動が違う（Claude Code は最終ターン無し / opencode はテキストのみ 1 ターン）。

**incomplete-attempt guard 自体も決定的ではない。** レビュー受領証は「ちょうど 1 つの現行 `## Review` 節 + ちょうど 1 つの正規 verdict」でのみ有効で、欠落・複数・verdict 無しは INCOMPLETE として `--retry-pending` で 1 回だけ無消費リトライし、2 回目の不完全でも `NOT-READY` を確定受領証として記録する（デッドロックはしない）。しかし **verdict のパースは導体（LLM）のプロトコル遵守に依存**しており、ツール側の強制は無い。上流テスト `tests/unit/t279-reviewer-turn-budget.test.ts` の冒頭コメント自身が *"Mechanism: none. Pure content checks over authored + shipped bytes"* と書いている（＝このテストは散文が全ハーネスへ正しく配布されたかしか見ていない）。

そのため**ペルソナ本文の側で最悪ケースを前提にさせる**設計になっている。実際の出荷本文は「上限に達すると、最悪の場合は警告も final-message ターンも無く停止する。書かれなかったレビューは単に失われる。毎回その最悪ケースを想定し、上限より十分手前でレビューを書け（最終ターンに書くな）」と指示している。運用上は、**レビュアが長考して黙って消えることを利用者側も想定しておく**必要がある。

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
| **Kiro CLI** | **2.5.74 で接続可能になった**。出荷 Composer 設定が `includeMcpJson: true` になったため、`.kiro/settings/mcp.json` に CodeKB を `"disabled": true` **なしで**追加し、Composer の `tools` に `@<server>` を足せば使える（**CodeKB 自体は同梱されない**）。**追加しても `allowedTools` には入らないため、呼び出しごとに承認が要る** |
| **Kiro IDE** | **フォールバック専用のまま**。常に **workspace-scan フォールバック** |

CodeKB は AI-DLC 同梱ではない外部 MCP。紛らわしい名前の store が**ほかに 2 つ**あるので、この節末の三者比較表を必ず参照すること。

> **この行は 2.6.2 で変わった。** 上流ガイドの記述自体が書き換わっている（`docs/guide/05-scopes-and-depth.md`）。
> 2.5.62: *"On Kiro CLI **and Kiro IDE** the shipped composer agent config does not grant MCP tools,
> so those harnesses **always** use the workspace-scan fallback."*
> 2.6.2: *"On **Kiro CLI** the shipped composer config sets `includeMcpJson: true`, so connecting CodeKB
> means adding it to `.kiro/settings/mcp.json` without `"disabled": true` and adding its `@<server>` grant
> to the composer agent's `tools`; **Kiro IDE remains fallback-only**."*
> 実ファイルでも Kiro CLI の Composer に `includeMcpJson` が 2.5.62 では無く 2.6.2 で `true` になっている。
> **版の帰属も特定済み**: `git log -S'includeMcpJson' -- dist/kiro/.kiro/agents/aidlc-composer-agent.json`
> が返すのは 1 コミット（`bf1a9c36`）だけで、そのコミットが CHANGELOG に足した見出しは `## [2.5.74]`。
> （このコミットは subject に `(2.5.71)` と書かれているが実体は 2.5.74。→ [12 章 12.10](./12-release-impact-2602.md)）

> **同梱された 5 本と CodeKB を混同しないこと。** 2.5.74 が Kiro CLI に入れたレジストリは
> `@context7` / `@aws-mcp` / `@aws-pricing` / `@aws-iac` / `@aws-serverless` の 5 本で、**CodeKB は含まれない**。
> この 5 本の `disabled` を外しても構造推定は強化されない。CodeKB は自分で追加する必要がある。
> レジストリ自体の仕組み（Kiro CLI のみ・per-agent 付与・既定無効）は 4.5 を参照。

> **接続手順は 4.6 の方針と衝突する。** 上流ガイドが案内する「Composer の `tools` に `@<server>` を足す」は、
> 出荷エージェント定義の直接編集にあたり、4.6 で**非推奨**としている操作（アップグレードで上書きされる）である。
> 上流が他の手段を示していないため手順としてはこれが正だが、**`dist/` を再コピーすると消える**。
> CodeKB を常用するなら、アップグレード手順に再付与を組み込んでおくこと。

実行中の reshape は `recompose`（完了済みステージは凍結）。

### CodeKB MCP / `codekb/` / DocumentKB — 三者の違い

2.6.15 で **DocumentKB**（`aidlc/spaces/<space>/knowledge/documentkb/`）が加わり、「知識の置き場」に見えるものが 3 つになった。**この 3 つはまったく別物**である。

| 問い | **CodeKB MCP**（外部） | **`aidlc/spaces/<space>/codekb/`** | **`.../knowledge/documentkb/`**（2.6.15 新設） |
|------|------------------------|------------------------------------|-----------------------------------------------|
| 実体は何か | AI-DLC 非同梱の外部 MCP サーバ | Reverse Engineering (2.1) の出力先。9 成果物 | ユーザー文書の索引（`index.json` + `<id>/metadata.json` + `<id>/content.md`） |
| 何を対象にするか | 既存コードの構造推定（call graph 等） | 既存コードの解析結果 | PDF / Word / Markdown / プレーンテキスト |
| 誰が作るか | 外部ツール | `aidlc-architect-agent`（ステージ実行） | `tools/aidlc-knowledge.ts`（決定的ツール、**LLM 不介在**） |
| どう操作するか | ハーネス依存の `@<server>` 登録 | ステージを回す | `/aidlc knowledge onboard` / `sync` / `list` / `show` / `associate` / `rebind` |
| **生成・取り込み工程**がネットワークに出るか（※モデルとのやり取りは別） | 出る（サーバ次第） | フレームワーク自身は出ない。ただし**生成するのは `aidlc-architect-agent`（LLM）**なので、リモートホストのモデルを使えばソースはモデル提供者に渡る | フレームワーク自身は出ない（`core/tools/aidlc-knowledge.ts` に `fetch(` / `http(s)://` の出現 0 件。抽出は `spawnSync` のみ） |
| 監査イベント | 対象外 | `PIPELINE_LINK_COMPLETED`（**2.6.39 で新設**。導入コミット `1718b103`、CHANGELOG 見出し `## [2.6.39]`。pipeline ステージの各リンク完了を証跡化する。ステージ自体は従来からある） | `DOCUMENT_INDEXED` / `DOCUMENT_UPDATED` / `DOCUMENT_REMOVED`（**2.6.15 で新設**） |

> **⚠ この行は「フレームワークのコードがネットワーク呼び出しをするか」を問うている。**
> **「情報が外部に出ないか」ではない。** `codekb/` の 9 成果物も DocumentKB の索引も、
> **エージェントが読んで使うために存在する**。リモートでホストされたモデルを使うハーネスでは、
> 読まれた時点でその内容はモデルとのやり取りに乗る。
> 機微なコードや文書を扱う判断は、この行ではなく**利用しているモデルの提供形態**で行うこと。

DocumentKB について、この章の文脈で押さえておく点:

- **索引と抽出の工程はローカル完結**。テキスト抽出は `node:child_process.spawnSync` でローカル実行ファイル（既定は PATH 上の `pdftotext`）を呼ぶだけで、`aidlc-knowledge.ts` にネットワーク呼び出しは無い。
  **ただし「文書が外部に出ない」という意味ではない。** 索引された内容はエージェントが読んで引用するために存在するので、リモートでホストされたモデルを使うハーネスでは、**読まれた時点でモデルとのやり取りに乗る**。機微な文書を投入する判断は、索引工程の実測ではなく**利用しているモデルの提供形態**で行うこと。
- **決定的ツールで、LLM が介在しない**。Composer や 14 ペルソナが構造推定に使う CodeKB とは、そもそも役割が違う。
- **破壊的操作（`remove`）は意図的に未実装**。文書を外したいときは**原本を消してから `sync`** する運用になる。
- 「S1」は**第一スライス**という意味。後続は上流 issue #714 で予告されているが、**「S2」という名称は上流文書のどこにも無い**（本ノートでも使わない）。

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

---

> **上流 2.6.2 → 2.6.49 の差分**: [13-release-impact-2649.md](./13-release-impact-2649.md)
