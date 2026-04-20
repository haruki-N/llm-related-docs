# SAM 3: Segment Anything with Concepts
source: https://scontent-nrt1-1.xx.fbcdn.net/v/t39.2365-6/585895112_1502482260871702_2839727966936571770_n.pdf?_nc_cat=108&ccb=1-7&_nc_sid=3c67a6&_nc_ohc=NSwfCWFsWo8Q7kNvwHyg4QX&_nc_oc=AdpCIYC5aEqLwjx47FlAXWiYJN-9cr5i04OtN7FoaF0R3_JjDKbQd7LszSw0cgFaQ7U&_nc_zt=14&_nc_ht=scontent-nrt1-1.xx&_nc_gid=2rUhayCp9whRQ02v6fsT3Q&_nc_ss=7a389&oh=00_Af3NQG99VfU83l8OLAY7I9OOL6-_KBysPMkowSCo1kTZEw&oe=69EB966D

# Abst
- Segment Anything Model (SAM) 3 を提案: _Concept Prompts_ (e.g., "yellow school bus") もしくは例示オブジェクト画像、さらにはその両方に基づいて、動画像内のオブジェクトを検出・セグメント・追跡できる統合モデル
- Promptable Concept Segmentation (PCS) では上述のようなプロンプトを入力として受け取り、インスタンスセグメンテーションマスクを返す
  - PCS の発展のため、4M 個のユニークなコンセプトラベルを使って高品質なデータを生成するデータエンジンを作成（画像、動画の両方が対象で、ハードネガティブ事例も含める）
- SAM3 はバックボーンを共有する画像レベルの検出器、メモリーベースの動画トラッカーを構成要素として持つ
  - Recognition と localization は presence head の予測を利用することで分離されており、検出精度の向上を狙う
