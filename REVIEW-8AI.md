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
| M7 | CodeKB は Kiro で workspace-scan 固定 | ✅ 反映済 |
| M8 | Codex trust 手順の補足 | ✅ 反映済 |
| M9 | 1.x 対比の留保強化 | ✅ 反映済 |
| M10 | Spec PDF は `assets/`、正本優先順位 | ✅ 反映済 |

LOW は対応任意（未対応）。

---

## 正確と確認された中核

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
| Devin CLI | ❌ 未実施（出力なし） | — | — |
| Kiro CLI | ✅ | clean | MEDIUM は免責強化等 |

### 公開レビューで合意した MEDIUM と対応

| 指摘 | 対応 |
|------|------|
| 冒頭の非公式・二次整理バナー強化 | ✅ README 冒頭に免責ブロック |
| 上流 MIT-0 と本リポ MIT の混同 | ✅ メトリクス表を分離 |
| 作業ログを公開資料と並べすぎ | ✅ README で「公開向け / メンテナ向け」分割 |
| 09 見出し「ローカルパス」 | ✅ 「上流リポジトリ内パス」へ変更 |

### 公開可否（オーケストレータ）

- **HIGH ブロッカー: なし**（privacy: clean が複数 AI で一致）
- **Devin: 未実施**（理由: CLI が出力なしで終了）
- **判定: PUBLISH WITH NITS（nit は上記対応済み）**
- **リモート push はユーザー最終確認後のみ**
