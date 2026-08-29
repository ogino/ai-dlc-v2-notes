# Remaining Tasks

> 未着手・検討中。完了したら削除またはアーカイブする。

---

## 公開後の任意改善

- [ ] LOW 指摘の文言改善（「端到端」→「エンドツーエンド」等）
- [ ] Spec PDF / Method Paper 全文精読後の差分追記
- [ ] `docs/reference/*` 精読後のエンジン内部メモ追加
- [ ] `docs/guide/agents/*.md`・`workshop-mode.md` が 2.5.11 時点で既に存在したか（新規追加か索引漏れか）の切り分け

## 上流追従（2.6.123 まで同期済み）

- [x] 上流 2.5.11 → 2.5.37 の差分反映（2026-08-05）
- [x] 上流 2.5.37 → 2.5.62 の差分反映（2026-08-11）→ [11-release-impact-2562.md](../11-release-impact-2562.md)
- [x] 上流 2.5.62 → 2.6.2 の差分反映（2026-08-14。HEAD `4569754e`）→ [12-release-impact-2602.md](../12-release-impact-2602.md)
- [x] 上流 2.6.2 → 2.6.49 の差分反映（2026-08-22。HEAD `71d9a9e0`）→ [13-release-impact-2649.md](../13-release-impact-2649.md)
- [x] 上流 2.6.49 → 2.6.55 の差分反映（2026-08-22。HEAD `840ba653`）→ [14-release-impact-2655.md](../14-release-impact-2655.md)
- [x] 上流 2.6.55 → 2.6.123 の差分反映（2026-08-29。HEAD `2fbee12f`）→ [15-release-impact-26123.md](../15-release-impact-26123.md)
- [ ] **追従フローの定例化** — 上流には in-place upgrade も版数比較の仕組みも無い（`docs/roadmap.md` #535 未実装）ため、追従は運用で担保するしかない。担当者と頻度を決める
  - 確認コマンド（**上流リポジトリのローカル clone 内**で実行。本リポジトリには `core/` は無い）: `rg 'AIDLC_VERSION' core/tools/aidlc-version.ts` と CHANGELOG の差分
- [ ] **2.6.1 の破壊的変更を読者向け移行手順として点検する** — 永続 state が v8 に上がり、`/aidlc next` / `/aidlc report` / `/aidlc --doctor` が pre-v8 state を拒否する。アップグレード時は `skills/aidlc-application-design/` の**手動削除**が要る（`cp -R` マージでは残る）。実機での再現は未実施

## 2.6.2 調査で残った未確認事項

- [ ] `04-agents.md:107` の「`Task` はエージェント禁止」— 全ハーネスの agent 定義を走査して確認していない
- [ ] `05-scopes-depth-test.md:87-88` の `aidlc-graph.ts ars` サブコマンド引数の完全一致
- [x] 「どのハーネスの設定にもメトリクス送出エンドポイントは同梱されていない」— **Cursor を含む `dist/` 全体を再走査し、2.6.2 でも同梱ゼロを確認済み**（2026-08-14）
- [x] `11-release-impact-2562.md:178` の `tools/data/model-rates.json` — **`core/tools/data/model-rates.json` として実在を確認済み**（2026-08-14）
- [ ] `11-release-impact-2562.md:8` の「変更ファイル 1,425 件」— 2.5.37 → 2.5.62 区間の再計測は未実施
- [ ] Cursor ハーネス: `bun scripts/package.ts cursor --check` による dist ドリフト検証、および上流テストの実行（調査環境に `bun` が無い）
- [ ] 2.5.63 が core の `aidlc-review-freeze.ts` に入れたシェルラッパ剥がし強化（`sudo` / `env` / `xargs` / `timeout` 等）が、**既存ハーネスで false positive を生むか**未検証

## 2.6.49 調査で残った未確認事項

- [ ] プロトコルモジュール分割（4 → 8）による固定コンテキスト削減の定量値 — 上流が数値化していない
- [ ] `tests/integration/t304` / `t307` のループバックテストが「3 回」の数値上限自体をアサートしているか — ファイル存在のみ確認
- [ ] DocumentKB の後続段階（「S2」相当）の正式名称・時期 — 上流 issue #714 として予告されるのみ
- [ ] `core/tools/aidlc-swarm.ts` 本体が Testing Contract を検証する具体的コード位置
- [ ] `bun scripts/package.ts codex trust` の内部ハッシュ生成ロジックが 2.6.44 で変わったか
- [ ] Kiro IDE の新 `.kiro/hooks/*.json` が実機 Kiro IDE 1.x で実際に SessionStart 発火するか — 実機未検証
- [ ] `<active-space>` プレースホルダが claude / codex / cursor / kiro / kiro-ide の現行 dist に残っていないかの横断 grep — 該当コミットの変更ファイル範囲でのみ確認
- [ ] 上流ドキュメントの stale 3 箇所（`docs/guide/05-scopes-and-depth.md` のキーワード表 fallback 行、`docs/reference/03-orchestrator.md`、`core/scopes/aidlc-feature.md` 本文）が意図的な緩さか単なる更新漏れか — 上流 Issue / PR 未参照

