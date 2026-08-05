# 10. 2.5.11 → 2.5.37 の差分と、ソース読解で分かったこと

対象: [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows) v2 ブランチ
調査日: 2026-08-05（HEAD = 2.5.37 / CHANGELOG 最新エントリ 2026-08-03）

本章は、01〜09 が対象とした **2.5.11** から **2.5.37** までの 12 リリースの差分と、その過程でソースを読んで確認した挙動をまとめる。行番号は 2.5.37 時点のもの。

> 本ノートの他章と同じく非公式の二次整理である。実装の正は常に上流のソースを参照すること。

---

## 10.1 中核メトリクスは変わっていない

まず結論として、01〜09 で示した主要な数値は 2.5.37 でも変化がない。実測で確認した。

| 項目 | 値 | 確認方法 |
|------|-----|----------|
| フェーズ / ステージ | 5 / 32 | `find core/aidlc-common/stages -name '*.md'` の件数 |
| ステージのフェーズ別内訳 | init 3 / ideation 7 / inception 8 / construction 7 / operation 7 | 同上 |
| 実行モード集計 | inline 28 / subagent 2 / pipeline 1 / mob 1 | 全ステージの `mode:` frontmatter を集計 |
| エージェント | 14 | `ls core/agents/*.md` の件数 |
| スコープ | 9 | `ls core/scopes/*.md` の件数 |
| センサー | 5 | `ls core/sensors/*.md` の件数 |
| 監査イベント種別 | 74 | `docs/guide/10-state-and-audit.md` |
| スコープ別ステージ数 | enterprise 32 / feature 32 / mvp 22 / poc 8 / bugfix 7 / refactor 8 / infra 13 / security-patch 10 / workshop 25 | 全ステージの `scopes:` を集計 |

変わったのは次の 2 つ。

| 項目 | 2.5.11 | 2.5.37 |
|------|--------|--------|
| hooks | 13 | **14**（2.5.33 で `aidlc-dispatch-rules.ts` 追加） |
| tools | 32 | **35**（2.5.36 で workspace 系 3 本追加。うち `aidlc-*.ts` が 34 本＋ディスパッチャ `aidlc.ts`） |

---

## 10.2 12 リリースの性格

2.5.12 / 17 / 25 / 26 / 30 / 31 / 32 / 33 / 34 / 35 / 36 / 37 の 12 本を分類すると、機能追加よりも**「チェックが黙って素通りする経路を塞ぐ」方向**に偏っている。

| 分類 | リリース |
|------|----------|
| 決定論の強化（モデル裁量の削減） | 2.5.25（ARS 演算）/ 2.5.33（ルール配信）/ 2.5.37（質問レンダリング契約） |
| サイレント失敗の解消 | 2.5.26（幽霊ディレクトリ）/ 2.5.31（stdout パース）/ 2.5.32（発見不能マニフェスト）/ 2.5.35（codekb 上書き） |
| 拡張サーフェスの拡大 | 2.5.30（既定 scope 解決）/ 2.5.34（`adds.scopes`）/ 2.5.36（`repos.json`） |
| ハーネス固有の修正 | 2.5.12（Kiro IDE hook payload）/ 2.5.17（Kiro 権限リスト） |

各リリースの要点は [08-v1-vs-v2.md](./08-v1-vs-v2.md) の 8.3 表を参照。

---

## 10.3 センサーは fail-open である

`core/tools/aidlc-sensor.ts` の冒頭コメントが判定分岐を列挙している。

```
a) SIGTERM かつ timeout 到達        → BUDGET_OVERRIDE
0) spawn 失敗                        → PASSED (spawn-failed)
b) exit 127（ツール不在）            → PASSED (tool-unavailable)
c) exit 0 かつ JSON.pass === false   → FAILED
d) exit 0 かつ JSON.pass === true    → PASSED
e) exit が 0/127 以外                → PASSED (script-error: exit-<n>)
f) 不正 JSON / pass 欠如             → PASSED (script-error: bad-output)
   既定                              → PASSED (script-error: unknown)
```

**FAILED になるのは c) の 1 経路のみ。** ツール未インストール、spawn 失敗、異常終了、不正 JSON はいずれも PASSED として記録される（`Note:` に理由が残る）。

2.5.31 は stdout 先頭のノイズ（パッケージマネージャのバナー等）を読み飛ばす修正で、実際の FAILED が `bad-output` として握りつぶされる頻度は下がった。ただし fail-open の構造自体は変わっていない。

**含意**: 監査証跡上の `SENSOR_PASSED` は「チェックが通った」ことを必ずしも意味しない。ガバナンス証跡として使う場合は `Note:` フィールドまで見る必要がある。

