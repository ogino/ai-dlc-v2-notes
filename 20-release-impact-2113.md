# 20. リリース差分 2.10.0 → 2.11.3 — Guard Policy off 既定、Bun CVE、チーム Unit

作成日: 2026-11-05
基準: **タグ `v2.10.0`（`2a883858`）→ タグ `v2.11.3`（`a8721be3`）**（取得 2026-11-05）
前回: [19-release-impact-2100.md](./19-release-impact-2100.md)（2.9.0 → 2.10.0）

コミット **1,575 件**、変更ファイル **1,813 件**（+537,484 −89,414 行）、期間 **約 6 週間**（上流コミット日 2026-09-24 〜 2026-11-04）。
`v2.11.3` の公開は **2026-11-04 05:46:34 UTC**。

**要点は 6 つある。**

1. **セキュリティ: ネイティブ `aidlc` が Bun 1.4.4 でビルドされた**（CVE-2026-111497 対応）
   2.11.2 以前のネイティブバイナリは影響を受けるため、`aidlc update` で更新する（→ 20.3）
2. **Guard Policy の出荷既定が `enterprise` を除く全スコープで `off` になった**（v2.10.0 は `relaxed` 既定）（→ 20.4）
3. **`collaborators` と `plan_approval` の frontmatter が導入され、デフォルトで co-agent 停止・express/poc の Plan Approval off になった**（→ 20.4）
4. **ネイティブ導入でのチーム担当 Unit が 2.11.3 で実用経路になった**（v2.10.0 の既知不具合は 2.11.0 で解消）（→ 20.5）
5. **スクリプト・CI 互換性が大きく動いた**（`AIDLC_UNATTENDED`、システムパッケージ導入、opencode 2、Windows `aidlc.exe`、plugin 名称）（→ 20.6）
6. **内部資産・監査イベントが大きく増えた**（docs 103 → 107、`core/tools/*.ts` 76 → 113、監査イベント 105 → 131）（→ 20.7）

---

## 20.1 測定方法と同期地点

この章は上流 clone の `origin/main` とタグを使って測定した。リポジトリの working tree ではなく、`git show <ref>:<path>` で上流ファイルを読んでいる。

```bash
# 同期地点のタグを確認
git ls-remote --heads origin main
git fetch origin main --tags

# タグの実体と最終同期地点
git rev-parse v2.10.0 v2.11.3 origin/main
git rev-parse v2.10.0^{} v2.11.3^{} origin/main

# 範囲計測
git log --oneline --no-merges v2.10.0..v2.11.3 | wc -l
git diff --shortstat v2.10.0..v2.11.3

# ファイル体系の計測
for ref in v2.10.0 v2.11.3 v2.11.4-preview.20261104.1 origin/main; do
  echo "$ref docs=$(git ls-tree -r --name-only "$ref" docs | wc -l)"
  echo "$ref tools=$(git ls-tree -r --name-only "$ref" core/tools | rg '\.ts$' | wc -l)"
  echo "$ref hooks=$(git ls-tree -r --name-only "$ref" core/hooks | rg '\.ts$' | wc -l)"
  echo "$ref protocols=$(git ls-tree -r --name-only "$ref" core/aidlc-common/protocols | wc -l)"
done
```

### 主要タグ

| タグ | peeled commit | commit date | 備考 |
|---|---:|---:|---|
| `v2.10.0` | `2a883858` | 2026-09-24T05:33:54Z | 前回終点 |
| `v2.11.0` | `6a378b53` | 2026-10-08T07:39:40Z | `guard_policy: off` / `collaborators: off` 既定化 |
| `v2.11.1` | `895a35d8` | 2026-10-20T00:22:57Z | Code Generation の worktree / チーム Unit 系の下地 |
| `v2.11.2` | `10a9bc9d` | 2026-10-28T08:37:18Z | worktree review / チーム運用の補強 |
| `v2.11.3` | `a8721be3` | 2026-11-04T05:46:34Z | **本章の終点。Bun 1.4.4 ネイティブビルド** |
| `v2.11.4-preview.20261104.1` | `17c123ca` | 2026-11-04T20:02:51Z | 非安定 preview。本章の終点には含めない |