## 2.6.55 調査で残った未確認事項

（`scratch/` は Git 管理外の作業ディレクトリなので、根拠は本リポジトリには残らない。判断の経緯は [14-release-impact-2655.md](../14-release-impact-2655.md) の 14.9 を参照）

- [ ] 2.6.54 より前に生成された**実データ**の監査シャードに、無関係な `SUBAGENT_COMPLETED` が
      実際に混入しているか — 調査環境にサンプルが無く、コード構造から導かれる可能性のみ
- [ ] 上流 issue #695 の `Directory not found` fumble が本当に `memory.md` のプローブだったか —
      `Closes #695` は上流の主張で、差分中に両者を結ぶ記述は無い
- [ ] read-probe の失敗が、Kiro CLI を ACP 経由で動かした場合以外（他の 6 ハーネス、および Kiro CLI の TUI 経路）でも起きていたか — 上流にも実測が無い（**ACP はハーネスではなく Kiro CLI の接続方式**）
- [ ] 2.6.55 による改善量（削減されたターン数）— 上流も定性的記述にとどまる
- [ ] 「新旧混在はロードエラー」は ESM の named-import 契約からの**推論**。
      実際に半々のツリーで実行して確かめていない
- [ ] ネットワーク FS（NFS / SMB / FUSE / オブジェクト同期フォルダ）上で、
      実際にどのタイミングでどのエラーが出るか — 上流の unsupported 宣言を確認したのみ
- [ ] `AIDLC_SKIP_REVIEWER_GATE_GUARD=1` を一般利用者が設定できないようにする機構の有無
- [ ] 上流テストスイートは未実行（調査環境に `bun` が無い）

## 2.6.123 調査で残った未確認事項

（`scratch/` は Git 管理外の作業ディレクトリなので、根拠は本リポジトリには残らない。判断の経緯は [15-release-impact-26123.md](../15-release-impact-26123.md) を参照）

- [ ] `bugfix` / `refactor` の**旧**承認ゲート数の絶対値 — 新値 6 / 7 は CHANGELOG が明示しているが、
      旧 `bugfix` 4 は旧ドキュメントの例から読めるだけで、旧 `refactor` は算術推定にとどまる
- [ ] 2.6.121 の `incompleteFallback` が advisory ステージで実際にゲートを開けるか — コード読解による推論
- [ ] `Review Challenge` を conductor が自作して受領証を通せる具体的攻撃経路 — 同上
- [ ] 2.6.74（品質目標）の実効性 — エージェントが実際に閾値の緩和を拒否するか
- [ ] 2.6.116 / 2.6.112 の効果 — 散文ルールのみでガードが無いため、動作としては未検証
- [ ] 2.6.71 の localization が各ハーネスでどこまで日本語化されるか
- [ ] 2.6.119 / 2.6.117 の文脈削減が実トークン消費・コスト・レイテンシをどれだけ下げるか
      — 測ったのは Git オブジェクトのバイト数のみ
- [ ] `audit-format.md` の "Interaction Events (10 events)" と実際の表行数（9 行）の不一致
- [ ] 2.6.92 の混在セパレータ問題が実データで悪用可能だったか — コード構造上の経路のみ確認
- [ ] Copilot / OpenCode がエンジン再インストール後に compose フックで自己修復するか
      — 上流は Claude / Codex / Cursor / Kiro IDE と Kiro CLI しか名指ししていない
- [ ] `aidlc-plugin-validate.ts` と compose のテストペイロード判定の非対称
      （validate は `tests` / `fixtures` / `*.test.ts` のみ error、compose は `__tests__` / `*.spec.ts` も drop）が
      意図的か検証漏れか
- [ ] 上流 `10-authoring-a-plugin.md` が「neither an AIDLC project nor a framework checkout」と書く一方、
      `aidlc-plugin-test.ts` は `--install <project-root>` が必須 — 誤記か意図的か断定できない
- [ ] 2.6.99 の active-directive ロックが `withAuditLock` と物理的に同一実装か — シンボルレベルまで未追跡
- [ ] 上流テストスイートは未実行（調査環境に `bun` が無い）— **区間をまたいで継続中の項目**（初出は「2.6.55 調査で残った未確認事項」。本区間でも解消していない）

## 運用

- [ ] GitHub リポジトリの Topics / Description 整備
- [ ] 必要なら GitHub Pages や簡易目次の追加
- [ ] **`DENYLIST_PATTERNS` シークレットの設定**（未設定の間はリークチェックが**失敗する**。fail-closed のため、設定するまで PR はマージできない。手順は `PUBLIC_CONTENT_POLICY.md`）
- [x] master ブランチ保護ルールの設定（直接 push 禁止・PR 必須・CI パス必須。2026-08-05 設定済み）
  - **fork PR との相互作用**: leak-check は fork PR で必ず失敗するため（secrets 不達）、外部 PR は管理者バイパスなしにはマージできない。手順は `PUBLIC_CONTENT_POLICY.md` の「fork PR の運用手順」を参照
