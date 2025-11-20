# About this note

- MCP の公開から一年が経過したので、その変遷や各社事例等の動向を調査し、まとめる


# MCPサーバーを巡る1年の変動
## MCPの動向

### リリースと初期構成
- 2024/11/25: MCPがAnthropicより発表
  - https://www.anthropic.com/news/model-context-protocol
  - ねらい: AI アシスタント とデータソースを接続する「$M \times N$問題」を「$M＋N$問題」に削減する
  - 着想: MS が提唱していた LanguageServerProtocol([LSP](https://microsoft.github.io/language-server-protocol/))をAI領域に拡張

- 基本アーキテクチャ: https://modelcontextprotocol.io/docs/learn/architecture
  - Host: 各種 LLM アプリケーション (Claude Desktop, Cursor, 各種チャットボットなど)
  - Client: ホスト内で動作し、サーバとの通信を担うコネクタ
  - Server: コンテキストとツールを提供する外部サービス (Google Drive, GitHub など様々)
    サーバが提供する主要な機能
    1. Resources: モデルが読み取り可能なコンテキストやデータ
    2. Tools: モデルが実行（起動）できる関数
    3. Prompts: ユーザー向けのテンプレート化されたメッセージや特定のタスクを実行するための事前定義されたワークフロー

### バージョンの変遷
**Release note**: https://www.speakeasy.com/mcp/release-notes

<hr>

<u>**ver. as of 2024-11-05**</u>
- トランスポート層は **HTTP+SSE**
- 認証は標準化されていない
- Tools, Resources, Prompts などの基本スキーマが提唱

<hr>

<u>**ver. as of 2025-03-26**</u>
- 一度目の更新: https://github.com/modelcontextprotocol/modelcontextprotocol/compare/2024-11-05...2025-03-26
- Key Changes: https://modelcontextprotocol.io/specification/2025-03-26/changelog
- トランスポート層が **HTTP+SSE** から **Streamable HTTP** に変更
- **認証として OAuth2.1 をサポート**
- 主要追加機能
  - ツールアノテーションが追加
    - 読み取り専用 (read-only), 破壊的変更 (destructive)
  -   - 音声コンテンツの対応
  - JSON-RPC バッチング (次のバージョンで削除)
    - ねらい: 複数のツールコールリクエストを1つの大きなリクエストとしてパッケージ化して処理

<hr>

<u>**ver. as of 2025-06-18(latest)**</u>

- 二度目の更新: https://github.com/modelcontextprotocol/modelcontextprotocol/compare/2025-03-26...2025-06-18
- Key Changes: https://modelcontextprotocol.io/specification/2025-06-18/changelog
- トランスポート層としては Streamable HTTP を継承
- 認証として Resource Indicators (RFC8707) を必須化
- 主要追加機能
  - 構造化されたツール出力
  - サーバー主導のユーザー要求 (Elicitation)
  - HTTPリクエストに `MCP-Protocol-Version` ヘッダを必須化
- 主要な機能削除
  - JSON-RPC バッチングを削除 (JSON-RPC のプロトコル自体は引き続きサポート)
    - [PR](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/416)上での削除理由は「バッチ処理の必要性が低い」というもの

<hr>

<u>**ver. as of 2025-11-25**</u>

- 三度目の更新
TBA
https://modelcontextprotocol.info/blog/mcp-next-version-update/

## 世間の動向

ここでは世間的な動向をまとめます。

<hr>

<u>**開発コミュニティにおける主要なプレイヤーの対応**</u>

2025/03/26: OpenAIがMCP採用を[発表](https://x.com/sama/status/1904957253456941061)

2025/05/05: Docker が MCP Toolkit および MCP Catalog を[リリース](https://www.docker.com/blog/announcing-docker-mcp-catalog-and-toolkit-beta/)

2025/05/20-21: GoogleI/Oで API および SDK でのサポートを[発表](https://thenewstack.io/google-embraces-mcp/)

2025/09/16: GitHub が [MCP Registry](https://github.com/mcp) が開設

<u>**セキュリティ等**</u>