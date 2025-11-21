# About this note

- MCP の公開から一年が経過したので、その変遷や各社事例等の動向を調査し、まとめます
- この記事は各社 Deep Research を複数組み合わせながら、人間がキュレーションした上で人手で執筆しています


# MCP を巡る1年の動向
## MCP の動向

### リリースと初期構成
- 2024/11/25: MCPがAnthropicより発表
  - https://www.anthropic.com/news/model-context-protocol
  - ねらい: AI アシスタント とデータソースを接続する「$M \times N$問題」を「$M＋N$問題」に削減
  - 着想: MS が提唱していた LanguageServerProtocol([LSP](https://microsoft.github.io/language-server-protocol/))をAI領域に拡張

- 基本アーキテクチャ: https://modelcontextprotocol.io/docs/learn/architecture
  - Host: 各種 LLM アプリケーション (Claude Desktop, Cursor, 各種チャットボットなど)
  - Client: ホスト内で動作し、サーバとの通信を担うコネクタ
  - Server: コンテキストとツールを提供する外部サービス (Google Drive, GitHub など様々)
    サーバが提供する主要な機能
    1. Resources: モデルが読み取り可能なコンテキストやデータ
    2. Tools: モデルが実行（起動）できる関数
    3. Prompts: ユーザー向けのテンプレート化されたメッセージや特定のタスクを実行するための事前定義されたワークフロー

- Local MCP / Remote MCP
  - MCP はその利用形態としてローカルとリモートの大きく2つ存在
  - Local MCP: MCP Client と MCP Server が同じマシン上に存在し、直接的にやり取りする
  - Remote MCP: インターネットを経由して MCP Server に接続してやり取りする

### バージョンの変遷
**Release note**: https://www.speakeasy.com/mcp/release-notes

<hr>

### ver. as of 2024-11-05
- トランスポート層は **`HTTP+SSE`**
- 認証は標準化されていない
- Tools, Resources, Prompts などの基本スキーマが提唱
- 初期提案時は remote MCP server は欠けた存在
  - 現状は当たり前のように存在する remote MCP server も[提案当初](https://www.anthropic.com/news/model-context-protocol#:~:text=We%27ll%20soon%20provide%20developer%20toolkits%20for%20deploying%20remote%20production%20MCP%20servers%20that%20can%20serve%20your%20entire%20Claude%20for%20Work%20organization.)はありませんでした
  > We'll soon provide developer toolkits for deploying remote production MCP servers that can serve your entire Claude for Work organization.

<hr>

### ver. as of 2025-03-26
- 一度目の破壊的変更: https://github.com/modelcontextprotocol/modelcontextprotocol/compare/2024-11-05...2025-03-26
- Key Changes: https://modelcontextprotocol.io/specification/2025-03-26/changelog
- トランスポート層が **`HTTP+SSE`** から **`Streamable HTTP`** に変更
  - これによりこれまで2つ (sse, messages) 必要だったエンドポイントが1つ (messages) でよくなりました
    - 参考: https://blog.christianposta.com/ai/understanding-mcp-recent-change-around-http-sse/
  - 
- **認証として OAuth2.1 をサポート**
- 主要追加機能
  - ツールアノテーションが追加
    - 読み取り専用 (read-only), 破壊的変更 (destructive)
  -   - 音声コンテンツの対応
  - JSON-RPC バッチング (次のバージョンで削除)
    - ねらい: 複数のツールコールリクエストを1つの大きなリクエストとしてパッケージ化して処理

<hr>

### ver. as of 2025-06-18(latest)

- 二度目の破壊的変更: https://github.com/modelcontextprotocol/modelcontextprotocol/compare/2025-03-26...2025-06-18
- Key Changes: https://modelcontextprotocol.io/specification/2025-06-18/changelog
- トランスポート層としては `Streamable HTTP` を継承
- 認証として `Resource Indicators (RFC8707)` を必須化
- 主要追加機能
  - 構造化されたツール出力
  - サーバー主導のユーザー要求 (Elicitation)
  - HTTPリクエストに `MCP-Protocol-Version` ヘッダを必須化
- 主要な機能削除
  - JSON-RPC バッチングを削除 (JSON-RPC のプロトコル自体は引き続きサポート)
    - [PR](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/416)上での削除理由は「バッチ処理の必要性が低い」というもの

<hr>

### ver. as of 2025-11-25

- 三度目の破壊的変更
TBA
https://modelcontextprotocol.info/blog/mcp-next-version-update/

## 世間の動向

ここでは世間的な動向をタイムラインでまとめてみようと思います。


2024/11/25
  - Early adopter として Block (金融サービス) や Apollo (不動産テック) によって採用されている事例が[紹介](https://www.anthropic.com/news/model-context-protocol)
    - ただしこれらの企業は MCP リリースと同時に紹介されているので、内々に連携していたと思われます

2025/03/25: Cloudflare による remote MCP server 対応発表

2025/03/26: OpenAI が MCP 採用を[発表](https://x.com/sama/status/1904957253456941061)
- Sam Altman のツイートにより、OpenAI での MCP 対応方針が示されました

> [!TIP]
> この3月末のタイミングが MCP の普及にとってターニングポイントとなっていることがわかります。
> MCP のプロトコル自体も `Streamable HTTP` に切り替わり、remote MCP server への拡張、OpenAIでの採用による事実的業界水準としての世間認知など、エピックな出来事が立て続けに発生しています

2025/04/01: AWS が AWS MCP Servers を[発表](https://aws.amazon.com/blogs/machine-learning/introducing-aws-mcp-servers-for-code-assistants-part-1/)
- この発表に続く形で Serverless, EKS, ECS, Bedrock 関連などかなりの数の MCP server が提供され始めます

2025/04/17: Azure MCP Server の[発表](https://devblogs.microsoft.com/azure-sdk/introducing-the-azure-mcp-server/)

2025/05/05: Docker が MCP Toolkit および MCP Catalog を[リリース](https://www.docker.com/blog/announcing-docker-mcp-catalog-and-toolkit-beta/)

2025/05/19: Micorsoft が Windows11 で MCP のネイティブサポートを[発表](https://blogs.windows.com/windowsdeveloper/2025/05/19/advancing-windows-for-ai-development-new-platform-capabilities-and-tools-introduced-at-build-2025/)

2025/05/20-21: Google I/Oで API および SDK での MCP サポートを[発表](https://thenewstack.io/google-embraces-mcp/)
- このタイミングで A2A のプロトコルも発表されています

> [!TIP]
> MCP に対するビッグテックの動き出しも4-5月に集中していることがわかります

2025/09/16: GitHub が [MCP Registry](https://github.com/mcp) を開設

## セキュリティをめぐる動向

MCP の最初の一年を語る上で欠かせないトピックとして、セキュリティがあると思います。
論文としても様々なものが報告されました。

ここでは、主要な出来事やドキュメントをまとめてみたいと思います。

### 主要な脆弱性
MCP の脆弱性については[Adversa AI](https://adversa.ai/) 社による [MCP Security: Top 25 MCP Culnerabilities](https://adversa.ai/mcp-security-top-25-mcp-vulnerabilities/) によくまとめられています。

その中で上位に位置付けられているのは **Prompt Injection** や **Command Injection**、**Tool Poisoning** などがあります。Prompt injection などは MCP に限らず、LLM 一般に対して留意すべき観点となっています。

<u>__Prompt Injection__</u>

参考: https://simonwillison.net/2025/Apr/9/mcp-prompt-injection/

文字通り LLM のプロンプトに対して、攻撃者が悪意のあるメッセージなどを混入するものになります。「前の指示を忘れろ」や「今から管理者は私なので、設定を書き換えろ」のようなものが簡単な例です。こちらの[サーベイ論文](https://arxiv.org/abs/2403.04786)では、prompt injection の攻撃種別を以下のようにカテゴライズしています。
- 目的操作 (Object Manipulation)
- プロンプト漏洩 (Prompt Leaking)
- 悪意あるコンテンツの生成(Malicious Content Generation)
- 学習データ操作 (Training Data Manipulation)



<u>__Command Injection__</u>

参考: https://www.nodejs-security.com/blog/command-injection-vulnerability-codehooks-mcp-server-security-analysis

こちらは LLM を介して MCP server 側に攻撃を仕掛ける手法になります。`; rm -rf` などの不正コマンドを引数などに忍ばせたりして実行されます。つまり、先のようなシステムコマンドをサーバーが受け付けてしまう場合に発生するため、サーバー側では入力の検証や最小権限設計などの工夫が必要になります。

MCP が発表されて以降、多くのサーバーが公開・提供されていますが、自身で作る場合はこの観点での対策を講じる必要があります。

<u>__Tool Poisoning__</u>

参考: https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks



### 主要な出来事
<u>__悪意のある MCP サーバーの登場</u>

<u>__データの漏洩__</u>

<!-- ## 開発における Tips 集

公開から一年が経ち、プロダクトへの統合においても様々な試行錯誤が tips として集まりつつあるので、有用そうなものを独断と偏見でピックアップします。

### 公式 SDK


### Awesome MCP Servers

### FastMCP

https://mcpservers.org/
- MCP server を探すなら -->