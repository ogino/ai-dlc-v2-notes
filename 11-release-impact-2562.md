# 11. リリース差分 2.5.37 → 2.5.62

作成日: 2026-08-11
基準: `awslabs/aidlc-workflows` branch `v2` HEAD `2ce654d1`（2026-08-10）／実装バージョン **2.5.62**
前回: [10-release-impact-2537.md](./10-release-impact-2537.md)（2.5.11 → 2.5.37）

CHANGELOG の実エントリ **17 件**（2.5.38 / 39 / 40 / 41 / 42 / 43 / 44 / 45 / 53 / 54 / 55 / 56 / 57 / 58 / 59 / 60 / 62）。
変更ファイル **1,425 件**（新規 382・削除 196）。うち **259 件が新規ハーネス `dist/copilot`**。
削除 196 件は主に 2.5.57 のフック改名が全 dist に波及したもの（`git diff` は改名検出なしでは削除＋追加として数える）。

---

## 11.1 数値の変化

| 項目 | 2.5.37 | 2.5.62 | 測定方法 |
|---|---:|---:|---|
| フェーズ | 5 | 5 | `core/aidlc-common/stages/` の直下ディレクトリ |
| ステージ | 32 | 32 | 同配下の `*.md` |
| エージェント | 14 | 14 | `core/agents/` |
| スコープ | 9 | 9 | `core/scopes/` |
| センサー | 5 | 5 | `core/sensors/` |
| **監査イベント種別** | **74**（19 分類） | **82**（21 分類） | `core/knowledge/aidlc-shared/audit-format.md` のレジストリ表 |
| **TypeScript フック** | **14** | **17** | `core/hooks/*.ts` |
| フック登録数（`dist/claude`） | 14 | **18** | `settings.json` の `hooks` |
| **CLI tools** | **35** | **37** | `core/tools/*.ts`（`aidlc-*.ts` ＋ ディスパッチャ `aidlc.ts`） |
| **ハーネス** | **5** | **6** | `dist/` 配下（`plugins/` を除く） |

> **フック 17 本に対し登録が 18 なのは**、`aidlc-fold-usage` が PreToolUse と PostToolUse の
> **両方に登録されている**ため。ファイル数と登録数は一致しない。

> **監査イベントの数え方**: 前回ノートは「74 の独立集計は未実施（ドキュメント間の内部一貫性のみ確認）」と
> 限定を付けていた。今回はレジストリ表から一意なイベント名を抽出して実際に数え、
> **2.5.37 で 74、2.5.62 で 82**、いずれも見出しの自称値と一致することを確認した。

---

## 11.2 フック 7 本と intent コマンドの改名（2.5.57）

**旧名はエラーにならず no-op になる。** スクリプトや CI が旧名を呼んでいると、
**動いているつもりで何も実行されない**。

| 旧名 | 新名 |
|---|---|
| `aidlc-audit-logger` | `aidlc-write-audit-log` |
| `aidlc-sync-statusline` | `aidlc-sync-workflow-state` |
| `aidlc-mint-presence` | `aidlc-record-human-turn` |
| `aidlc-runtime-compile` | `aidlc-rebuild-stage-graph` |
| `aidlc-stop` | `aidlc-continue-workflow` |
| `aidlc-dispatch-rules` | `aidlc-deliver-stage-rules` |
| `aidlc-sensor-fire` | `aidlc-run-sensors` |

**2.5.37 時点の 14 本のうち、7 本が改名され、7 本は名前据え置き。**
別途 3 本が新規（`aidlc-review-freeze` = **2.5.39**、`aidlc-fold-usage` = **2.5.40**、`aidlc-plan-approval-guard` = **2.5.41**。各フックの初出コミットで確認）。
7 ＋ 7 ＋ 3 = **17**。7 本消えて 10 本増えたわけではない。

> 上流 CHANGELOG は「The other ten keep their names」と書いているが、これは **2.5.57 時点の 17 本**を
> 基準にした数（改名時にはすでに新規 3 本が入っていた）。**2.5.37 を基準に数えると据え置きは 7 本**である。
> 実測: `git ls-tree` で 2.5.37 の 14 本を取り出し、改名 7 本を差し引いて確認。

併せて変わるもの:

- **`aidlc-utility intent-birth` → `intent-create`**。ワークスペース形式も `aidlc intent birth` → `intent create`
- **ハーネスアダプタの target word も全面改名**（**フックのファイル名とは別系統のリストで、1 対 1 では対応しない**。
  `block` / `pretool-block` に対応するフックファイル名の改名は無く、逆に `audit-logger` / `sensor-fire` に
  対応する target word の改名は無い）（`stop`→`continue-workflow`、`mint`→`record-human-turn`、
  `block`→`enforce-approval-gate`、`pretool-block`→`guard-tool-call`、`state-sync`→`sync-workflow-state`、
  `runtime-compile`→`rebuild-stage-graph`、`dispatch-rules`→`deliver-stage-rules`）。
  **旧 target word は受け付けられず、no-op になる**（エラーにはならない）
- `/aidlc --doctor` のハートビート表示も新名に（`write-audit-log` など）

**アップグレード時の注意**（上流 CHANGELOG より）:

- `dist/<harness>/` の再コピーは**必須であって任意ではない**。古いコピーは古い配線を使い続ける
- **Kiro IDE** は旧登録を自動削除できないため、上流ガイドの retired-hook クリーンアップを先に実行する。
  放置すると**新旧のフックが同時に発火する**
- **Codex** はフック信頼ハッシュがコマンド文字列を含むため `bun scripts/package.ts codex trust` の再実行が要る

---

## 11.3 GitHub Copilot ハーネス（2.5.60）