`v2.11.3`、`v2.11.4-preview.20261104.1`、`origin/main`（`0caa915560b912c23c032f8898724ba624f4d88f`、2026-11-05T06:39:50Z 時点）の `core/tools/aidlc-version.ts` はすべて `AIDLC_VERSION = "2.11.3"` だった。
**版文字列だけを見ると preview / main を区別できないため、導入判断ではタグまたは SHA を使う。**

2026-11-11 再取得でも安定 Latest は `v2.11.3` のまま。最新 preview は `v2.11.4-preview.20261110.1`（`92d39a09`、2026-11-10T17:53:49Z）、`origin/main` は `59b64b0d`（2026-11-11T09:41:02Z）。GitHub compare では preview が 208 commits、`main` が 222 commits 先行し、いずれも tag commit 自体を含まない。ファイル数と行数は compare API が 300 files で打ち切るため、再取得分はコミット数だけを記す。`AIDLC_VERSION` はどちらも `2.11.3`。以前参照した `caea995a` は `v2.11.4-preview.20261108.1` であり、安定 `v2.11.3` には含まれない。

### 範囲の大きさ

| 区間 | commits | diff | 主な top-level dirs |
|---|---:|---:|---|
| `v2.10.0..v2.11.3` | 1,575 | 1,813 files / +537,484 / −89,414 | tests 1,278 / core 232 / harness 115 / docs 93 |
| `v2.10.0..v2.11.0` | 562 | — | — |
| `v2.10.0..v2.11.1` | 1,107 | — | — |
| `v2.10.0..v2.11.2` | 1,480 | — | — |
| `v2.11.3..v2.11.4-preview.20261104.1` | 46 | 149 files / +9,604 / −1,003 | tests 78 / core 30 / docs 21 |
| `v2.11.3..origin/main`（2026-11-05） | 82 | 276 files / +40,114 / −3,540 | tests 146 / core 64 / docs 31 |
| `v2.11.3..v2.11.4-preview.20261110.1`（2026-11-11） | 208 | compare API が 300 files で打ち切り | コミット数のみ採用 |
| `v2.11.3..origin/main`（2026-11-11、`59b64b0d`） | 222 | compare API が 300 files で打ち切り | コミット数のみ採用 |

---

## 20.2 アップグレード手順

`v2.11.3` の CHANGELOG 要旨:

- ネイティブ導入: `aidlc update` を実行し、プロジェクト側で `aidlc config --yes` を実行する。
- ピン留めしているプロジェクトは `aidlc config --pin 2.11.3` で移す。
- 手動コピー運用は完全な `runtime/<harness>/` ツリーを `v2.11.3` の runtime アセットから置き換える。
- opencode を使うプロジェクトは **opencode 1.18.29 以上、または OpenCode 2** を要求する。

**v2.10.0 から直接上げる場合も、まず 2.11.0 / 2.11.1 の breaking changes を読む。**
v2.11.3 だけを見ると、`AIDLC_UNATTENDED` と plugin カタログ変更を過小評価する。

---

## 20.3 🔴 Security: Bun 1.4.4 と CVE-2026-111497

`v2.11.3` の native `aidlc` は **Bun 1.4.4** でビルドされた。これは CISA が actively exploited と列挙する **CVE-2026-111497**（Bun に同梱される Brotli decoder の out-of-bounds write）への対応である。

CHANGELOG の重要な注意:

- 2.11.2 以前のネイティブバイナリは該当 decoder を内包する。
- ** Bun を個別更新しても、ネイティブバイナリは更新されない。**
- `aidlc update` と、必要なら `aidlc config --pin 2.11.3` が要る。
- `AIDLC_METRICS_ENDPOINT` が `http://` の場合、collector の応答が decoder に届くため、HTTPS へ移す。
- copy channel では利用する Bun 自体を 1.4.4 以上にする。

