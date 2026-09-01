# 追跡性

## 正本

| 種別 | 正本 |
|------|------|
| AI-DLC 実装・仕様 | [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows) の **`main`** ブランチ（`docs/`・`core/`・`CHANGELOG`）※ |
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
| `10-`〜`16-release-impact-*.md` | 版ごとの日付付き測定記録（本文は書き換えない） |

※ **2026-09-01 以前は `v2` ブランチだった。**`v2` は上流で削除され、`main` がその線形な継続である。
11〜15 章の `基準:` 行が記録している HEAD SHA は、いずれも `main` から到達できる（10 章は SHA を持たない）
（→ [16-release-impact-2700.md](../16-release-impact-2700.md)）。`v2_backup` というヘッドが残っているが、
**旧 `v2` HEAD のコピーではない**ので参照先にしないこと。

## 更新方針

1. 上流に追従する事実変更 → 該当ノート章 + SOURCES の精読範囲を更新
   （区間差分は新しい `NN-release-impact-*.md` を起こし、README の目次と
   `docs/REMAINING_TASKS.md` の追従記録にも行を足す）
2. 作業セッション終了時 → CONVERSATION_LOG + archive を更新
3. 公開前 → privacy スキャンとレビュー結果を記録
