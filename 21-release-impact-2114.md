# 21. リリース差分 2.11.3 → 2.11.4 — Kiro harness 統合、team Unit ギャップ解消

作成日: 2026-11-18
基準: **タグ `v2.11.3`（`a8721be3`）→ タグ `v2.11.4`（`61fdb122`）**
前回: [20-release-impact-2113.md](./20-release-impact-2113.md)
出典: ローカル clone のタグ `v2.11.4`、`CHANGELOG.md`、`core/tools/aidlc-version.ts`、`.github/workflows/release.yml`。監査イベント数などの内部資産数は再測定していない。

## 21.1 測定

`v2.11.4` は 2026-11-13T05:23:39Z の promotion commit で、`v2.11.4-preview.20261112.1` を安定版へ上げたもの。公開は 2026-11-13T05:48:13Z。`AIDLC_VERSION` は `2.11.4`。

ローカル clone の `git rev-list --count v2.11.3..v2.11.4` は 293、`git diff --shortstat v2.11.3...v2.11.4` は 808 files / +106,747 −12,434。

非安定追跡（評価終点ではない。行ごとの日付を見る）:

| 区間 | commits | diff | 備考 |
|---|---:|---|---|
| `v2.11.4..v2.11.5-preview.20261118.1` | 47 | 190 files / +10,525 / −1,178 | `ff2452ba`、2026-11-18T16:41:27Z。`AIDLC_VERSION` は `2.11.4` |
| `v2.11.4..c515d065`（2026-11-18 夜の `origin/main`） | 49 | 196 files / +10,895 / −1,304 | 2026-11-18T22:17:33Z。`AIDLC_VERSION` は `2.11.4` |
| `v2.11.4..v2.11.5-preview.20261121.1` | 62 | 233 files / +13,313 / −1,794 | `7098f0ef`、2026-11-21T17:46:05Z。`AIDLC_VERSION` は `2.11.4` |
| `v2.11.4..464626b6`（2026-11-22 の `origin/main`） | 69 | 251 files / +14,636 / −1,944 | 2026-11-22T11:19:24Z。`AIDLC_VERSION` は `2.11.4` |
| `v2.11.4..v2.11.5-preview.20261123.1` | 83 | 281 files / +19,314 / −2,320 | `1e64f313`、2026-11-23T17:52:29Z。`AIDLC_VERSION` は `2.11.4` |
| `v2.11.4..441b73ac`（2026-11-24 の `origin/main`） | 89 | 293 files / +21,131 / −2,592 | 2026-11-24T08:56:15Z。`AIDLC_VERSION` は `2.11.4` |

preview と `main` は同一 SHA ではない。版文字列だけでは `v2.11.4` と後続 preview / `main` を区別できない。

## 21.2 CHANGELOG で確認した運用差分

実機確認はしていない。

1. `--harness kiro` は Kiro IDE と Kiro CLI v3 の共通導入。Kiro CLI V2 は `--harness kiro-v2`。`kiro-ide` は残る。既存プロジェクトの refresh は、そのプロジェクトが持っている Kiro harness を維持する。
2. v2.11.3 の team Unit ギャップ 3 件は CHANGELOG 上は閉じた。skipped stage が end-of-Unit 承認を止めない。merge question は open finding を clean pass と表示しない。claim checkout の承認済み stage 変更は、人の変更として reopen する。
3. team の受け渡しは、次の受け渡しが必要になった段階で 1 問になる。期限でまとめる機能ではない。
4. `$`、backtick、`>`、`<`、`&`、改行を含む Kiro の依頼は file 経由で渡る。`caea995a` は安定 `v2.11.4` に含まれる。
5. Copilot CLI で `shell(bun)` または `shell(aidlc)` を拒否しているなら、`.aidlc/bin/aidlc-engine` も拒否対象に加える。
6. Windows では、Group Policy が execution policy を `AllSigned` / `Restricted` にしている場合、または 2.11.3 が invalid uninstall journal で起動できない場合、2.11.4 への更新に one-line installer が一度必要。その後は通常の更新に戻る。
7. `infra` plan は Code Generation と Build and Test を含む 15 stages。適用は Environment Provisioning。
8. `release.yml` の `native-smoke` は引き続き `tests/unit/t238-build-binaries.test.ts` を実行する。full suite job は確認範囲では見当たらなかった。

## 21.3 引用しない値

`docs/` 数、`core/tools/*.ts` 数、監査イベント数は v2.11.4 で測っていない。20 章の v2.11.3 値を v2.11.4 の現在値として引用しない。
