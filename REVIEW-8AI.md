# 8-AI Review Results — AI-DLC Workflows 2.0 整理ノート

実施日: 2026-07-28  
対象: 本リポジトリのドキュメント一式（ドキュメント精度レビュー）  
種別: feature ブランチ向けコードレビュー手順を、**二次整理ドキュメントの事実性レビュー**に転用

---

## サマリー表

| AI | Status | HIGH | MEDIUM | LOW | 判定 |
|----|--------|------|--------|-----|------|
| Session self | ✅ Complete | 0 | 3 | 4 | APPROVE WITH NITS |
| Codex CLI | ✅ Complete（再実行後） | 0 | 3 | 1 | NEEDS FIXES |
| Copilot CLI | ✅ Complete | 0 | 1 | 3 | APPROVE WITH NITS |
| Antigravity (agy) | ✅ Complete | 3* | 1 | 1 | NEEDS FIXES* |
| Grok CLI | ✅ Complete | 0 | 6 | 8 | APPROVE WITH NITS |
| Cursor Agent | ✅ Complete（復旧後） | 0 | 7 | 6 | APPROVE WITH NITS |
| Devin CLI | ✅ Complete | 0 | 5 | 1 | NEEDS FIXES |
| Kiro CLI | ✅ Complete（復旧後） | 1* | 8 | 4 | APPROVE WITH NITS |

\* 下記「HIGH の裁定」参照。一次ソース照合の結果、AI-DLC 事実誤認としての **ブロッキング HIGH は 0**。

---

## レビュー対象

- ドキュメントセット: README + 01〜09 + SOURCES（レビュー時点）
- 対照一次情報: `awslabs/aidlc-workflows` v2（`AIDLC_VERSION=2.5.11`）の README / docs / CHANGELOG / core
- 送信前: クレデンシャル類のスキャン済み

---

## HIGH の裁定（オーケストレータ）

| 指摘元 | 主張 | 裁定 |
|--------|------|------|
| Antigravity | GA が invented（Reddit preview のみ） | **却下**。v2 README に *Announcing 2.0 (GA)* / *generally available* |
| Antigravity | Opus 4.8 / Codex≥0.145 / opencode≥1.17 が invented | **却下**。公式 README 記載 |
| Antigravity | builder.aws.com が internal で privacy 違反 | **却下（過剰）**。公開の AWS Builder コミュニティ URL |
| Kiro | 整理者モデル記録の厳密性 | **運用注意**（AI-DLC 事実誤りではない） |

**結論: 公開を阻む HIGH（AI-DLC の誤誘導）は 0 件。**

---

## 合意の高い MEDIUM と対応状況

| ID | 内容 | 状態 |
|----|------|------|
| M1 | ゲート例外（autonomous / Init / Verification） | ✅ 反映済 |
| M2 | Bedrock 出荷既定 vs ハーネス別必須 | ✅ 反映済 |
| M3 | Construction 3.1–3.4 CONDITIONAL | ✅ 反映済 |
| M4 | hub-and-spoke ≠ mode 名（`mode: subagent`） | ✅ 反映済 |
| M5 | tools 本数の出典差（25 vs 約30） | ✅ 反映済 |
| M6 | mvp スキップ詳細（Ideation + Operation） | ✅ 反映済 |
| M7 | CodeKB は Kiro で workspace-scan 固定 | ✅ 反映済（**2.5.74 で状況が変わった**。Kiro CLI は接続可能になり、固定なのは Kiro IDE のみ → [04-agents.md](./04-agents.md) 4.4） |
| M8 | Codex trust 手順の補足 | ✅ 反映済 |
| M9 | 1.x 対比の留保強化 | ✅ 反映済 |
| M10 | Spec PDF は `assets/`、正本優先順位 | ✅ 反映済 |

LOW は対応任意（未対応）。

---

## 正確と確認された中核

