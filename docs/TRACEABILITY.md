# 追跡性

## 正本

| 種別 | 正本 |
|------|------|
| AI-DLC 実装・仕様 | [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows) の **v2** ブランチ（`docs/`・`core/`・`CHANGELOG`） |
| 方法論の導入 | AWS DevOps ブログ / Method Definition Paper |
| 本リポジトリ | 二次整理。推測は SOURCES / 各章の留保に従う |

## 本リポジトリ内の記録

| ファイル | 役割 |
|----------|------|
| `CONVERSATION_LOG.md` | 直近セッション要約 |
| `docs/conversation-archive/` | セッション詳細 |
| `SOURCES.md` | 調査ソースと精読範囲 |
| `REVIEW-8AI.md` | 複数 AI レビュー結果 |
| `docs/REMAINING_TASKS.md` | 未完了タスク |

## 更新方針

1. 上流に追従する事実変更 → 該当ノート章 + SOURCES の精読範囲を更新
2. 作業セッション終了時 → CONVERSATION_LOG + archive を更新
3. 公開前 → privacy スキャンとレビュー結果を記録
