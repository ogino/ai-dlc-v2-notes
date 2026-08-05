# Remaining Tasks

> 未着手・検討中。完了したら削除またはアーカイブする。

---

## 公開後の任意改善

- [ ] LOW 指摘の文言改善（「端到端」→「エンドツーエンド」等）
- [ ] Spec PDF / Method Paper 全文精読後の差分追記
- [ ] `docs/reference/*` 精読後のエンジン内部メモ追加
- [ ] `docs/guide/agents/*.md`・`workshop-mode.md` が 2.5.11 時点で既に存在したか（新規追加か索引漏れか）の切り分け

## 上流追従（2.5.37 まで同期済み）

- [x] 上流 2.5.11 → 2.5.37 の差分反映（2026-08-05）
- [ ] **追従フローの定例化** — 上流には in-place upgrade も版数比較の仕組みも無い（`docs/roadmap.md` #535 未実装）ため、追従は運用で担保するしかない。担当者と頻度を決める
  - 確認コマンド（**上流リポジトリのローカル clone 内**で実行。本リポジトリには `core/` は無い）: `rg 'AIDLC_VERSION' core/tools/aidlc-version.ts` と CHANGELOG の差分

## 運用

- [ ] GitHub リポジトリの Topics / Description 整備
- [ ] 必要なら GitHub Pages や簡易目次の追加
- [ ] **`DENYLIST_PATTERNS` シークレットの設定**（未設定の間はリークチェックが**失敗する**。fail-closed のため、設定するまで PR はマージできない。手順は `PUBLIC_CONTENT_POLICY.md`）
- [ ] main ブランチ保護ルールの設定（直接 push 禁止・PR 必須・CI パス必須）