これは公開ノート側で **「版の新しさ」ではなく「native バイナリの更新」が必要**と書くべき項目である。

---

## 20.4 🔴 Guard Policy / collaborators / plan_approval — 既定の重心が変わった

### frontmatter 既定値（v2.10.0 → v2.11.3）

`v2.11.0` で `collaborators` と `plan_approval` の frontmatter が導入され、Guard Policy の出荷既定も `enterprise` 以外すべて `off` になった。

| scope | guard_policy v2.10 → v2.11 | collaborators v2.11 | plan_approval v2.11 | summary_confirmation v2.11 |
|---|---:|---:|---:|---:|
| `classic` | `relaxed` → **`off`** | **`off`** | `on` | `off` |
| `bugfix` | `relaxed` → **`off`** | **`off`** | `on` | `on` → **`off`** |
| `express` | `relaxed` → **`off`** | **`off`** | **`off`** | `off` |
| `poc` | `relaxed` → **`off`** | **`off`** | **`off`** | `on` |
| `feature` | `relaxed` → **`off`** | **`off`** | `on` | `on` |
| `mvp` | `relaxed` → **`off`** | **`off`** | `on` | `on` |
| `refactor` | `relaxed` → **`off`** | **`off`** | `on` | `on` |
| `infra` | `strict` → **`off`** | **`off`** | `on` | `on` |
| `security-patch` | `strict` → **`off`** | **`off`** | `on` | `on` |
| `workshop` | `relaxed` → **`off`** | **`off`** | `on` | `on` |
| `enterprise` | `strict` | `on` | `on` | `on` |

**`classic` の既定 `off` は、レビュー済み成果物や承認後計画の変更に対する拘束が「記録つき素通し」からさらに一段下がったことを意味する。**

### 既存 intent

CHANGELOG は、既存 intent が記録済みの Guard Policy・sensors・learnings・summary confirmation を保持すると述べている。
一方、`plan_approval` と `collaborators` は以前 record に含まれていなかったため、**スコープの新しい既定に従う**。
したがって、**`enterprise` 以外の進行中 intent は、明示的に `/aidlc --collaborators on` を打たない限り lead agent だけで走る**。

### `/aidlc --collaborators` と `--plan-approval`

- `/aidlc --collaborators on` は support agents を戻す。既定 off。
- `/aidlc --plan-approval on|off` はコード計画の承認可否を変える。`express` / `poc` は off 出荷。
- `human-presence` は切替不能。これは人間の最終承認を機械的に回避するためのスイッチではない。

### CI / 無人実行

v2.11.2 で **「`AIDLC_UNATTENDED=1` を設定せず、自律 Construction でもない実行は人間が駆動している扱い」** になった。
CI やスクリプトから AI-DLC を走らせるには、**`AIDLC_UNATTENDED=1` を明示する**必要がある。

---

## 20.5 チーム担当 Unit — v2.10.0 の blocker と v2.11.3 の実用経路

19 章で記録した v2.10.0 の既知不具合（ネイティブ導入で `Unit Ownership: team` の Construction が始まらない）について、修正コミット `057b13be` は **`v2.11.0` 以降に含まれる**。

```bash
git tag --contains 057b13be --sort=creatordate | rg '^v2\.11\.0$'
```

v2.11.3 ではチーム Unit が Claude Code と GitHub Copilot CLI で end-to-end に動くことを CHANGELOG が明示している。

```
Unit claim → stage 実行 → /aidlc unit publish → /aidlc unit pin → merge question → landing
→ 元 checkout は git pull で main を取る
```

ただし v2.11.3 の CHANGELOG は **既知の未解決ギャップ**も書いている。

- end-of-Unit リズムでは、Unit がスキップした stage が end-of-Unit 承認を止めることがある。
- merge question が、finding が残っている stage を「clean pass」と表示することがある。
- Unit claim 中の checkout で承認済み stage へ変更したときに、人間の発言ではなく jump command として記録されることがある。