> **VS Code 1.130 の裏取り（2026-08-11）**: **2.5.60 を記録した前回 PR のレビュー**で
> 「2026-08 時点では未リリースの版番号ではないか（`1.103` の転記ミスでは）」という指摘を受け、
> 当時は判断を保留していた。**確認したところ、指摘のほうが誤り**で、上流記載が正しい。
>
> - [`updates/v1_130`](https://code.visualstudio.com/updates/v1_130) — 見出し「Visual Studio Code 1.130」、
>   本文に `Release date: July 22, 2026`
> - `1.103` は 2025-07。**1 年以上前**にあたり、要件としては低すぎる
>
> **「月刊リリースなら 1 年で 27 版は速すぎる」という反証も出たため、隣接版で確認した**:
> [`updates/v1_125`](https://code.visualstudio.com/updates/v1_125) が `Release date: June 17, 2026`。
> **1.125 → 1.130 が約 5 週間**であり、VS Code は月刊ではない。
> 2.5.60 のリリース（2026-08-09）は 1.130 の **18 日後**で、新規要件として整合する。

CLI と VS Code agent mode を **1 つの dist** でカバーする。

| 項目 | 内容 |
|---|---|
| 必要バージョン | Copilot CLI ≥ 1.0.74 / **VS Code ≥ 1.130**（要件の出典は上流の `docs/guide/harnesses/copilot.md` と `dist/copilot/AGENTS.md` の 2 ファイル。1.130 が実在すること＝2026-07-22 リリースは VS Code 更新履歴で裏取り。上の注記を参照） |
| 配置 | `.github/skills/`（`/aidlc`・ステージ／スコープランナー）、`.github/agents/`（14 ペルソナを custom agent 化） |
| フック | `.github/hooks/aidlc.json` から AIDLC アダプタ経由。**PreToolUse deny がネイティブでブロックする** |
| 統制 | 静的な許可／拒否リストは持たない。フックで強制する |

**他ハーネスと質的に違う前提**:

- **folder trust が必須**。プロジェクトの絶対パスが `~/.copilot/config.json` の `trustedFolders` に
  無いとリポジトリフックは動かない。**未信頼＝全フック無効**
- ヘッドレス（`copilot -p`）では `GITHUB_COPILOT_PROMPT_MODE_REPO_HOOKS=1` が要る
- VS Code は `SessionEnd` を持たないため共有マニフェストから除外され、
  次回 `SessionStart` で前セッションを事後整合する

---

## 11.4 reviewer class（2.5.54）— レビュー待ちのコストダイヤル

上流が「人間ゲート付きステージのレビューが長い」問題を認識し、対処した。

CHANGELOG によれば、実機 A/B で adversarial な refute-fix-re-review ループが
**inception ステージあたり 12 分以上**をレビュー段取りだけに費やしていた。

> **上流 CHANGELOG の原文**: 「the shape behind reports of 30-minute Requirements Analysis waits」。
> **この「30 分待ち」は上流が受け取った報告として CHANGELOG に書かれているもの**であり、
> 本ノートの観測でも特定組織の事例でもない。

| 対象 | クラス |
|---|---|
| 散文 7 ステージ（intent-capture / rough-mockups / requirements-analysis / user-stories / refined-mockups / application-design / units-generation） | **advisory 既定**（1 パス。指摘は承認ゲートで逐語引用され、人間が仕分ける） |
| Construction 設計・実装 5 ステージ（functional-design / nfr-requirements / nfr-design / infrastructure-design / code-generation） | **adversarial 維持** |

新しい制御:

- ステージ frontmatter `review_class: adversarial | advisory`
- スコープ上限 `review_cap: adversarial | advisory | none`（`bugfix` / `poc` / `workshop` は advisory 上限）
- 実行時オーバーライド `/aidlc --review <adversarial|advisory|none>`
  （**低い方が勝つ**。`adversarial` は解除扱いで、クラスを引き上げることはできない）
- 反復上限が**エンジン強制**に（従来は散文上の指示のみ）
- 新イベント `REVIEW_CLASS_CHANGED`
- 自律 swarm（Bolt）内のレビューは**スコープ上限・実行時オーバーライドの対象外**
  （Bolt ではレビューがマージ前の唯一の検証のため）

> 「レビュー実時間が約半分、指摘品質の低下なし」は**上流の測定値**であり、本ノートでは再現していない。

---

## 11.5 トークン・コスト追跡（2.5.40 / 2.5.53）

- `aidlc-fold-usage.ts` が PreToolUse ＋ PostToolUse で Claude Code の `transcript_path` を読み、
  **前回 fold 以降に追記された差分バイトのみ**をトークン数に畳み込む
- ステージ別・ワークフロー別の使用量とコストを記録。statusline に `up/down/$` セグメント
- **メトリクス送出は opt-in で既定無効**。`AIDLC_METRICS_ENDPOINT` を設定したときだけ
  StatsD を HTTP 送信する。**どのハーネスの設定にもエンドポイントは同梱されていない**
- 2.5.53 でキルスイッチ `AIDLC_DISABLE_USAGE_TRACKING` を追加
- モデル単価は `tools/data/model-rates.json`。**未知のモデルはコストを捏造せず出さない**

---

## 11.6 その他

| リリース | 要点 |
|---|---|
| 2.5.38 | 回答レビューが**独立した必須の人間チェックポイント**に。プロンプト固有の receipt ＋ 質問ファイルのダイジェスト ＋ 確認後の書き込み証跡で裏付け（conductor が自分で「Looks correct」と書いても満たせない） |
| 2.5.39 | reviewer receipt の自己無効化ループを決定論的な review freeze で停止 |
| 2.5.41 | **code-generation の plan-before-generation を決定論的に強制**。現場でコードを先に生成し計画を後付けする挙動が報告されていた |
| 2.5.42 | clone 後に gitignore されている `aidlc/active-space` カーソルを復元 |
| 2.5.43 | Kiro IDE で書き込み失敗を「フック decay」と誤報告していたのを修正 |
| 2.5.44 | **成果物が会話言語に追従**（既定の英語固定をやめた）。`org.md` に `## Mandated` 4 本を追加 |
| 2.5.45 | プラグイン管理コマンドを全 `/aidlc` ハーネスで決定論的にディスパッチ |
| 2.5.55 | 人間権限境界の強化。公開監査 CLI は診断用のみとなり、`HUMAN_TURN` / `GATE_APPROVED` などの**権限を持つ receipt を書けなくなった**。自律モードへの切替は**新しい人間ターン**を要求 |
| 2.5.56 | `Construction Iteration: unit-major` で、Unit ごとに設計（3.1–3.4）→ 実装（3.5）を回す |
| 2.5.58 | `claim-sources` センサーが隣接記述のソースタグを落とさないように |
| 2.5.59 | Stop フックの会話カーブアウトに 2 つ目の根拠（`.aidlc-human-turn` / `.aidlc-engine-touch` の mtime）。transcript を出さないハーネス（Kiro IDE / Kiro CLI / opencode）で、純粋な会話ターンが no-progress ブロック扱いになるのを解消 |
| 2.5.62 | `intent-create` が work details 無しで**fail closed**。2 本目の無関係な intent は**fresh session** に渡し、前の intent の transcript を継承しない |

---

## 11.7 本ノートの限界

- **CHANGELOG と実ファイルの静的確認のみ**。2.5.62 を実際に動かしていない
- reviewer class の効果、`dist/copilot` の folder trust 挙動、PreToolUse deny の実効性は
  いずれも**上流の記述による**
- 監査イベント 82 は**レジストリ表の集計**であり、82 種すべてが実際に発火することは確認していない
  （上流は `tests/feature/t48-audit-event-emitters.sh` が MANDATORY 印のものを検証すると記載）
