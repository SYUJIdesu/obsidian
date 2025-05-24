---
title: AWS MCP Serversが登場したよ
source: https://zenn.dev/shirochan/articles/72827d09fdd0df
author:
  - "[[Zenn]]"
published: 2025-04-09
created: 2025-05-15
description: 
tags:
  - web
  - AI
---
11

3

## はじめに

みなさんMCPを使ってますか？  
AWSが、AWS MCP Serversをリリースしました。  
このツールを使えば、開発者は Claude、ChatGPT、Geminiなどのお馴染みのAIアシスタントを通じて、AWSの最新ドキュメントへのアクセス、ベストプラクティスの参照、コスト分析情報の取得などが可能になります。MCPサーバーは、AIアシスタントがAWSの様々なリソースやサービスに関する情報を取得し、より適切な支援を提供するためのブリッジとして機能します。  
本記事では、MCP（Model Context Protocol）の基本から、各サーバーの機能、セットアップ方法など、AWS MCP Serversの全体像をわかりやすく解説します。

## MCPとは？

MCP（Model Context Protocol）は、Anthropic社が主導するオープンソースプロジェクトで、AI言語モデルとツールを連携させるためのプロトコルです。MCPを利用することで、AIアシスタントは様々なツールやシステムと接続し、より高度なタスクを実行することが可能になります。

詳細はこちらをご覧ください

## AWS MCP Serversの概要

AWS MCP Serversは、AIコードアシスタント（Claude、ChatGPT、Geminiなど）がAWSのサービスやリソースに関する情報にアクセスするための特殊なサーバー群です。これにより、開発者はAIアシスタントを通じてAWSのドキュメント検索、ベストプラクティスの取得、コスト分析情報の参照などができるようになります。

このサーバー群は、開発者のローカル環境で動作し、AWSのベストプラクティスをAIアシスタントの回答に直接組み込むことができます。例えば、開発者がAIアシスタントに「S3バケットのセキュリティベストプラクティスは？」と質問すると、AWS Documentation MCP Serverが最新のドキュメントから関連情報を取得し、AIアシスタントの回答に反映させることが可能です。

## 主なAWS MCP Servers

AWS MCP Serversには、以下のような機能を持ったMCPサーバーが含まれています  
言語対応が英語のみの場合もあるのでご利用の際はよくREADMEを読んでください。

#### Core MCP Server

- 他のMCPサーバーの管理と調整
- 自動インストール、設定、管理機能の提供
- AWS Solutions構築のためのプラン提供の支援

#### AWS Documentation MCP Server

- AWSドキュメントへのアクセスとベストプラクティスの提供
- ドキュメントの検索機能
- AWSドキュメントページのコンテンツ推奨機能

#### Bedrock Knowledge Base Retrieval MCP Server

- Amazon Bedrockナレッジベースからの情報検索機能の提供
- 特定のタグ付けされたナレッジベースへのアクセス

#### AWS CDK MCP Server

- AWS Cloud Development Kit（CDK）のベストプラクティスの提供
- インフラストラクチャ・アズ・コードのパターン提案
- CDK Nagによるセキュリティコンプライアンスの確保

#### AWS Cost Analysis MCP Server

- AWSサービスのコスト分析機能の提供
- コスト関連情報へのアクセス

#### AWS Nova Canvas MCP Server

- Amazon Nova Canvasを使用した画像生成機能の提供

#### AWS Diagram MCP Server

- Python diagramsパッケージDSLを使用してシームレスに図表を作成機能を提供

#### AWS Lambda MCP Server

- MCPクライアントとAWS Lambda関数の間のブリッジとして機能を提供
	- コード変更なしにAWS Lambda関数をMCPツールとして選択・実行可能
	- 内部アプリケーションやデータベースなどのプライベートリソースにパブリックネットワークアクセスを提供せずにアクセス可能

#### AWS Terraform MCP Server

- AWSのTerraformベストプラクティスを提供
	- セキュリティ重視の開発ワークフロー
	- AWSおよびAWSCCプロバイダーのドキュメント
	- Terraformワークフローの実行

## 技術的な仕組み

AWS MCP Serversは以下のような仕組みで動作します：

1. **Model Context Protocol (MCP)の活用** ：MCPは、AIモデルとツールを連携させるためのプロトコルで、AIアシスタントがAWSサービスにアクセスするための標準化されたインターフェースを提供します。
2. **ローカル実行** ：MCPサーバーは開発者のローカル環境で動作し、AIアシスタントとAWSサービスの間のブリッジとなります。
3. **認証とアクセス** ：AWS IAM認証情報を使用して、セキュアにAWSサービスにアクセスします。開発者のIAM権限に基づいて操作が制限されるため、セキュリティが確保されます。
4. **コンテキスト処理** ：AIアシスタントとの会話の文脈を理解し、適切なAWSリソースや操作を提案・実行します。

## インストールと使用方法

AWS MCP Serversをインストールして使用するには、githubのドキュメントに記載されている以下の手順に従います

1. **Astralから `uv` をインストール**  
	uvはPythonパッケージマネージャーで、MCPサーバーの依存関係管理に使用されます。
2. **`uv` を使用してPythonをインストール**
	```bash
	uv python install 3.10
	```
3. **必要なサービスにアクセスできるAWS認証情報を設定**  
	AWS CLIの設定または環境変数を通じて、適切なIAM権限を持つ認証情報を設定します。
4. **MCPクライアントの設定に追加**  
	使用するMCPクライアント（Cline, Claude Desktopなど）の設定ファイルにMCPサーバーを追加します。

example

```json
{
  "mcpServers": {
    "awslabs.cost-analysis-mcp-server": {
      "command": "uvx",
      "args": ["awslabs.cost-analysis-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR",
        "AWS_PROFILE": "your-aws-profile"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

設定ファイルは使用するAIアシスタントによって適切な場所に配置します。  
Claude Desktopを使用する場合は、 `claude_desktop_config.json` に記述します。  
具体的な要件と構成オプションについては、個々のMCPサーバーの README を参照してください。

## まとめ

AWS MCP Serversは、AIアシスタントがAWSの情報にアクセスするためのオープンソースツールです。ドキュメント検索、ベストプラクティス提供、コスト分析などの機能を通じて、開発者のAWS活用を支援します。  
将来的には、EC2やS3などのAWSリソースを直接操作できるMCPサーバーが登場すれば、さらに便利になるでしょう。  
AIアシスタントを通じてAWSサービスに関する情報にアクセスしたい方は、このツールを検討してみてください。

## 参考ソース

- [AWS公式ブログ: AWS MCP Serversの紹介](https://aws.amazon.com/jp/blogs/machine-learning/introducing-aws-mcp-servers-for-code-assistants-part-1/)
- [AWS MCP Servers公式ドキュメント](https://awslabs.github.io/mcp/)
- [GitHub: AWS MCP Servers](https://github.com/awslabs/mcp/)

11

3

### Discussion

![](https://static.zenn.studio/images/drawing/discussion.png)

ログインするとコメントできます