v2.11.4 preview と origin/main では team Unit 系の修正がさらに多い。したがって **「チーム Unit が動く」は v2.11.3 で安定保証された機能ではなく、2.11.x の開発継続中領域**として扱う。

---

## 20.6 破壊的・運用上の注意

### opencode

`v2.11.3` は OpenCode 2 に対応した。opencode 1 は **1.18.29 以上**が必要。

```bash
aidlc doctor
# warning: Harness CLI: opencode <version> is below 1.18.29
```

`aidlc config runtime --check` は opencode 1.18.29 未満で失敗する（plain `aidlc config --harness opencode --yes` は止まらないが doctor は警告する）。

### Windows native launcher

`v2.11.3` は native install の bin に `aidlc.exe` を書く。PowerShell / CMD / Git Bash で `aidlc.cmd` より先に実行される。
ただし、より古い release の `install.ps1` や `aidlc uninstall` が `aidlc.exe` を検出して失敗・残置する互換性問題がある。
古い版へ戻るには `aidlc use <version>` を使う。

### システムパッケージ導入

AI-DLC が計画外に `sudo apt-get install ...` などシステムパッケージを入れる場合、2.11.3 は人間にコマンドと目的を伝えて待機する。
`AIDLC_UNATTENDED=1` の CI でも保持されるため、CI は **実行前に必要なツールを自分で入れる**必要がある。

### `aidlc config` と pin

v2.11.2 で、copied project の config 実行時、`--from` なしで native install の release files を黙って取り込まなくなった。
run without terminal では exit 4 になる。`--download` または `--from <install root>/versions/<version>/runtime` が必要。

### Bedrock / Claude Code

v2.11.2 で Claude Code の Amazon Bedrock 設定は git-ignored の `.claude/settings.local.json` に書く方針になった。
`.claude/tools/data/harness.json` から AWS profile を読まない。CI は `aidlc config providers --show --json` を使う。

### plugins

v2.11.1 以降、plugin のカタログ名と manifest name が変わる。
`aidlc-<name>@aidlc-plugins` に依存するスクリプトは `<name>@<catalogue>` へ直す。
`aidlc plugin build` の出力フォルダから `marketplace.json` は消え、`plugins` フォルダを marketplace として追加する。

### hard links / exFAT

v2.11.3 は hard link を使わない。exFAT で config / workspace sync / repo 追加が動くようになった。
同時に、同一 project で 2 つの AI-DLC コマンドを同時起動した場合の lock 競合も明示的に扱う。

---

## 20.7 ファイル体系・監査・ガード系の増減

| 指標 | v2.10.0 | v2.11.3 | preview | origin/main |
|---|---:|---:|---:|---:|
| `docs/` | 103 | 107 | 107 | 107 |
| `core/tools/*.ts` | 76 | 113 | 115 | 120 |
| `core/hooks/*.ts` | 19 | 20 | 20 | 20 |
| `core/aidlc-common/protocols` | 9 | 17 | 17 | 17 |
| `VALID_EVENT_TYPES` | 105 | 131 | 131 | 131 |
| `audit-format.md` | 105 events / 25 categories | 131 / 27 | 131 / 27 | 131 / 27 |

不変:

- `core/aidlc-common/stages`: 33
- `core/scopes`: 11
- `core/agents`: 14
- `core/sensors`: 6
- harness 7 種: claude / codex / copilot / cursor / kiro / kiro-ide / opencode

`docs/reference/12-state-machine.md` の presentational grouping は v2.11.3 でも 20 categories を示す。
正準 registry の categories は 27 であり、**state-machine 表と canonical audit-format を混用しない**。

### 追加 event 26 件（v2.10.0 → v2.11.3）