> **本節は 2026-07-28・実装 2.5.11 時点の確認結果**であり、現在値ではない。
> **2.6.2（CHANGELOG 日付 2026-08-13 / 取得日 2026-08-14）時点では 5 フェーズ / 33 ステージ / 14 agents / 9 scopes / 7 harnesses、
> センサー 6 種、監査 82 イベント種別（21 分類）、mode 集計 29 inline / 2 subagent / 1 pipeline / 1 mob。**
> 変化の経緯は [11-release-impact-2562.md](./11-release-impact-2562.md) と
> [12-release-impact-2602.md](./12-release-impact-2602.md) を参照。
>
> **時点限定（追記 2026-08-22）**: 上記も 2.6.2 時点の記録。**上流実測（HEAD `71d9a9e0` / 実装 2.6.49）**
> ではスコープが **11**、監査イベントが **86 種（22 分類）**に変わっている。5 フェーズ / 33 ステージ /
> 14 agents / センサー 6 / TypeScript フック 17 / 7 harnesses は不変。→ [13-release-impact-2649.md](./13-release-impact-2649.md)
>
> **さらに追記（2026-08-22・同日の再同期）**: 上流は **2.6.55**（HEAD `840ba653`）まで進んだが、
> **中核メトリクスは全項目不変**で、`stage-graph.json` / `scope-grid.json` は 2.6.49 と
> **バイト単位で一致**する。変わったのは実行時のガードと継続トークンと監査の発火条件である。
> → [14-release-impact-2655.md](./14-release-impact-2655.md)
>
> **さらに追記（2026-08-29）**: 上流は **2.6.123**（HEAD `2fbee12f`）まで進み、今度は中核メトリクスが動いた。
> **TypeScript フック 17 → 18 / `core/tools/*.ts` 41 → 51 / 監査 86 → 91 イベント種別**、
> スコープのステージ数が **`bugfix` 7/33 → 9/33・`refactor` 8/33 → 10/33**（2.6.70）。
> ステージ 33 / スコープ数 11 / 14 agents / センサー数 6 / 監査カテゴリ 22 / 7 harnesses は不変。
> **51 は CLI の本数ではない**（起動可能な CLI は 32、残り 19 はライブラリモジュール）。
> → [15-release-impact-26123.md](./15-release-impact-26123.md)
>
> **さらに追記（2026-09-01）**: 上流は **2.7.0**（`main` HEAD `96b11d39`）まで進んだが、
> **中核メトリクスは全項目不変**である。今回変わったのはブランチ構成で、
> **`v2` ブランチが削除され `main` が 2.x の正本になった**（旧 1.x は新設 `v1` ブランチへ）。
> 実装が動いたのは 2.6.124 の 1 版のみ、**2.7.0 のコード変更はゼロ**である。
> → [16-release-impact-2700.md](./16-release-impact-2700.md)

**⚠ 以下の箇条書きは 2.5.11 時点の確認結果であり、現行値ではない。**
現行値は上の追記ブロック（2026-09-01 / 2.7.0）を参照すること。
**ステージ 32 → 33、scopes 9 → 11、harnesses 5 → 7、センサー 5 → 6、監査 74 → 91 と、
すべて動いている。**

- 5 フェーズ / 32 ステージ / 14 agents / 9 scopes / 5 harnesses
- バージョン 2.5.11、GA 表記、MIT-0（上流実装）
- Engine / Conductor、Bolt / walking skeleton / ladder
- 学習ループは「次回ワークフローから」
- センサー 5 種、監査 74 イベント種別
- mode 集計 28 inline / 2 subagent / 1 pipeline / 1 mob

---

## 総合判定（ドキュメント精度レビュー時点）

| 観点 | 判定 |
|------|------|
| 事実誤認（HIGH, 裁定後） | **0** |
| MEDIUM | **対応済み** |
| ドキュメント精度 | **APPROVE WITH NITS**（LOW のみ残） |

---

## 公開可否レビュー（2026-07-28）

観点: privacy / 二次整理の明示 / ライセンス / 公開体裁

| AI | Status | privacy | 判定 |
|----|--------|---------|------|
| Session self | ✅ | clean | PUBLISH WITH NITS |
| Codex CLI | ✅ | clean | PUBLISH WITH NITS |
| Copilot CLI | ✅ | clean | PUBLISH WITH NITS |
| Antigravity (agy) | ✅ | clean | PUBLISH |
| Grok CLI | ✅ | clean | PUBLISH WITH NITS |
| Cursor Agent | ✅ | clean | PUBLISH WITH NITS |
| Devin CLI | ✅ 単独リトライ後 | clean | PUBLISH WITH NITS |
| Kiro CLI | ✅ | clean | MEDIUM は免責強化等 |

### 公開レビューで合意した MEDIUM と対応

| 指摘 | 対応 |
|------|------|
| 冒頭の非公式・二次整理バナー強化 | ✅ README 冒頭に免責ブロック |
| 上流 MIT-0 と本リポ MIT の混同 | ✅ メトリクス表を分離 |
| 作業ログを公開資料と並べすぎ | ✅ README で「公開向け / メンテナ向け」分割 |
| 09 見出し「ローカルパス」 | ✅ 「上流リポジトリ内パス」へ変更 |

