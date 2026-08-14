# Remaining Tasks

> 未着手・検討中。完了したら削除またはアーカイブする。

---

## 公開後の任意改善

- [ ] LOW 指摘の文言改善（「端到端」→「エンドツーエンド」等）
- [ ] Spec PDF / Method Paper 全文精読後の差分追記
- [ ] `docs/reference/*` 精読後のエンジン内部メモ追加
- [ ] `docs/guide/agents/*.md`・`workshop-mode.md` が 2.5.11 時点で既に存在したか（新規追加か索引漏れか）の切り分け

## 上流追従（2.6.2 まで同期済み）

- [x] 上流 2.5.11 → 2.5.37 の差分反映（2026-08-05）
- [x] 上流 2.5.37 → 2.5.62 の差分反映（2026-08-11）→ [11-release-impact-2562.md](../11-release-impact-2562.md)
- [x] 上流 2.5.62 → 2.6.2 の差分反映（2026-08-14。HEAD `4569754e`）→ [12-release-impact-2602.md](../12-release-impact-2602.md)
- [ ] **追従フローの定例化** — 上流には in-place upgrade も版数比較の仕組みも無い（`docs/roadmap.md` #535 未実装）ため、追従は運用で担保するしかない。担当者と頻度を決める
  - 確認コマンド（**上流リポジトリのローカル clone 内**で実行。本リポジトリには `core/` は無い）: `rg 'AIDLC_VERSION' core/tools/aidlc-version.ts` と CHANGELOG の差分
- [ ] **2.6.1 の破壊的変更を読者向け移行手順として点検する** — 永続 state が v8 に上がり、`next` / `report` / `--doctor` が pre-v8 state を拒否する。アップグレード時は `skills/aidlc-application-design/` の**手動削除**が要る（`cp -R` マージでは残る）。実機での再現は未実施

## 2.6.2 調査で残った未確認事項

- [ ] `04-agents.md:107` の「`Task` はエージェント禁止」— 全ハーネスの agent 定義を走査して確認していない
- [ ] `05-scopes-depth-test.md:87-88` の `aidlc-graph.ts ars` サブコマンド引数の完全一致
- [x] 「どのハーネスの設定にもメトリクス送出エンドポイントは同梱されていない」— **Cursor を含む `dist/` 全体を再走査し、2.6.2 でも同梱ゼロを確認済み**（2026-08-14）
- [x] `11-release-impact-2562.md:178` の `tools/data/model-rates.json` — **`core/tools/data/model-rates.json` として実在を確認済み**（2026-08-14）
- [ ] `11-release-impact-2562.md:8` の「変更ファイル 1,425 件」— 2.5.37 → 2.5.62 区間の再計測は未実施
- [ ] Cursor ハーネス: `bun scripts/package.ts cursor --check` による dist ドリフト検証、および上流テストの実行（調査環境に `bun` が無い）
- [ ] 2.5.63 が core の `aidlc-review-freeze.ts` に入れたシェルラッパ剥がし強化（`sudo` / `env` / `xargs` / `timeout` 等）が、**既存ハーネスで false positive を生むか**未検証

## 運用

- [ ] GitHub リポジトリの Topics / Description 整備
- [ ] 必要なら GitHub Pages や簡易目次の追加
- [ ] **`DENYLIST_PATTERNS` シークレットの設定**（未設定の間はリークチェックが**失敗する**。fail-closed のため、設定するまで PR はマージできない。手順は `PUBLIC_CONTENT_POLICY.md`）
- [x] master ブランチ保護ルールの設定（直接 push 禁止・PR 必須・CI パス必須。2026-08-05 設定済み）
  - **fork PR との相互作用**: leak-check は fork PR で必ず失敗するため（secrets 不達）、外部 PR は管理者バイパスなしにはマージできない。手順は `PUBLIC_CONTENT_POLICY.md` の「fork PR の運用手順」を参照