### マニフェストの注意点

- フィールド名は `severity` ではなく **`default_severity`**。受理値は `advisory` のみ（`core/tools/aidlc-sensor-schema.ts:177`）
- `blocking` は将来の "v0.10.0 ralph driver" 向けに予約（`docs/reference/07-sensor-system.md:383-384`、`docs/roadmap.md:101` の #431）
- `matches` 未設定のセンサーは**絶対に発火しない**（`core/hooks/aidlc-sensor-fire.ts:190` の `if (!entry.matches) continue;`）
- 命名規約（`aidlc-<id>.md`）から外れたマニフェストは、**Plugin 経由なら 2.5.32 以降 `--doctor` に degraded drop として出る**が、`sensors/` へ直接置いた場合は今も黙って無視される（`core/tools/aidlc-graph.ts:690-691` のコメントが *"Anything not matching the prefix is silently ignored"* と明言）

---

## 10.4 ステージ遷移と人間の関与

`aidlc-state.md` のカーソルが前進する経路と、それぞれで人間の関与が要求されるかを整理した。

| 経路 | human presence | エンジンの制約 |
|------|:---:|------|
| 承認 `report --result approved` | **要求する** | `HUMAN_TURN` の鮮度検査（`core/tools/aidlc-state.ts:2014-2035`） |
| スキップ `report --result skipped` | **要求しない** | CONDITIONAL のみ／`--reason` 必須／Current Stage 一致／状態が in-progress・revising・skipped（`core/tools/aidlc-orchestrate.ts:4242-4275`） |
| `aidlc-state.ts` の動詞を直接呼ぶ | — | PreToolUse フックがブロック（11 動詞。`core/hooks/aidlc-state-transition-guard.ts:13-25`） |
| `aidlc-jump.ts`（`--stage` / `--phase`） | 要求しない | 上記フックの対象外 |

### human presence の実装

`core/hooks/aidlc-mint-presence.ts` が UserPromptSubmit で `HUMAN_TURN` を監査台帳に追記し、承認処理がその鮮度を検査する。ソースのコメントが意図を明言している。

> so a model under autopilot cannot fabricate an approval with no human having acted this turn

`dist/<harness>/` の settings では、`PostToolUse: AskUserQuestion` にも同じフックが配線されている。つまり質問ツールへの回答でも presence が立つ。

**例外が 3 つある**。

1. Construction の autonomous モードは設計上 presence を要求しない（`aidlc-state.ts:2020`）
2. 監査台帳が空の場合は fail-open
3. 環境変数 `AIDLC_SKIP_HUMAN_PRESENCE_GUARD=1` でガード自体を無効化できる（`core/tools/aidlc-lib.ts:2910`）。テスト用スイッチだが**配布物にも同梱される**ため、導入時に未設定であることを確認しておくとよい

### スキップ経路にガードが無い意味

`execution:` の内訳は **ALWAYS 11 / CONDITIONAL 21**。CONDITIONAL の 21 ステージは、導体が `--reason` を付けて単独でスキップ報告できる。エンジンが課す 5 つの制約はいずれも「操作の整合性」を守るものであり、「判定の妥当性」を人間に確認する仕組みではない。

これは欠陥ではなく設計である。CONDITIONAL ステージの条件判定は本来 conductor の職務であり、AI-DLC の「AI が実行を主導し、人間が重要決定を持つ」という前提そのものにあたる。ただし**その判定品質はモデルの能力に直結する**。上流 README が弱いモデルでの利用に注意を促しているのは、この構造と整合する。

> なお README のその注記（*"On weaker models the conductor may skip optional stage steps..."*）は 3 箇所とも **Kiro（IDE / CLI）向けのセクション**にあり、有料 Kiro プラン要件を説明する文脈に置かれている。他ハーネスのセクションに同種の記載はない。

### reviewer 強制のカバレッジ

`reviewer:` を宣言したステージは、当該レビュアの新鮮な `REVIEW_COMPLETED` が無いと 4 つの完了経路すべてで拒否される（`verifyReviewerPrecondition()`、`core/tools/aidlc-state.ts:1287`。導入は 2.5.5）。

ただし宣言しているのは **32 中 12 ステージ**で、うち 8 が CONDITIONAL。**回避不能なのは ALWAYS の 4 つ**（intent-capture / requirements-analysis / units-generation / code-generation）である。

---

## 10.5 配布とアップグレードの仕組み

各リリースの CHANGELOG はほぼ例外なく *"**Upgrade:** re-copy your `dist/<harness>/` shell into the project"* と述べる。この配布モデルについて確認したこと。