### Devin CLI — 単独リトライ結果（2026-07-28）

**経緯**: 公開可否レビューの **7 並列実行時**は stdout 空で helper が `Devin CLI returned no output.` を返した。CLI 未導入ではなく、並列時の空レスポンスと判断。

**再診断**:
- `devin` 3000.2.17、認証 OK
- 極小 smoke 成功
- 同一公開プロンプトを **単独実行** → 成功（下記）

**Devin の指摘一覧（単独リトライ）**:

| # | Severity | File | Issue | 本リポジトリでの扱い |
|---|----------|------|-------|----------------------|
| 1 | MEDIUM | `09-references.md`, `SOURCES.md` | Method Definition Paper の URL が Amplify 既定ドメイン（`prod.d13rzhkk8cj2z0.amplifyapp.com`）で、公式ホストか読者に分かりにくい | **既知**。上流 README が同 URL を参照している。二次整理として踏襲。公式ドメインが確認でき次第差し替え（任意） |
| 2 | LOW | `README.md` | メトリクス表のライセンスが MIT-0 で本リポ MIT と混同しうる | **対応済み**（上流 MIT-0 / 本ノート MIT を分離） |
| 3 | LOW | 複数 | 「端到端」→「エンドツーエンド」 | 任意（`docs/REMAINING_TASKS.md` に記載） |
| 4 | LOW | `06-harnesses-install.md` | Kiro「≥2.6」が CLI 版か曖昧 | 任意（Kiro **CLI** バージョンの意味で記載） |

Devin 判定: **PUBLISH WITH NITS** / **privacy: clean** / HIGH なし

### 公開可否（オーケストレータ）

- **HIGH ブロッカー: なし**（privacy: clean が **8 系統すべて**で一致）
- **Devin: 単独リトライで完了**（並列時のみ失敗）
- **判定: PUBLISH WITH NITS**（公開向け nit は主要対応済み。Devin #1 の Amplify URL は上流踏襲の任意改善）
- **リモート push はユーザー最終確認後のみ**

---

## 8AI レビュー（2026-09-01 / 2.6.123 → 2.7.0・ブランチ再編）

対象: 新章 [16-release-impact-2700.md](./16-release-impact-2700.md) と、既存 15 ファイルの改訂差分。
非公開リポジトリ側の同区間レポートも同一プロンプトで併せてレビューした
（**送信前に機密識別子をマスクし、残存 0 件を機械確認**）。

| AI | Status | HIGH | MEDIUM | LOW | 判定 |
|----|--------|------|--------|-----|------|
| Session self | ✅ Complete | 0 | 3 | 2 | NEEDS FIXES（自己検出） |
| Codex CLI | ✅ Complete | 1 | 4 | 1 | NEEDS FIXES |
| Cursor Agent | ✅ Complete | 2 | 3 | 2 | NEEDS FIXES |
| Copilot CLI | ✅ Complete | 2 | 3 | 1 | NEEDS FIXES |
| Antigravity (agy) | ✅ Complete | 0 | 2 | 2 | APPROVE WITH NITS |
| Devin CLI | ⚠ 未実施 | — | — | — | 非対話モードでツール確認が要求され停止。`--permission-mode dangerous` は付与しない判断 |
| Kiro CLI | ⚠ 未実施 | — | — | — | 本環境の `kiro` は IDE の CLI で、`kiro chat` が stdout に出力しない（極小プロンプトでも空） |
| Grok CLI | ⚠ 除外 | — | — | — | ユーザー指示により今回除外 |

### 採用した指摘

