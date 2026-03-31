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

## 1. Why naive implementations fall short
- これまでの取り組みから、ハーネス設計は長時間の自律コーディングに本質的に効いてくるものだとわかっていた。先の実験では以下を試した：
  - `Initializer agent`: product spec をタスクリストに変換する
  - `Coding agent`: セッションを跨がず、一度に1つの機能を実装する
  - この構成は開発者コミュニティでも試されていて、["Ralph Wiggum"](https://ghuntley.com/ralph/) メソッドとして hooks やスクリプトを使ってイテレーションサイクルを回すように作られるものもあった
- ただこの方法だと、複雑なタスクはエージェントがレールから外れやすいなどの欠点があった。問題を分解していく中で、このような失敗には2つの共通した要素があることを発見した
  1. コンテキストウィンドウがいっぱいになるにつれ、エージェントは一貫性を失いやすい
     - 一部のモデルでは Context anxiety (コンテキスト上限に近づいているとモデルが感じたタイミングで、終わるべきでないタスクを早めに締めようとする問題) などが見られた: コンテキストを完全にクリアし、新しいエージェントを起動した上で前のエージェントの状態および次のステップを含む構造化されたハンドオフの組み合わせからなる context reset は上記の問題を改善できる
     - Context reset は compaction とは異なる: Compaction は同じエージェントが短い要約を使って処理を続けられるよう、実行履歴の前半部分を要約し続けるもの。連続性を担保できるが「まっさらな状態」をエージェントに与えられないため、context anxiety は残り得る。
       - Claude Sonnet 4.5 ではこの context anxiety が強く表れることが観察されていたので、**compaction だけでは長時間タスクで十分な性能を引き出せず、context reset がハーネス設計において不可欠になった**: ただその分オーケストレーションの複雑さ、トークンのオーバーヘッド、各ハーネス実行あたりのレイテンシを増加させることになる。
  2. 高すぎる自己評価
     - 人間には平凡に見える成果物も、エージェントは往々にして自身の成果物を高く自己評価する傾向があった
     - この傾向はテスタブルでない、デザインのような主観タスクで特に顕著だった
     - ただ、テスタブルなタスクであっても、上記のような歪んだ傾向は生じうるため、「作業をするエージェント」と「それを評価するエージェント」を分離することが重要になる
       - `Evaluator` も LLM であるため、本質的な解決ではないが「自分の仕事に厳しくさせる」よりも「懐疑的にレビューさせる」方がまだコントロールしやすいため、問題の緩和に繋げられる構成となる
       - また外部評価としてのフィードバックループが存在することで、作業者である `generator` もそれを足場にして具体的な改善サイクルを回しやすくなる

## 2. Frontend desing: making subjective quality gradable

## 3. Scalling to full-stack coding

## 4. Running the harness

## 5. Iterating on the harness

## 6. Removing the sprint construct

## 7. Results from the updated harness

## 8. What comes next