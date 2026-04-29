# SAM 3: Segment Anything with Concepts
source: https://scontent-nrt1-1.xx.fbcdn.net/v/t39.2365-6/585895112_1502482260871702_2839727966936571770_n.pdf?_nc_cat=108&ccb=1-7&_nc_sid=3c67a6&_nc_ohc=NSwfCWFsWo8Q7kNvwHyg4QX&_nc_oc=AdpCIYC5aEqLwjx47FlAXWiYJN-9cr5i04OtN7FoaF0R3_JjDKbQd7LszSw0cgFaQ7U&_nc_zt=14&_nc_ht=scontent-nrt1-1.xx&_nc_gid=2rUhayCp9whRQ02v6fsT3Q&_nc_ss=7a389&oh=00_Af3NQG99VfU83l8OLAY7I9OOL6-_KBysPMkowSCo1kTZEw&oe=69EB966D

# Abst
- Segment Anything Model (SAM) 3 を提案: _Concept Prompts_ (e.g., "yellow school bus") もしくは例示オブジェクト画像、さらにはその両方に基づいて、動画像内のオブジェクトを検出・セグメント・追跡できる統合モデル
- Promptable Concept Segmentation (PCS) では上述のようなプロンプトを入力として受け取り、インスタンスセグメンテーションマスクを返す
  - PCS の発展のため、4M 個のユニークなコンセプトラベルを使って高品質なデータを生成するデータエンジンを作成（画像、動画の両方が対象で、ハードネガティブ事例も含める）
- SAM3 はバックボーンを共有する画像レベルの検出器、メモリーベースの動画トラッカーを構成要素として持つ
  - Recognition と localization は presence head の予測を利用することで分離されており、検出精度の向上を狙う
    - Recognition (認識): 何が映っているかを当てるタスクで、いわゆる画像分類に近い。出力はラベル（カテゴリ）
    - Localization (位置特定): 対象物がどこに映っているかを当てるタスク。出力は座標・領域

# 1 Intro
- SAM シリーズにおいて Promptable Visual Segmentation (PVS) というアプローチが導入された
  - PVS では、点やbbox, マスクなど(ある種のビジュアル指示)をオブジェクトに与えるというやり方で、性能面はある種のブレイクスルーになった
  - しかし、画像中に現れる全てのインスタンスを検出するのが苦手だったりなどの制約があることが確認された

- これを改善するため、SAM3 では Promptable Concept Segmentation (PCS) を導入し、タスクとして定義
  - PCS task: テキスト and/or 画像 exampler を入力として受け取り、全てのオブジェクトがその概念に合致するかどうかを予測するタスク（動画の場合はオブジェクトアイデンティティも一貫させながら）のタスクになる
  - アトミックな視覚概念への認識に集中すべく、SAM3 側ではシンプルなテキストプロンプト (赤いリンゴ、横縞の猫とか)に限定している: ただ、MLLM を組み合わせることでより複雑なテキストプロンプトや推論が必要なケースにも拡張しうることを確認

![pvs-vs-pcs](materials/pvs_pcs.png)

![sam3-example](materials/sam3_examples.png)


- SAM3 の構成は以下
  - vision encoder を共有する detector と tracker
    - detecor: DETR-based のモデルで、テキスト、ジオメトリ、例示画像によって条件付けが可能
    - tracker: SAM2 の transformer encoder-decoder arch. を継承し、動画のセグメンテーションとインタラクティブなリファインメントを実現
  - 物体の存在判定を行う presence head
    - これによって recognition と localization のタスクを分割可能にする
    - ハードネガティブでも学習を行い、オープンエンドな語彙に対応する

- 学習データの構築には、human/model-in-the-loop なデータエンジンを構築し、大規模で広範な学習データをアノテーション
- 従来と比較して大きく3つの改善を実行
  1. media curation: より巨大で広範なメディアドメインをキュレーション
  2. label curation: オントロジーやMLLMをあのテータとして利用することでラベルの多様性や難易度を向上
  3. label verification: fine-tuned MLLM を利用して、人間と同水準で品質チェックを自動化

- Segment Anything with Concepts (SA-Co) を PCS 用のベンチマークとして作成
- 実験の結果、SAM3 は zero-shot mask AP で大幅な性能向上を確認
- また、アブレーションによって、バックボーンの選択、presence headの追加、ハードネガティブを利用した学習のそれぞれが性能向上に寄与していることを確認


# 2 Promptable Concept Segmentation (PCS)
- PCS タスクを次のように定義: 画像もしくは30秒以下の短尺動画を入力として、テキストプロンプト/例示画像/もしくはその組み合わせによって指定される視覚概念に合致する全てのインスタンスを検出・セグメンテーション・追跡するタスク
- 視覚概念はシンプルな名詞句に限定
- 名詞句は全てのフレームに対してグローバルに条件付けられるが、例示画像はフレーム個別にローカルに与えることができ、ターゲットマスクを繰り返しリファイン可能
![pcs-task](materials/pcs_task.png)

- 全てのプロンプトはそのカテゴリ定義に対して一貫させる必要がある; そうでない場合、モデルの挙動は未定義になる
- タスクの語彙構成には多義性・主観的な記述・文脈依存性・物体境界の曖昧さが含まれるため、本質的にこのタスクは曖昧性を孕む
  - このような曖昧性は先行研究 (LVIS) にも含まれるが、対象語彙をキュレーションし、関心対象となる全てのクラスに名医確な定義を設定することで緩和した