`CONSTRUCTION_POLICY_SET`, `COORDINATION_STOOD_ASIDE`, `HOST_TURN`, `INTENT_CHECKOUTS_CHANGED`, `INTENT_REPOS_CHANGED`, `MAIN_CHECKOUT_CHANGE_ALLOWED`, `PLAN_APPROVAL_SKIPPED`, `QUESTION_ASKED`, `QUESTION_REPLIED`, `QUESTION_SHOWN`, `QUESTION_UNANSWERED`, `REQUEST_ROUTED`, `REVIEW_VERDICT_DICTATED`, `SCOPE_SAVED`, `SENSOR_COULD_NOT_RUN`, `SOURCE_WORK_NOT_FOUND`, `STAGE_DEFINITION_CHANGED`, `STAGE_OUTPUT_AMENDED`, `STAGE_RULES_BLOCKED`, `STATE_TRANSITION_BLOCKED`, `SUBAGENT_DISPATCHED`, `SUBAGENT_PROMPT_UNMATCHED`, `SYSTEM_INSTALL_ALLOWED`, `SYSTEM_INSTALL_BLOCKED`, `UNIT_SKIPPED`, `WORKSPACE_RECLASSIFIED`

削除 event は 0 件。

### protocol 追加

`stage-protocol-checkpoint.md`, `stage-protocol-collaborators.md`, `stage-protocol-loopback.md`, `stage-protocol-reviewer-repair.md`, `stage-protocol-reviewer-unit.md`, `stage-protocol-sensors.md`, `stage-protocol-summary.md`, `stage-protocol-wave.md`

### docs 追加

`docs/guide/facilitator-guide.md`, `docs/guide/onboarding.md`, `docs/guide/writing-inputs/technical-environment-guide.md`, `docs/guide/writing-inputs/vision-document-guide.md`

### hooks 追加

`core/hooks/system-install.ts`

---

## 20.8 リリース別要約

### 2.11.0（2026-10-08）

- leaner defaults: `enterprise` 以外で承認後の変更を 1 回だけ伝え、各 stage は lead agent のみ。
- `/aidlc --collaborators on` で support agents を戻す。
- `plan_approval` / `collaborators` frontmatter 導入。`express` / `poc` の Plan Approval off 出荷。
- Guard Policy の出荷既定が `enterprise` 以外 off。
- Plan Approval を engine が保持し、承認後の fingerprint 管理が明確化。
- bare `/aidlc` が進行中 work を拾う。question は人間の言葉で解釈される。
- copied project のファイル保全、config 中の open work 実行、update channel 制御など運用系の変更。

### 2.11.1（2026-10-20）

- Code Generation の review が worktree / nested checkout / outside-project code を拾えるようになった。
- Guard Policy off で chat compaction 後も build が続く。
- pause した agent を AI-DLC が勝手に continuation しない。
- plugin の manifest name / marketplace 名前、共通 plugin catalogue、native compose が大きく変わった。
- 2.8.0 以降の project が停止せず refresh できる。

### 2.11.2（2026-10-28）

- worktree・code plan・unit checkpoint・team owned Units・side-by-side builds の補強。
- `aidlc config --yes` が non-interactive で project refresh する方向へ。
- Bedrock を `.claude/settings.local.json` に分離。
- hook lock / protected file / many repos 系でスクリプト互換影響。

### 2.11.3（2026-11-04）

- Bun 1.4.4 native build。
- OpenCode 2 / opencode 1.18.29 対応。
- team Units end-to-end 対応。known gaps あり。
- 多くのハーネスで「人間の言葉・記号」が正確に残るようになった。
- Windows native `aidlc.exe`。system install gate。hard link 廃止。

---

## 20.9 公開ノートとして残す判断

- **v2.11.3 を現 Latest として優先。** ただし CI・plugin・Copilot・opencode・Windows native 導入では breaking changes を確認する。
- **チーム Unit を評価するなら v2.11.3 以降を前提にする。ただし known gaps がある。**
- **セキュリティ更新は必須**。v2.11.2 以前 native binary は Brotli CVE 影響下。
- **`origin/main` には v2.11.3 後の未リリース team Unit 修正が多数ある**。実装判断では preview tag や main SHA ではなく、stable tag と既知ギャップを分けて書く。
