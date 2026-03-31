# Harness design for long-running application development を読む
- source: https://www.anthropic.com/engineering/harness-design-long-running-apps
- published: 2026/03/24
- Author: Prithvi Rajasekaran (Claudeのさらなる使い方を探求する [Labs](https://www.anthropic.com/news/introducing-anthropic-labs) というチームのメンバーっぽい)

**Harness design はエージェンティックコーディングのパフォーマンスのキーとなるもの。**
**この記事ではフロントエンドデザインおよび長時間の自律的ソフトウェアエンジニアリングにおいて、どのように Claude を押し上げたかを解説する。**

- 過去数ヶ月に渡り、互いに関係する2つの問題に取り組んでいた
  1. Claude に高品質なフロントエンドデザインを作らせること
  2. 人間の介入なしに完全なアプリケーションを構築させること
  - この取り組みは [frontend design skill](https://github.com/anthropics/claude-code/blob/main/plugins/frontend-design/skills/frontend-design/SKILL.md) と [long-running coding agent harness](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) という投稿にもあるように、以前から取り組まれていた。プロンプトエンジニアリングとハーネス設計でベースラインより大きく性能を伸ばせたが、いずれも最終的には限界に突き当たった

- 上記状況打破のため、2つの異なるドメイン (主観的な好みによって定義されるものと定量化可能な正確性およびユーザビリティ) に跨る新しいアプローチを試した
  - GAN にインスピレーションを受け、`generator` と `evaluator` からなるマルチエージェントアーキを考えた
  - 特に `evaluator` は具体的かつ段階評価可能な指標を持つことで "is this design good?" のような主観評価も可能になるように設計
  - 先行していた取り組みから、ハーネスの取り付けかたとして「ビルドを扱いやすいチャンクに分割すること」「構造化されたアーティファクト（成果物ファイル）を使って、セッションを跨いでコンテキストを引き継げるようにすること」を学んでいたので、これらも活用
    - したがって最終的には上記の2つのエージェントだけでなく、`planner` のエージェントも加えた3体の構造を取ることで、長時間・フルスタック・高品質な自律コーディングを実現した

