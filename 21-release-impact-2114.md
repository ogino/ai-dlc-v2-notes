# 21. リリース差分 2.11.3 → 2.11.4 — Kiro harness 統合、team Unit ギャップ解消

作成日: 2026-11-18
基準: **タグ `v2.11.3`（`a8721be3`）→ タグ `v2.11.4`（`61fdb122`）**
前回: [20-release-impact-2113.md](./20-release-impact-2113.md)
出典: タグ `v2.11.4` の `CHANGELOG.md`、GitHub compare、`core/tools/aidlc-version.ts`、`.github/workflows/release.yml`。資産数は再測定していない。

## 21.1 測定

`v2.11.4` は 2026-11-13T05:23:39Z の promotion commit で、`v2.11.4-preview.20261112.1` を安定版へ上げたもの。公開は 2026-11-13T05:48:13Z。`AIDLC_VERSION` は `2.11.4`。

GitHub compare の `v2.11.3...v2.11.4` は 293 commits 先行、behind 1、diverged。ファイル一覧は 300 files で打ち切られるため、行数は記さない。

2026-11-18 の非安定追跡:

| 区間 | commits | diff | 備考 |
|---|---:|---|---|
| `v2.11.4..v2.11.5-preview.20261117.1` | 20 | 125 files / +5,066 / −761 | `ce29ceab`。`AIDLC_VERSION` は `2.11.4` |
| `v2.11.4..origin/main` | 44 | 179 files / +9,783 / −1,107 | `e67406fe`、2026-11-18T10:24:25Z。`AIDLC_VERSION` は `2.11.4` |

preview と `main` は同一 SHA ではない。版文字列だけでは `v2.11.4` と後続 preview / `main` を区別できない。

## 21.2 CHANGELOG で確認した運用差分

実機確認はしていない。

1. `--harness kiro` は Kiro IDE と Kiro CLI v3 の共通導入。Kiro CLI V2 は `--harness kiro-v2`。`kiro-ide` は残る。既存プロジェクトの refresh は、そのプロジェクトが持っている Kiro harness を維持する。
2. v2.11.3 の team Unit ギャップ 3 件は CHANGELOG 上は閉じた。skipped stage が end-of-Unit 承認を止めない。merge question は open finding を clean pass と表示しない。claim checkout の承認済み stage 変更は、人の変更として reopen する。
3. team の受け渡しは、期限が来た操作を 1 問にまとめる。
4. `$`、backtick、`>`、`<`、`&`、改行を含む Kiro の依頼は file 経由で渡る。`caea995a` は安定 `v2.11.4` に含まれる。
5. Copilot CLI で `shell(bun)` または `shell(aidlc)` を拒否しているなら、`.aidlc/bin/aidlc-engine` も拒否対象に加える。
6. Windows で execution policy が `AllSigned` / `Restricted` の場合、または uninstall journal が壊れている場合、`aidlc update` ではなく one-line installer が必要。
7. `infra` plan は Code Generation と Build and Test を含む 15 stages。適用は Environment Provisioning。
8. `release.yml` の `native-smoke` は引き続き `tests/unit/t238-build-binaries.test.ts` を実行する。full suite job は確認範囲では見当たらなかった。

## 21.3 引用しない値

`docs/` 数、`core/tools/*.ts` 数、監査イベント数は v2.11.4 で測っていない。20 章の v2.11.3 値を v2.11.4 の現在値として引用しない。