| # | 指摘元 | Severity | 内容 | 対応 |
|---|--------|:--:|---|---|
| 1 | Codex / Cursor | **HIGH** | 「2.6.1 より前のワークフロー拒否」を **2.7.0 が追加した前提**として書いていた。2.7.0 はコード変更ゼロなので両立しない | ✅ **2.6.1 の永続 state スキーマ v8 の再掲**と書き直した（16.4 / 6.4） |
| 2 | Copilot / Cursor / self | **HIGH** | 非公開側レポートの変更内訳表が合計 76 で、総数 74 と合わない | ✅ 再集計（`docs` 22→18、`scripts` 2 を追加）。4+28+18+15+5+2+2=74 |
| 3 | self | **HIGH** | 16.1 の表で「v2 系のタグ無し」と書いていたが、`v2.1.1` / `v2.2.0` / `v2.3.0` は以前から存在する | ✅ 「GitHub Release の Latest が `v1.0.1` → `v2.7.0`」に訂正。**2.x が初めてリリース公開された**ことを新知見として追記 |
| 4 | self | MEDIUM | roadmap から外れた 7 件を「いずれも出荷済みへ移った」と書いたが、**#754 は移動先が無い** | ✅ 6 件と #754 を分け、#754 は「移動先が無い。取り下げか記載漏れか判断できない」と明記 |
| 5 | self | MEDIUM | `.github/ISSUE_TEMPLATE/` を「1.x の main から引き継がれた」と書いたが、**`v1` に存在しない** | ✅ `labeler.yml` のみ引き継ぎ、テンプレート類はこの移行で新設と訂正 |
| 6 | Cursor | MEDIUM | 「履歴の書き換えなし」が `main` の force-update と衝突しうる | ✅ 「旧 v2 先端→現 main は線形。ただし `main` 自体は force-update。旧 main の clone は `reset --hard` か再 clone」と書き分け |
| 7 | Codex / Cursor | MEDIUM | 「上げる作業は dist 再コピーだけ」が同文書内の注意と矛盾 | ✅ 「配布物の差替えは再コピーで足りる。ただし取得元が `main` に変わり、既存 state と履歴は残る」と分けた |
| 8 | Codex | MEDIUM | 非公開側で「数値も機序も 1 つも動いていない」が過大な一般化 | ✅ 「本レポート群が依拠する数値と機序」に射程を限定 |
| 9 | Antigravity / Cursor | MEDIUM | 非公開側の追記ブロックと差分表の時系列が逆転（2.6.55 → 2.7.0 → 2.6.123） | ✅ 4 ファイルで昇順に並べ直した |
| 10 | Antigravity / Codex / Cursor / self | LOW | 16.8 の本文が「10 章から 15 章の `基準:` 行」なのに表は 11〜15 章 | ✅ 10 章は `対象:` 行で HEAD SHA を持たない旨を明記。SOURCES / TRACEABILITY の同一記述も同期 |
| 11 | Cursor | LOW | 16.2 の不変メトリクス表と CONVERSATION_LOG の列挙に **センサー 6** が抜けている | ✅ 追加（両 ref で `core/sensors/` はバイト単位で同一を確認） |
| 12 | Antigravity | LOW | 非公開側の比較レポート冒頭の「差分の全体像」参照先が前区間のまま | ✅ 最新区間へ更新 |
| 13 | Codex | LOW | 追従実績表に「3 日で 2 件」の行を足したのに「ペースは落ちていない」評価が残る | ✅ 評価の射程が 3 期目までである旨と、**追従コストが件数に比例しない**ことを注記 |
| 14 | self | LOW | 「2.6.1 〜 2.6.124 の 124 版分」は誤り（欠番がある）／「2.6.1 は 2026-08-14 頃」 | ✅ 「2.6.x サイクル全体（公開 CHANGELOG 上 95 エントリ）」／CHANGELOG 日付 **2026-08-13** に訂正 |

### 却下した指摘

| 指摘元 | 主張 | 裁定 |
|--------|------|------|
| Copilot | `10-release-impact-*.md` は実在しないので TRACEABILITY の `10-`〜`16-` は誤り | **却下**。`10-release-impact-2537.md` は実在する。レビュー用パケットに全ファイルを含めなかったための誤認 |
| Cursor | release-impact 章は `11-` からなので範囲表記を `11-` に直すべき | **却下**（同上） |
| Copilot | `v1` の HEAD SHA・タグ時刻・`release.yml` の内部実装・Mermaid 修正の動機などは一次事実に無く未検証の断定 | **却下**。いずれも上流クローンで直接確認済み（Mermaid の動機は `518578ad` のコミットメッセージの逐語）。ただし **ISSUE_TEMPLATE の由来だけは実際に誤り**だったので採用（上表 #5） |
| Copilot | CHANGELOG 日付 2026-09-01 は未確認 | **却下**。`## [2.7.0] - 2026-09-01` を直接確認済み |

### オーケストレータの判定

- **ブロッキング HIGH: 0**（HIGH 3 件はいずれも本改訂内の記述誤りで、**すべて修正済み**）
- 過去章の日付付き測定記録は本文を 1 行も書き換えていない（Cursor が明示的に確認）
- 内部リンク・アンカーは機械検査で全解決
- 公開リポジトリの機密語スキャン: **残存 0 件**
- **判定: APPROVE**（マージ可）