- 配布物にはバージョンが焼き込まれ（`dist/<harness>/.../aidlc-version.ts`）、`/aidlc --version` と `--doctor --export` が自己申告する
- しかし**上流の最新版と比較する仕組みは無い**。`--doctor` にバージョン照合の行は無い
- `bun scripts/package.ts --check` は `core/` から `dist/` が正しく再生成されているかを見る**上流 CI 向けの drift guard** であり、`dist/` をコピーした利用者側のインストール健全性は検証しない
- **in-place upgrade サブコマンドは未実装**（`docs/roadmap.md:94` の #535）

つまり、コピーして使う側には「上流が進んだ」ことを知らせるシグナルが仕組みとして無い。**上流追従は運用で担保する**必要がある（誰が、どの頻度で `--version` と CHANGELOG を照合するか）。

### 既存インストールへの影響

| リリース | ディスク上の変化 | 判定 |
|---|---|---|
| 2.5.35 | codekb に `## Scope of Analysis` ブロック追加 | 再コピー自体は安全。既存ストアは `UNKNOWN_SCOPE` 扱いとなり、**再スキャンするまで機能低下**（狭いスキャンでの上書き警告が効かない） |
| 2.5.26 | 幽霊ディレクトリの削除 | 破壊的移行ではない。既存の空ディレクトリは任意の手動削除 |
| 2.5.36 | `repos.json` | opt-in。無ければ sync は休眠し doctor 行も出ない |
| 2.5.33 | `.aidlc-steering-token-key` の新規 mint | 既存の状態ファイルは破壊しない |

**状態スキーマの破壊的変更は見当たらない。**2.5.11 から 2.5.37 への一跳びは、状態ファイルの観点では安全と判断できる。

最も影響が大きいのは **`dist/` への手編集**である。上流は `dist/` を生成物として手編集を禁じており（`docs/reference/11-contributing.md:33,48`）、インストール先で加えた agent 定義や `settings.json` のパッチは再コピーで消える。永続的なカスタマイズは `memory/`（team.md / project.md）か Plugin 機構に置くのが文書上の正道。

---

## 10.6 2.5.33 のルール配信を運用で見るべき点

ステージルールの配信が「導体がパスを読む」から「エンジンが内容を配信」へ変わった（`load-steering` ディレクティブ）。解こうとした問題は *"closing the observed skip where stages ran with none of their org/phase memory applied"* と CHANGELOG が明言している。

一方で新しい運用上の観点が生まれた。

- ディレクティブは **28 KiB 上限**（`DIRECTIVE_MAX_BYTES`、`core/tools/aidlc-orchestrate.ts:712`）。超過分は見出し → UTF-8 境界で分割され、継続トークンで連結する
- 継続トークンは machine-local 鍵（`.aidlc-steering-token-key`）で署名される。鍵の破損・権限異常は continuation の失敗として現れる
- 連鎖が途中で切れると、ルールが不完全なままステージが進む可能性がある
- 巨大なルールセットは part 数の増加として現れる。監視対象は part 回数と `context_warnings`
- **Kiro CLI は tool input を書き換えられない**ため、oversized brief が advisory で進行しうる。他ハーネスと失敗条件が揃わない

対象は**ルール（`memory/*.md`）のみ**で、ペルソナと知識ファイルは引き続き path-loaded である（`docs/reference/03-orchestrator.md:365`）。

---

## 10.7 プラグイン機構の変化（2.5.34）

- `adds.scopes` が「宣言できるが無視される」状態から**実際にマージされる**ようになった。プラグイン独自の scope にコアの既存ステージを取り込める
- ガードレールは 2 つ。①対象 scope の `scopes/<name>.md` が既にインストール済みであること ②その `plugin:` frontmatter が寄与元プラグイン名と完全一致すること（名前の prefix からの推測は無効）
- ステージ番号は**エンジンが所有**する。プラグインが書く `number:` は同一フェーズ内の相対順序ヒントであり、絶対値ではない。依存順序は `requires_stage` が優先される
- `adds.requires_stage` は 2.5.37 でも **deferred**（宣言しても drop-log されマージされない）

参照実装は `plugins/test-pro/`。手順は `docs/harness-engineering/10-authoring-a-plugin.md`、内部仕様は `docs/reference/18-plugin-mechanism.md`。

---

## 10.8 確認していないこと

- Spec PDF の全文（他章と同じく未精読）
- Codex CLI / opencode 向け配布物の権限設定（Claude Code の `settings.json` と Kiro の agent config のみ確認）
- 監査イベント 74 種の独立集計（ドキュメント間の内部一貫性のみ確認）
- 各ハーネス固有 projection における挙動差の全件確認（`core/` の共通ロジックを中心に確認した）
