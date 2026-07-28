# 09. 参照リンクとローカルパス

## 9.1 公式

| リソース | URL |
|----------|-----|
| リポジトリ | https://github.com/awslabs/aidlc-workflows |
| v2 ブランチ | https://github.com/awslabs/aidlc-workflows/tree/v2 |
| Workflows 2.0 Spec PDF（正: `assets/`） | https://github.com/awslabs/aidlc-workflows/blob/v2/assets/AI-DLC-Workflows-2.0-Specification.pdf |
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
| `core/` | 手書き正本 |
| `dist/` | 生成配布物 |
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
docs/guide/harnesses/*.md
```

## 9.4 本ノートの更新のしかた

```bash
git clone --depth 1 --branch v2 https://github.com/awslabs/aidlc-workflows.git
cd aidlc-workflows
# バージョン確認
rg 'AIDLC_VERSION' core/tools/aidlc-version.ts
```

上流の変更に合わせて、本ノート側の数値・差分表を再同期すること。
