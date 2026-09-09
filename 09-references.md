# 09. 参照リンクと上流リポジトリ内パス

## 9.1 公式

| リソース | URL |
|----------|-----|
| リポジトリ | https://github.com/awslabs/aidlc-workflows |
| `main` ブランチ（2.x の正本） | https://github.com/awslabs/aidlc-workflows/tree/main |
| `v1` ブランチ（1.x） | https://github.com/awslabs/aidlc-workflows/tree/v1 |
| Workflows 2.0 Spec PDF（正: `assets/`） | https://github.com/awslabs/aidlc-workflows/blob/main/assets/AI-DLC-Workflows-2.0-Specification.pdf |
| Roadmap | https://awslabs.github.io/aidlc-workflows/roadmap.html |
| Method Blog | https://aws.amazon.com/blogs/devops/ai-driven-development-life-cycle/ |
| Open-sourcing adaptive workflows | https://aws.amazon.com/blogs/devops/open-sourcing-adaptive-workflows-for-ai-driven-development-life-cycle-ai-dlc/ |
| Building with Q Developer | https://aws.amazon.com/blogs/devops/building-with-ai-dlc-using-amazon-q-developer/ |
| Method Definition Paper | https://prod.d13rzhkk8cj2z0.amplifyapp.com/ |
| Responsible AI Policy | https://aws.amazon.com/ai/responsible-ai/policy/ |

## 9.2 コミュニティ・解説

| リソース | URL |
|----------|-----|
| AI-DLC Flow Overview (specs.md) | https://specs.md/aidlc/overview |
| ELEKS 解説 | https://eleks.com/blog/aws-ai-dlc-explained/ |
| Zenn: Rules deep dive | https://zenn.dev/aecomet/articles/ai-dlc-workflows-deep-dive |
| Builder.aws: v2 core concepts | https://builder.aws.com/content/3GnQs6zaYp4mAx9FWBulDkhWnBo/ai-dlc-v2-core-concepts |
| Discovery Tool | https://builder.aws.com/content/3Edi3vEnVpRCQNFWzbQJMtqn1Py/discovery-tool-for-ai-dlc-one-core-every-platform |
| AI-Native Builders | https://ai-nativebuilders.org/ |

## 9.3 公式リポジトリ内の主なパス

クローン先を `aidlc-workflows/` とした場合:

| パス | 内容 |
|------|------|
| `README.md` | 公式 README |
| `docs/guide/` | User Guide |
| `docs/harness-engineering/` | ハーネス拡張ガイド |
| `docs/reference/` | 開発者リファレンス |
| ~~`docs/rfcs/`~~ | **2.8.x で削除**（2026-09-08）。上流の**作業用メモ置き場で仕様書ではなかった**。現在は `.gitignore` 済みで、設計提案は GitHub issue の RFC テンプレートへ移った。当時の読み方は [12.9 節](./12-release-impact-2602.md) を参照 |
| `core/` | 手書き正本 |
| ~~`dist/`~~ | **2.8.x で削除**。`.gitignore` 済みのローカル生成物になった（→ [17.1](./17-release-impact-2801.md#171-いちばん大きい変更は-dist-の消滅)） |
| `CHANGELOG.md` | 2.x 変更履歴 |
| `assets/AI-DLC-Workflows-2.0-Specification.pdf` | 2.0 Specification（公式パス） |

### 公式ガイド索引

```
docs/guide/00-introduction.md
docs/guide/01-getting-started.md
docs/guide/02-your-first-workflow.md
docs/guide/03-spaces-and-intents.md
docs/guide/04-phases-and-stages.md
docs/guide/05-scopes-and-depth.md
docs/guide/06-agents.md
docs/guide/07-interaction-modes.md
docs/guide/08-knowledge.md
docs/guide/09-rules-and-the-learning-loop.md
docs/guide/10-state-and-audit.md
docs/guide/11-session-management.md
docs/guide/12-cli-commands.md
docs/guide/13-customization.md
docs/guide/14-artifacts-reference.md
docs/guide/15-troubleshooting.md
docs/guide/16-worked-examples.md
docs/guide/17-skills.md
docs/guide/glossary.md
docs/guide/workshop-mode.md
docs/guide/agents/README.md
docs/guide/agents/*.md          # ドメインエージェント別の深掘りページ
docs/guide/harnesses/*.md
```

## 9.4 本ノートの更新のしかた

**この節だけは意図的に `main`（動くブランチ）を見る。** 目的が「上流がどこまで進んだかを知ること」だからで、
タグに固定してしまうと差分の検知そのものができない。

```bash
git clone --depth 1 --branch main https://github.com/awslabs/aidlc-workflows.git
cd aidlc-workflows
# バージョン確認
rg 'AIDLC_VERSION' core/tools/aidlc-version.ts
```

上流の変更に合わせて、本ノート側の数値・差分表を再同期すること。

**本ノートが記述している版をそのまま再現したいときは、`main` ではなくタグを使う**
（**本ノートが対象とする 2.8.1 のソースを照合するなら SHA `c03f9e28` を checkout する** ——
`v2.8.0` は `0d399dd8` を指し、その後の 2 コミットを含まない。
**`v2.8.1` タグは存在しない**。リリース資産・公開済み版の確認なら `--branch v2.8.0`。
手順は [6.7](./06-harnesses-install.md#67-ソースの確認方法)）。
用途が違うので、この 2 つを取り違えないこと。
