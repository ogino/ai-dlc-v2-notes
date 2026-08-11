# 01. 概要 — AI-DLC とは何か

## 1.1 背景：なぜ「新しい SDLC」が必要か

AWS が指摘する問題意識は次のとおりである。

1. **AI 補助開発（AI-assisted）**  
   ドキュメント生成・補完・テストなどタスク単位で AI を使う。既存の人間中心プロセスに AI を後付けする形になり、AI の能力を抑え、古い非効率も温存しがち。

2. **AI 自律開発（AI-autonomous）**  
   要件からアプリ全体を AI に任せる。速度と品質の両方で、期待ほど安定した結果が出にくい。

いずれも「北の星」である生産性・品質・開発者体験を十分に満たせない、というのが方法論の出発点である。

出典: [AI-Driven Development Life Cycle（AWS DevOps Blog）](https://aws.amazon.com/blogs/devops/ai-driven-development-life-cycle/)

---

## 1.2 方法論の定義

**AI-DLC（AI-Driven Development Life Cycle）** は、AI を「補助ツール」ではなく **中心的な協働者（central collaborator）** として位置づけるソフトウェア開発方法論である。

2 本の柱:

| 柱 | 内容 |
|----|------|
| **AI 実行 + 人間監督** | AI が詳細計画を作り、明確化を求め、実装する。**重要決定は人間**が持つ |
| **動的なチーム協働** | 定型作業を AI が担う一方、人間はリアルタイムの判断・創造・合意形成に集中する |

基本パターン（メンタルモデル）:

```
AI: 計画を作る
 → AI: 文脈を得るため明確化の質問をする
 → 人間: 検証・決定
 → AI: 承認後に実装
 → （各 SDLC 活動で高速に繰り返す）
```

---

## 1.3 方法論上の 3 フェーズ（原典）

方法論の原典ブログ／Method Paper では、次の **3 フェーズ**が語られる。

| フェーズ | 内容 |
|----------|------|
| **Inception** | 意図を要件・ストーリー・作業単位へ展開（Mob Elaboration） |
| **Construction** | アーキテクチャ・ドメインモデル・コード・テスト（Mob Construction） |
| **Operations** | IaC・デプロイ・運用（蓄積コンテキストを利用） |

用語の刷新例:

- Sprint → **Bolt**（時間単位が時間〜日）
- Epic → **Unit of Work**

---

## 1.4 実装「Workflows 2.0」の位置づけ

| 層 | 説明 |
|----|------|
| **方法論** | AWS が定義する AI-DLC（ブログ + Method Definition Paper） |
| **実装** | `awslabs/aidlc-workflows` の **v2 ブランチ**（GA 発表済み） |

2.0 実装は方法論を **「検証可能で自己修正するエンジニアリング・ワークフロー」**として具体化し、次を実現する。

- **5 フェーズ / 32 ステージ**（原典 3 フェーズを実装都合で細分化・拡張）
- **14 エージェント・ロスタ**
- **9 アダプティブ・スコープ** + Composer
- **決定論的エンジン + LLM 導体**
- **1 core → 多ハーネス**（Claude Code / Kiro IDE / Kiro CLI / Codex / opencode / **GitHub Copilot**）
- **承認ゲート・監査・学習ループ**（人が重要決定を持つ。ただし次の例外あり）

**ゲートの例外（実装の正確な姿）**:

| 種別 | 挙動 |
|------|------|
| Initialization（0.1–0.3） | 承認ゲートなし（決定論・自動） |
| Verification Gate（フェーズ境界） | トレース検査は **自動**。問題時に続行／戻るを人が判断 |
| 通常ステージの承認ゲート | **人が最終判断**（レビュアは READY/NOT-READY のみ。拒否権なし） |
| Construction autonomous | Walking skeleton 後の ladder で選択。残 Bolt のゲートを省略可。**失敗時は halt-and-ask** |

GA は v2 README の *Announcing 2.0 (GA)* に根拠がある。同所 NOTE どおり、継続改善されるため **既知良版の pin** と生成物レビューを推奨する。

README の表現:

> turning AI agents into verifiable, self-correcting engineering workflows

---

## 1.5 解決しようとする「現実の痛み」

| 痛み | 2.0 の打ち手 |
|------|----------------|
| プロンプト間のコンテキスト・ドリフト | 永続 state（`aidlc-state.md`）・監査・Space/Intent |
| 決定理由が残らない | アーティファクト + **82 種**イベントの監査証跡 |
| 依頼していない作業の勝手実行 | ステージ境界の承認ゲート・Human presence |
| 小規模 PoC と規制対応の両立 | スコープ × 深度 × テスト戦略で儀式量を調整 |
| ハーネスごとのルール分裂 | `core/` 単一正本 → `dist/<harness>/` 生成 |

---

## 1.6 哲学: Small Mob, Broad Agents

狭い専門家を大量に並べる（ウォーターフォール的ハンドオフ連鎖）のではなく、

- **広く能力のあるエージェント少数（11 ドメイン）**
- 人間の 3〜5 人モブが機能単位を端到端で担うイメージ

を採用する。エージェント境界が減るほど、情報損失と調整コストが減る、という設計思想。

---

## 1.7 成果として期待されるもの

AWS ブログが挙げる便益:

- **Velocity**: 要件・設計・コード・テストの生成加速
- **Innovation**: 定型作業削減による創造時間
- **Quality**: 組織標準の一貫適用、トレース可能性
- **Market Responsiveness**: 短いサイクルでの市場反応
- **Developer Experience**: ルーチンから問題解決へのシフト

実装側の現実的な注意:

- ゲート数・アーティファクト量はスコープにより桁違い（例: `poc` は 8 ステージ / 数ゲート、`feature` は 32 ステージ / 約 29 ゲート）
- 弱いモデルでは任意ステップ（レビュア・学習儀式）がスキップされ得る

---

## 1.8 用語の対応（方法論 vs 実装）

| 方法論（原典） | 2.0 実装 |
|----------------|----------|
| Inception / Construction / Operations | Initialization + Ideation + Inception + Construction + Operation |
| Bolt | Construction での Unit 単位の 3.1–3.5 一巡 |
| Unit of Work | ステージ 2.7 で分解される実装単位 |
| Mob Elaboration / Construction | `mode: mob` / `mode: subagent`（hub-and-spoke 形状）/ `mode: pipeline` などのトポロジ |
| 1.x 時代のルール／ステアリング配布（コミュニティ対比） | TypeScript エンジン + skills/agents/hooks のネイティブ実装（[08](./08-v1-vs-v2.md) は要一次確認） |

詳細な 1.x との差分は [08-v1-vs-v2.md](./08-v1-vs-v2.md)。
