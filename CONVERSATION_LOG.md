# Conversation Log

> 直近のセッションサマリー。詳細は `docs/conversation-archive/` を参照。

---

## Session 2 (2026-08-05)

**作業内容**: 上流 2.5.11 → 2.5.37（12リリース）の差分調査と反映、公開／非公開の切り分け

**調査体制**: マルチエージェント並列（内部監査4本 ＋ 外部AI ＋ 機械的事実確認）。主要な指摘はオーケストレータが実ファイルで独立再検証。

**中核メトリクスは不変**（5フェーズ / 32ステージ / 14エージェント / 9スコープ / 5センサー / 監査74イベント）。変化は hooks 13→14、tools 32→35。

**主な反映**:
- README / 02 / SOURCES のバージョン表記を 2.5.37 に更新
- 02: tools 本数の「出典差」注記は解消（README 34 = 実測 34。35本目の `aidlc.ts` はパターン外）
- 03: Reverse Engineering の 9 成果物は space レベル codekb ストアに書かれる旨を追記（2.5.26）
- 04: reviewer のエンジン強制とカバレッジ（32中12、うち8がCONDITIONAL）、チェックリストのビルド時埋め込み（2.5.33）
- 05: ARS 演算の決定論化（2.5.25）、`freeform_default`（2.5.30）
- 06: `codekb-scope-diff` / `aidlc-workspace-sync` の直接ツール呼び出し
- 07: `load-steering` による確定配信（2.5.33）、センサーの fail-open と堅牢化（2.5.31 / 2.5.32）、`repos.json`
- 08: 8.3 表に 12 リリース分を追加
- **新章 `10-release-impact-2537.md`** を追加

**ソース読解で確認した挙動**（新章に記載）:
- センサーは fail-open。FAILED は「exit 0 かつ `pass === false`」の1経路のみ
- 承認ゲートは human presence をエンジンが機械的に強制（例外3つ: autonomous / 空台帳 fail-open / 環境変数）
- CONDITIONAL 21ステージのスキップ経路には presence 検査が無い
- 上流には版数比較も in-place upgrade も無く、追従は運用で担保するしかない

**公開コンテンツ方針の整備**: `PUBLIC_CONTENT_POLICY.md` と `.github/workflows/leak-check.yml` を追加。公開判断の基準は「上流 OSS の URL を示せば読者が同じ結論に到達できるか」、判定は章単位ではなく文単位で行う。禁止語リストはリスト自体が機密になりうるため平文で置かず、GitHub Actions Secrets から CI 実行時のみ供給する設計とした。

**ブランチ**: `feature/sync-upstream-2537`（マージはユーザー判断）

---

## Session 1 (2026-07-28)

**作業内容**: AWS AI-DLC Workflows 2.0 の調査・日本語整理ノート作成、8-AI レビュー、MEDIUM 修正、公開準備

**成果物**:
- `01`〜`09` 整理ノート、`README.md`、`SOURCES.md`
- `REVIEW-8AI.md`（ドキュメント精度レビュー統合）
- MEDIUM（M1–M10）反映
- 公開向け: `LICENSE`、`.gitignore`、`docs/*`、会話ログ、個人パス除去

**8-AI レビュー（ドキュメント精度）**:
- ブロッキング HIGH: 0（一次ソースで裁定）
- MEDIUM: 対応済み
- LOW: 未対応（任意）

**公開可否 8-AI レビュー**: privacy clean / HIGH なし。README 免責・ライセンス区別・作業ログ案内を追加対応。Devin は並列時に空出力 → **単独リトライで成功**（結果を `REVIEW-8AI.md` に追記）。

**ブランチ**: `feature/prepare-public-release`  
**コミット**: 初期 `772baf7` + 公開 nit 修正（後続）  
**注意**: リモート push はユーザー最終確認後のみ（未実施）
