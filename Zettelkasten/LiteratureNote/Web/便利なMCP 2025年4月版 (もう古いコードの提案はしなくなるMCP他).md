---
title: 便利なMCP 2025年4月版 (もう古いコードの提案はしなくなるMCP他)
source: https://qiita.com/masakinihirota/items/8971aa8ccead3193e77f
author:
  - "[[masakinihirota]]"
published: 2025-04-07
created: 2025-05-22
description: 追記 2025年4月24日追加したいMCPSequential Thinking複雑な問題をシンプルにhttps://mcp.so/ja/server/sequentialthinking/…
tags:
  - web
  - AI
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

追記 2025年4月24日  
追加したいMCP

## Sequential Thinking

複雑な問題をシンプルに

オープンソースで誰でも無料で使えます。

## 主な機能

段階的な思考: 複雑な問題を小さなステップに分解して解決します。  
思考の進化: 理解が深まるにつれて、前の思考を修正したり、新たな視点を取り入れたりできます。  
思考の分岐: 複数の解決策の可能性を探り、比較検討できます。  
柔軟な調整: 思考のステップ数を動的に変更可能です。  
仮説検証: 解決策のアイデアを生み出し、検証するのに役立ちます。

## こんな時に便利

頭の中がゴチャゴチャしている問題を整理したい時  
何度も修正が必要な計画やデザインに取り組む時  
最初は何がなんだかわからない問題に立ち向かう時  
雑然としたアイデアから設計書を書く時

settings.json

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sequential-thinking"
      ]
    }
  }
}
```

追記終了

追記 2025年4月14日  
追加したいMCP

## Firecrawl

ファイアクロール MCP(有料、500クレジット分無料)  
[https://www.firecrawl.dev/](https://www.firecrawl.dev/)

ファイアクロール MCPは、AI（大規模言語モデル）がウェブサイトを効率的にクロールし、データを抽出するための強力なツールです。MCP（Model Context Protocol）サーバーを通じて、AIアシスタントがウェブから情報を収集・分析する能力を大幅に強化します。

## 主な機能

- スクレイピング: JavaScriptレンダリングによる高度なウェブスクレイピング
- クロール: 指定サイトを起点に複数ページを自動探索
- 検索: ウェブ上の情報をキーワードで検索
- 情報抽出: 特定の要素やパターンに基づく構造化データの抽出
- バッチ処理: 複数URLの同時処理
- ディープリサーチ: 複数の情報源を組み合わせた詳細な調査
- サブページもクロール可能

これで最新のドキュメントを常に用意できます。  
ファイアクロール MCPは、Cursorの `@Doc` 機能より遥かに強力で広範囲なウェブデータ収集が可能ですが、設定や利用にはより多くの手順とコストが必要です。

```json
{
  "mcpServers": {
    "firecrawl-mcp": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "YOUR_API_KEY_HERE"
      }
    }
  }
}
```

---

- Vercel MCP
- 永続的メモリ:  
	永続的メモリとは、AIエージェントが異なるセッションや実行にわたって情報を保持します。これは開発時の作業履歴などを終わりまで保存します。
- Mastra  
	AIエージェント
AIの登場時系列順(AI調べ)

GitHub Copilot  
GPT-3.5/ChatGPT  
GPT-4  
Cursor AI  
Google AI Studio  
Gemini 1.5  
GPT-4o  
Claude 3.5 Sonnet  
Gemini 2.0  
Claude 3.7  
Cline  
Windsurf  
Roo Code  
Gemini 2.5  
GitHub Copilot Agent mode  
Firebase Studio

追記終了

※立ち上げるごとにMCPの起動が必要です。  
`settings.json` などで、起動ボタンを押します。  
これはどうやら起動・接続確認のためのボタンで、実際に動かすのはGitHub Copilotに指示を出したときに接続できていれば使ってくれます。

## GitHub Copilotシリーズ

## VSCodeでのMCP設定

VSCodeの左下の歯車アイコンから「設定」を開きます。

設定の検索窓から `mcp` で検索します。

👇MCP関連の設定画面が表示されます。  
ユーザーもしくはワークスペースを選択します。

```text
次でも変更されています(ワークスペース)
モデル コンテキスト プロトコル サーバー構成
settings.jsonで編集
```

- ユーザーだとVSCode本体の `settings.json` が開きます。
- ワークスペースだとワークスペースのルートに `.vscode` フォルダが作成され、そこに `settings.json` が作成されます。  
	※ワークスペースはワークスペースファイル内(\[ワークスペース名\].code-workspace)でも設定できます。

VSCode設定の優先順位は ワークスペース＞ユーザー＞デフォルトです。

「 `settings.json` で編集」を開きます。

`.vscode\settings.json`

👆(ワークスペースの場合)設定ファイルが作成されます。

mcpのサンプルが表示されます。  
このサンプルはGitHub Copilotが日時を取得するためのサンプルです。

settings.json

```json
...
    "mcp": {
        "inputs": [],
        "servers": {
            "mcp-server-time": {
                "command": "[PythonのPath]",
                "args": ["-m", "mcp_server_time", "--local-timezone=Asia/Tokyo"],
                "env": {
                    "PATH": "${env:PATH}"
                }
            }
        }
    }
```

※この `mcp-server-time` のMCPはPythonをインストールしておく必要があります。

👇VSCodeでMCPを有効にするための設定を行います。

settings.json

```json
...
"chat.mcp.enabled": true
```

## MCPを最大限に活用した次世代のWebアプリ開発

- Next.js、Honoは **Context7 MCP** 最新のコードやスニペット
- デザインを v0やReaddy、same.new、Gemini(=AIデザイン生成Webサービス)で作成。
- フロントエンドを **Figma MCP** 画面デザイン等
- バックエンドを **Supabase MCP** DB操作等
- 資料を **filesystem MCP** 指定したフォルダに資料や1ファイル化したファイルの読み込み
- テストを **Playwright MCP** 自然言語でのテストコードの自動生成
- Git管理を **GitHub MCP** GitHubの自動操作等
- 保守運用を **Raygun MCP** でエラーの自動追跡
- サブスクライブを **Stripe MCP** Stripeでの実装
- 検索を **Brave Search MCP** 検索エンジンの実装
- 設計書を **sequential-thinking** アイデアをステップに分けた段階的な思考
- 多様なAIモデルを GitHub Copilot Agent mode、Claude Sonnet 3.5 3.7、GPT-4oでコード生成 追加が可能
- AI搭載エディタを VSCode、Cursor、Windsurf 他で利用できます。
- AI搭載エディタのエージェント化を AIの自律サポートを Cline、Roo codeで行います。

👆これで開発が出来ます。開発に役立つMCPは他にも数多く公表されています。

## AIデザイン生成Webサービス

自然言語でフロントエンドのデザインを生成するWebサービスです。

### Readdy

### same.new

### v0

### Gemini

最近はGeminiのCanvasモードから直接デザインを生成することができるようになりました。

デザイン専用のサイトではありませんが十分使えます。

詳細: MCPを最大限に活用した次世代のWebアプリ開発

Next.js、Honoなどのフレームワークと、様々なツールやサービスを組み合わせることで、開発効率を飛躍的に向上します。

## MCP (Model Context Protocol) とは

MCPは、AIモデルが外部の情報源（ドキュメント、コードなど）にアクセスし、それらを活用するための標準化されたプロトコルです。これにより、AIはより正確で最新の情報に基づいてコード生成や提案を行うことが可能になります。

## 開発環境

以下は、記事で紹介されている開発環境の概要です。

## エージェント化とは

AIの進化は、大きく以下の段階に分けられます。

1. **AIによるコード生成、会話**: 開発者の指示に基づいてコードを生成したり、会話を行う段階。Copilot（副操縦士）がこの段階に相当し、開発者をサポートします。
2. **エージェント化**: 👈️今ココ AIが開発者の代理として、 **「与えられた目標」** に対しより自律的にタスクを実行する段階。
3. **自律型AI**: AI自身が考え、判断し、行動する段階。(スカイネット)

**「エージェント化」とは、AIが単に指示を待つだけでなく、自らの判断で行動し、与えられた目標を達成するために必要なタスクを自律的に実行できる段階** を指します。従来のAIが「ツール」として開発者をサポートする役割だったのに対し、エージェント化されたAIは「共同作業者」あるいは「代理人」として、より主体的な役割を担います。

### エージェント化されたAIの特徴

- **自律性**: **「与えられた目標」** に対し、必要なタスクを自ら計画し、実行します。
- **適応性**: 状況の変化に応じて行動を調整し、予期せぬ事態にも対応できます。
- **推論能力**: 過去の経験や知識に基づいて、最適な行動を選択します。
- **持続性**: 長期的な目標に向けて、継続的に行動し、学習します。
- **インタラクション**: 外部環境や他のエージェントとインタラクティブに（相互作用）、協調して作業を進めることができます。

### エージェント化のメリット

- **生産性の向上**: 開発者はより高度なタスクに集中でき、AIエージェントがルーチンワークや複雑な作業を代行することで、開発プロセス全体の効率が向上します。
- **開発期間の短縮**: AIエージェントがコード生成、テスト、デプロイなどの作業を自動化することで、開発期間を大幅に短縮できます。
- **品質の向上**: AIエージェントがコードの品質をチェックしたり、テストを自動化することで、バグの少ない高品質なソフトウェアを開発できます。
- **24時間稼働**: AIエージェントは、人間の開発者のように休憩や休日を必要とせず、24時間体制で開発作業を続けることができます。

### 各ツールの詳細

#### 開発フレームワーク・ライブラリ

- **Next.js**: Reactをベースとしたフレームワークで、サーバーサイドレンダリング（SSR）や静的サイト生成（SSG）などの機能を提供し、高速でSEOに強いWebアプリケーションの開発を可能にします。
	- コンポーネントベースの開発: Reactのコンポーネントを組み合わせてUIを構築します。
	- ルーティング機能: ページ間の遷移を容易に実装できます。
	- APIルート: バックエンドのAPIをNext.js内で作成できます。
	- 豊富なエコシステム: 多くのライブラリやツールが利用可能です。
- **Hono**: 軽量で高速なWebフレームワークで、特にAPI開発に適しています。
	- 高速なパフォーマンス: 独自のHTTPエンジンにより、非常に高速な処理を実現します。
	- シンプルな設計: 学習コストが低く、簡単に使い始めることができます。
	- 豊富なミドルウェア: 様々な機能を提供するミドルウェアが用意されています。
	- TypeScriptファースト: TypeScriptでの開発を強力にサポートします。

#### AIを活用したデザインツール

- **v0**: AIを活用したデザイン生成Webサービスで、Next.js、Tailwind CSS、Supabaseなどの技術を組み合わせて、コンポーネントを自動生成します。
	- プロンプトによるデザイン生成: テキストによる指示（プロンプト）に基づいて、UIコンポーネントを生成します。
	- カスタマイズ可能: 生成されたコンポーネントは、Tailwind CSSを使って自由にカスタマイズできます。
	- コード生成: 生成されたコンポーネントのコード（React）を直接取得できます。
- **Readdy**: AIとのチャットを通じて画面デザインを生成するWebサービスです。ドラッグ&ドロップ操作は不要で、AIとの対話によってWebサイトの見た目を構築できます。HTML、Vue、Reactなどの形式でエクスポート可能です。
	- チャットベースのUIデザイン: テキストベースの指示でUIをデザインできます。
	- 多様な出力形式: 生成されたデザインをHTML、Vue、Reactなどの形式でエクスポートできます。
	- 迅速なプロトタイピング: 短時間でWebサイトのプロトタイプを作成できます。
- **Figma**: Webベースのデザインツールで、UIデザインやプロトタイピングに広く利用されています。
	- 共同作業: 複数人で同時にデザインを編集できます。
	- 豊富なプラグイン: 様々な機能拡張を提供するプラグインが利用可能です。
	- プロトタイピング機能: デザインした画面遷移をインタラクティブに確認できます。
	- デザインシステム: デザインの一貫性を保つための仕組みを構築できます。

#### バックエンドプラットフォーム

- **Supabase**: オープンソースのFirebase代替となるプラットフォームで、データベース、認証、ストレージなどの機能を提供します。
	- PostgreSQLデータベース: 高性能なデータベースを利用できます。
	- 認証機能: ユーザー認証機能を簡単に実装できます。
	- ストレージ機能: ファイルのアップロードやダウンロードを管理できます。
	- リアルタイム機能: データの変更をリアルタイムに取得できます。

#### MCP (Model Context Protocol)

- **filesystem MCP**: 指定したフォルダ内のドキュメントやファイルを読み込み、AIが利用できるようにするMCPです。
	- ローカルファイルアクセス: ローカル環境にあるファイルの内容をAIが理解し、利用できます。
	- ドキュメント解析: テキストファイル、PDFファイルなど、様々な形式のドキュメントを解析できます。
	- 情報検索: ファイルの内容に基づいて、関連する情報を検索できます。
- **Playwright MCP**: Playwrightは、E2Eテストを自動化するためのフレームワークです。Playwright MCPを使うことで、自然言語による指示からテストコードを自動生成できます。
	- E2Eテスト自動化: ユーザーの操作をシミュレートして、Webアプリケーションのテストを自動化します。
	- クロスブラウザ対応: 主要なブラウザ（Chrome、Firefox、Safariなど）でテストを実行できます。
	- 自然言語によるテスト記述: 日本語などの自然言語でテストのシナリオを記述できます。
	- テストコード生成: 自然言語の記述から、Playwrightのテストコードを自動生成します。
- **GitHub MCP**: GitHubとの連携を自動化するMCPです。
	- コード管理自動化: GitHubのリポジトリ作成、プッシュ、プルリクエストなどの操作を自動化できます。
	- CI/CD連携: GitHub Actionsと連携して、継続的インテグレーション/継続的デリバリー（CI/CD）のプロセスを自動化できます。
	- Issue管理: GitHubのIssueの作成、更新、クローズなどの操作を自動化できます。
- **Raygun MCP**: Raygunは、アプリケーションのエラーやパフォーマンスを監視するツールです。Raygun MCPを使うことで、エラーの自動追跡などが可能になります。
	- エラー追跡: アプリケーションで発生したエラーを自動的に検知し、詳細な情報を提供します。
	- パフォーマンストラッキング: アプリケーションのパフォーマンスを監視し、ボトルネックを特定します。
	- ユーザー行動分析: ユーザーの行動を分析し、問題の原因を特定します。
- **Stripe MCP**: Stripeは、オンライン決済プラットフォームです。Stripe MCPを使うことで、Stripeの機能をWebアプリケーションに組み込むことができます。
	- 決済処理: クレジットカード決済、定期購読決済など、様々な決済処理を実装できます。
	- 顧客管理: 顧客情報、支払い履歴などを管理できます。
	- サブスクリプション管理: 定期購読のプラン作成、管理などを行えます。

#### AIモデル

- **GitHub Copilot Agent mode、Claude Sonnet 3.5 3.7、GPT-4o**: コード生成に利用できる様々なAIモデルです。これらのモデルを組み合わせることで、より高度なコード生成が可能になります。
	- コード補完: 開発者がコードを入力する際に、次の候補を提示します。
	- コード生成: 自然言語による指示から、コードを自動生成します。
	- 多言語対応: JavaScript、Python、TypeScriptなど、様々なプログラミング言語に対応しています。
	- 文脈理解: コードの文脈を理解し、より適切な候補やコードを生成します。

#### AI搭載コードエディタ

- **VSCode、Cursor、Windsurf**: AI機能を搭載したコードエディタです。
	- AIによるコード補完: 高度なAIモデルによる、より正確なコード補完。
	- 自然言語によるコード生成: 日本語などの自然言語で指示を記述し、コードを生成。
	- コードレビュー支援: AIがコードの問題点を指摘し、改善案を提案。
	- デバッグ支援: AIがエラーの原因を特定し、修正方法を提案。

#### AIエディタのエージェント化ツール

- **Cline、Roo code**: AIエディタをエージェント化するためのツールです。AIがより自律的に開発をサポートできるようになります。
	- タスク自動実行: 開発者からの指示に基づいて、複数のタスクを自動的に実行します。
	- 自律的な問題解決: 開発中に発生した問題を、AIが自律的に解決しようと試みます。
	- 継続的な学習: 開発者の行動やコードを学習し、より高度な支援を提供できるようになります。

これらのツールや技術を組み合わせることで、AIが開発者の指示を待つだけでなく、自律的にタスクを実行し、開発プロセスを効率化することが可能になります。

---

## 便利なMCP紹介

## Context7

Context7 MCP Server

upstash/context7-mcp: Context7 MCP Server

GitHub Copilotを使っていると、時々 古い情報のコードが提案されることがあります。  
これは、カットオフ（＝ある特定期間までの情報）されているので最新情報に追いついていないからです。

その弱点を補うのがこの Context7です。  
Context7を使用することで、常に最新バージョンのドキュメントやコード例が提案されるようになります。  
そしてスニペットと呼ばれる、具体的なコード例も使用することが出来ます。

このスニペットとは、最新バージョンに対応しているコードでの動作確認がされているミニコードです。

Cursorの `@Doc` 機能と似ていますが、 `@Doc` はユーザーがドキュメントの場所を毎回指定する必要があります。  
Context7を設定すれば、必要なドキュメントへのアクセスが自動化され、常に最新の情報と動作確認済みのスニペットが得られます。

2025年4月現在、Context7は446のサイトに対応しています。

### 主な対応ドキュメント、スニペット

- Next.js
- Supabase
- Hono
- Elasticsearch
- Laravel
- Clerk
- MongoDB
- FastAPI  
	その他

### 主な特徴

- 最新のバージョン固有ドキュメントを自動で提供
- ソースコードから実際に動作するコード例
- 無駄な情報がなく、関連性の高い情報を提供
- VSCodeやCursor、ClaudeなどのAIツールへの簡単な統合
- MCP (Model Context Protocol) サーバーとの連携
- 個人利用は1日あたり最大 50 クエリまで無料

### 簡単な使い方

- VSCodeのsettings.json にある起動ボタンを押します。  
	(c7-mcp-serverコマンドを実行)
- GitHub Copilot Agent modeに自然言語で尋ねる。
- 自然言語の指示の後に `use context7` をつけます。
- 👇️GitHub Copilotの指示書(settings.json等)に設定しておくと自動で使うようになります。

settings.json

```json
...
    "github.copilot.chat.codeGeneration.instructions": [
        {
            "text": "MCPの context7 を使用できるのなら。指示にかならず \`use context7\` を追加してください。"
        },
```

※👆️この設定をするとMCPを使い分けたい時は不便です。  
設定をしておかないと毎回指示する必要が出てきますが。

### インストール

```terminal
npm install -g c7-mcp-server
```

settings.json

```json
...
    "context7": {
        "command": "c7-mcp-server", // インストールまたはリンクされたコマンド名
        "args": [] // デフォルトでは引数は不要（stdioトランスポート用）
        // "env": {} // 必要に応じて環境変数を追加
    }
```

### 動作テスト

```terminal
npx -y @modelcontextprotocol/inspector npx @upstash/context7-mcp@latest
```

### 最新コード

#### Next.js

Context7 - Up-to-date documentation for LLMs and AI code editors

#### TypeScript

Context7 - Up-to-date documentation for LLMs and AI code editors

#### TailwindCSS

Context7 - Up-to-date documentation for LLMs and AI code editors

#### Hono

Context7 - Up-to-date documentation for LLMs and AI code editors

#### Drizzle

Context7 - Up-to-date documentation for LLMs and AI code editors

#### Zod

Context7 - Up-to-date documentation for LLMs and AI code editors

## Supabase

Model context protocol (MCP) | Supabase Docs

supabase-community/supabase-mcp: Connect Supabase to your AI assistants

supabase-community/supabase-mcp: Connect Supabase to your AI assistants

MCPはつなげるだけの役割で、実際に動かすのはGitHub Copilotです。  
GitHub Copilotに指示してDBの情報を取得できないときがありますが、  
それは人間の指示が悪い場合が多いと思います。

ダイレクトにQueryを発行してデータを取ってきてくれと指示を出すと。  
GitHub Copilotはその通りに動いてくれます。

曖昧な指示、「データを取ってきて」とか曖昧だと素直に動かないと思います。

Supabaseへのアクセスが便利になるMCP

### 主な特徴

- Supabaseへのアクセスを効率化
- 最新ドキュメントやコードスニペットを提供
- 開発者が簡単にリアルタイムアプリケーションや認証機能を構築可能
- 複雑なSQLクエリを簡単な言葉で実行
- レポート作成や洞察の抽出が容易

### tools

SupabaseのMCPは24個のツールが用意されています。

これにより、開発者はSupabaseを使用してリアルタイムアプリケーションや認証機能を迅速に構築できます。

supabase-community/supabase-mcp: SupabaseをAIアシスタントに接続します

**プロジェクト管理**

**注意:** サーバーのスコープがプロジェクトに設定されている場合、プロジェクト管理のツールは使用できません。

- `list_projects`: ユーザーのすべての Supabase プロジェクトを一覧表示します。
- `get_project`: プロジェクトの詳細を取得します。
- `create_project`: 新しい Supabase プロジェクトを作成します。
- `pause_project`: プロジェクトを一時停止します。
- `restore_project`: プロジェクトを復元します。
- `list_organizations`: ユーザーが所属するすべての組織を一覧表示します。
- `get_organization`: 組織の詳細を取得します。

**データベース操作**

- `list_tables`: 指定されたスキーマ内のすべてのテーブルを一覧表示します。
- `list_extensions`: データベース内のすべての拡張機能を一覧表示します。
- `list_migrations`: データベース内のすべての移行を一覧表示します。
- `apply_migration`: データベースにSQLマイグレーションを適用します。このツールに渡されたSQLはデータベース内で追跡されるため、LLMはDDL操作（スキーマ変更）にこれを使用する必要があります。
- `execute_sql`: データベース内で生のSQLを実行します。LLMは、スキーマを変更しない通常のクエリにこれを使用する必要があります。
- `get_logs`: Supabaseプロジェクトのログをサービスタイプ（API、Postgres、エッジ関数、認証、ストレージ、リアルタイム）別に取得します。LLMはこれを使用して、サービスパフォーマンスのデバッグと監視に役立てることができます。

## エッジ機能管理

- `list_edge_functions`: Supabase プロジェクト内のすべての Edge 関数を一覧表示します。
- `deploy_edge_function`: Supabase プロジェクトに新しい Edge Function をデプロイします。LLM はこれを使用して、新しい関数をデプロイしたり、既存の関数を更新したりできます。

**プロジェクト構成**

- `get_project_url`: プロジェクトの API URL を取得します。
- `get_anon_key`: プロジェクトの匿名 API キーを取得します。

**ブランチング（実験的、有料プランが必要）**

- `create_branch`: 本番ブランチからの移行を含む開発ブランチを作成します。
- `list_branches`: すべての開発ブランチを一覧表示します。
- `delete_branch`: 開発ブランチを削除します。
- `merge_branch`: 開発ブランチから本番環境への移行とエッジ機能をマージします。
- `reset_branch`: 開発ブランチの移行を以前のバージョンにリセットします。
- `rebase_branch`: 移行のドリフトを処理するために開発ブランチを本番環境にリベースします。

**開発ツール**

- `generate_typescript_types`: データベーススキーマに基づいて TypeScript 型を生成します。LLM はこれをファイルに保存し、コード内で使用できます。

**費用確認**

- `get_cost`: 組織の新しいプロジェクトまたはブランチのコストを取得します。
- `confirm_cost`: ユーザーが新しいプロジェクトまたはブランチのコストを理解していることを確認します。これは、新しいプロジェクトまたはブランチを作成するために必要です。

### Supabase MCPの設定

SupabaseのMCP用のアクセストークンを取得します。

アクセストークン  
Supabase | Supabase  
[https://app.supabase.com/account/tokens](https://app.supabase.com/account/tokens)

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": [
        "-y",
        "@supabase/mcp-server-supabase@latest",
        "--access-token",
        "<personal-access-token>"
      ]
    }
  }
}
```

読み取り専用モードも可能  
`"--read-only"` をつけるだけ

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": [
        "-y",
        "@supabase/mcp-server-supabase@latest",
        "--access-token",
        "<personal-access-token>",
                "--read-only"
      ]
    }
  }
}
```

👇️Windows用の設定

```json
"supabase": {
                "command": "cmd",
                "args": [
                    "/c",
                    "npx",
                    "-y",
                    "@supabase/mcp-server-supabase@latest",
                    "--access-token",
                    "sbp_ad5345d770ade0e8691e62da099fa976453f5b5c"
                ]
            },
```

※デフォルト値はローカルのSupabase用に設定されています。  
サーバーのSupabaseプロジェクトでは、SUPABASE\_PROJECT\_REFとSUPABASE\_DB\_PASSWORDに値を指定する必要があります。

### 環境変数の設定

優先順位

1. 環境変数: 環境内で直接設定される値
2. ローカル.envファイル:.env現在の作業ディレクトリ内のファイル (ソースから実行している場合にのみ機能します)
3. グローバル設定ファイル:  
	ウィンドウズ:%APPDATA%\\supabase-mcp.env  
	macOS/Linux:~/.config/supabase-mcp/.env
4. デフォルト設定: ローカル開発のデフォルト (他の設定が見つからない場合) 👈今回の設定

#### サーバーのSupabase MCPの設定 (未設定)

This Supabase MCP Server Will 10X Your Productivity

サーバーのSupabaseプロジェクトでは次の形式を使用します。

`postgresql://postgres.[project_ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres`

\[region\]  
`ap-northeast-1 北東アジア（東京）`

`$env:APPDATA\supabase-mcp`  
Supabase MCPフォルダを作成します。

```terminal
echo $env:APPDATA

# Create config directory
# On macOS/Linux
mkdir -p ~/.config/supabase-mcp
# On Windows (PowerShell)
mkdir -Force "$env:APPDATA\supabase-mcp"

# Create and edit .env file
# On macOS/Linux
nano ~/.config/supabase-mcp/.env
# On Windows (PowerShell)
notepad "$env:APPDATA\supabase-mcp\.env"

QUERY_API_KEY=your-api-key
SUPABASE_PROJECT_REF=your-project-ref
SUPABASE_DB_PASSWORD=your-db-password
SUPABASE_REGION=us-east-1
SUPABASE_ACCESS_TOKEN=your-access-token
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
```

```terminal
pipx install supabase-mcp-server
```

👇️動作未確認 (開発、分析はローカルで行うため)

```text
{
  "mcpServers": {
    "supabase": {
      "command": "/full/path/to/supabase-mcp-server",  // Replace with the actual path from step 1
      "env": {
        "QUERY_API_KEY": "your-api-key",  // Required - get your API key at thequery.dev
        "SUPABASE_PROJECT_REF": "your-project-ref",
        "SUPABASE_DB_PASSWORD": "your-db-password",
        "SUPABASE_REGION": "us-east-1",  // optional, defaults to us-east-1
        "SUPABASE_ACCESS_TOKEN": "your-access-token",  // optional, for management API
        "SUPABASE_SERVICE_ROLE_KEY": "your-service-role-key"  // optional, for Auth Admin SDK
      }
    }
  }
}
```

※詳細はドキュメントを見てください。

## Postgres (LOCALのSupabase用)

Docker Desktopをインストールして、ローカルのSupabaseを立ち上げます。

SupabaseのMCPをローカルのSupabaseで動かすための設定ファイルを作成します。

alexander-zuev/supabase-mcp-server

👇️Dockerで動くローカルのSupabaseに接続するための設定です。

```json
"Postgres(LOCAL-supabase)": {
                "command": "cmd",
                "args": [
                    "/c",
                    "npx",
                    "-y",
                    "@modelcontextprotocol/server-postgres",
                    "postgresql://postgres:postgres@127.0.0.1:54322/postgres"
                ]
            }
```

### MCPを複数設定

jsonの構造を利用して複数のMCPを設定することができます。  
`settings.json` の適当な場所に挿入してください。

設定後 起動させてください。

settings.json

```json
...
    //////////////////////////////////////////////////////////
    // MCP
    //////////////////////////////////////////////////////////
    "mcp": {
        "inputs": [],
        "servers": {
            "mcp-server-time": {
                "command": "[pythonのPath]",
                "args": ["-m", "mcp_server_time", "--local-timezone=Asia/Tokyo"],
                "env": {
                    "PATH": "${env:PATH}"
                }
            },
            "context7": {
                "command": "c7-mcp-server", // インストールまたはリンクされたコマンド名
                "args": [] // デフォルトでは引数は不要（stdioトランスポート用）
                // "env": {} // 必要に応じて環境変数を追加
            }
        }
    }
```

※👆このローカルのSupabaseはDocker Desktop等で立ち上げておいてください。

SupabaseのMCPをローカルで動かすための設定ファイルを作成します。

## Playwright

Playwrightは、テストを自然言語で記述できるようにするMCPです。  
AIと連携することで、高品質なテストコードの生成をサポートします。

### 主な特徴

- 自然言語によるテスト記述
- ドキュメントからコードスニペットと例を自動抽出
- LLMを活用した説明とメタデータの強化
- セマンティック検索のためのベクトル化機能
- 関連性に基づくカスタムアルゴリズムでの結果スコアリング
- 高速なパフォーマンスのためのRedisキャッシュ

## Figma

Figmaの図をダイレクトにコード化等してくれます。

GLips/Figma-Context-MCP: MCP server to provide Figma layout information to AI coding agents like Cursor

GitHubサイト  
[https://github.com/GLips/Figma-Context-MCP](https://github.com/GLips/Figma-Context-MCP)

日本語ドキュメント  
[https://github.com/GLips/Figma-Context-MCP/blob/main/README.ja.md](https://github.com/GLips/Figma-Context-MCP/blob/main/README.ja.md)

公式動画 Creating a UI with Figma to Cursor MCP Server - YouTube

```terminal
# Figma-Context-MCPのインストール
git clone https://github.com/GLips/Figma-Context-MCP.git

cd Figma-Context-MCP

# 依存関係のインストール
pnpm install
```

Figmaのアクセストークンが必要です。

無料ユーザーでもアクセストークンは作れます。  
Figmaのサイトで、  
左上の自分のアイコンから  
設定＞セキュリティのタブ＞個人アクセストークン  
で「新規トークンを作成」ボタンを押します。  
許可の項目を自分で選び、(期限は無期限、わからなかったら全部下)FigmaのAPIキーを生成します。  
1回だけ表示されるので適当な場所にメモしておきます。

### 主な特徴

- Figmaデザイン情報の提供
- Figmaコメント操作
- React等のコードを自動生成
- デザインに関する質疑応答

### Figma MCPで出来ること

デザイン情報とAIの連携を円滑にし、開発効率を向上させます。  
Figmaのデザイン情報を構造化してAIに提供します。  
活用例はコード自動生成、デザインに関する質問応答、コメント操作等。  
デザインと開発者間のギャップを埋め、より効率的で精度の高い開発を行います。

- Figmaデザイン情報のAI提供

レイアウト情報: 要素の位置、サイズ、親子関係などをAIに構造的に提供。  
スタイル情報: 色、フォント、ボーダー、シャドウなどのデザイン属性をAIに提供。  
テキスト情報: 要素内のテキストコンテンツをAIに提供。

- AIによるデザインの理解と活用

コード生成: 提供されたデザイン情報を基に、HTML/CSS、Reactなどのコードを自動生成。  
質問応答: Figmaデザインに関する質問（例：「このボタンの色は？」）に対して、正確な情報を基に回答。  
デザイン分析: （将来的に）AIがデザインの構造やスタイルを分析し、改善提案などを行う可能性。

- Figmaとのインタラクション

コメント操作: Figma上のコメントを読み取ったり、AIがコメントを書き込んだりすることが可能です。

```json
...
{
    "mcpServers": {
        "Framelink Figma MCP": {
            "command": "cmd",
            "args": [
                "/c",
                "npx",
                "-y",
                "figma-developer-mcp",
                "--figma-api-key=FIGMA-API-KEY(figd_XXXXX)",
                "--stdio"
            ]
        }
    }
}
```

### 動作確認

デザイン系のMCPなので気軽な動作確認がしづらいです。  
ですので、MCP起動後、手動確認をしてみました。  
👇のコマンドを実行します。

```terminal
npx -y figma-developer-mcp --figma-api-key="FIGMA-API-KEY" --stdio
```

```terminal
{"method":"notifications/message","params":{"level":"info","data":["Server connected and ready to process requests"]},"jsonrpc":"2.0"}
```

👆接続に成功したら、このようなメッセージが出ました。  
準備が出来ているようです。

GitHub Copilot Agent mode(エージェントに切り替えます)で使えるようになりました。

Figmaのディスクトップ用アプリをダウンロードして、そのアプリでデザインをして、そのデザインをGitHub Copilotに読み込ませるようにしました。  
簡単に動きますね。

Figmaのディスクトップ用アプリ ダウンロード

### 簡単な使い方

- Figmaのディスクトップ用アプリでの作業

Figmaのディスクトップ用アプリでデザインします。  
デザインをグループ化します。  
右クリックから  
コピー/貼り付けオプション＞＞選択範囲へのリンクをコピー  
これをGitHub Copilotに渡します。

- GitHub Copilotでの作業

(VSCodeはsettings.jsonから)MCPを起動します。  
GitHub Copilotをエージェントモードに変更します。  
リンクを貼り付けます。  
GitHub Copilotと適当な会話をします。

```chat会話
https://www.figma.com/design/***** ＜＜＜このデザインを取り込めますか？コンポーネントとしてcomponentsフォルダを作ってそこにおいてください。
```

色々と適切な許可を出します。  
コードを生成してくれました。

## Stripe

Stripe - MCP Server | Cursor Directory

CursorでStripe MCPを使ってサブスクリプションの実装をやらせてみる

## GitHub

### 主な特徴

- GitHubワークフローとプロセスを自動化します。
- プルリクエストやイシューの追跡が自動化します。
- GitHubリポジトリからデータを抽出して分析します。
- GitHub のエコシステムとやり取りする AI 搭載のツールとアプリケーションを構築します。

github/github-mcp-server: GitHub's official MCP Server  
[https://github.com/github/github-mcp-server](https://github.com/github/github-mcp-server)

Introduction - Model Context Protocol

コンテナ内でサーバーを実行するには、Dockerをインストールする必要があります。

GitHub のアクセストークンが必要です。  
アクセストークンを作成するには、GitHubの設定ページにアクセスし、\[Developer settings\] > \[Personal access tokens\] > \[Tokens (classic)\]を選択します。  
次に、\[Generate new token (classic)\]をクリックし、必要なスコープを選択してトークンを生成します。

Create a GitHub Personal Access Token.

👆認証必須

### GitHub MCP 1個目 (公式)

Dockerを利用できる場合

github/github-mcp-server: GitHub's official MCP Server

GitHubのアクセストークンが必要

mcp.json

```json
...
"github": {
    "command": "docker",
    "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "GITHUB_PERSONAL_ACCESS_TOKEN",
        "ghcr.io/github/github-mcp-server"
    ],
    "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<github_token>"
    }
},
```

👆️に取得したアクセストークンを設定します。

### GitHub MCP 2個目

Dockerを使わない場合

GitHubが発表した新技術「github-mcp-server」がなぜ便利なのか、従来との比較と使い方を徹底解説｜D × MirAI

GitHub - MCP Server | Cursor Directory

GitHubのアクセストークンが必要

```text
"github": {
    "command": "npx",
    "args": [
        "-y",
        "@modelcontextprotocol/server-github"
    ],
    "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<github_token>"
    }
},
```

👆️に取得したアクセストークンを設定します。

## Raygun

### 主な特徴

- エラー追跡・分析
- リアルタイムモニタリング
- ユーザーが気づく前にクラッシュを検出
- 実用的な洞察が可能
- アプリケーションの安定稼働をサポート

MindscapeHQ/mcp-server-raygun

mcp.json

```json
...
    "mcpServers": {
        "raygun": {
            "command": "npx",
            "args": ["-y", "@raygun.io/mcp-server-raygun"],
            "env": {
                "RAYGUN_PAT_TOKEN": "your-pat-token-here"
            }
        }
    }
```

## Brave Search

GitHub Copilotがネット検索をしてくれます

modelcontextprotocol/servers: Model Context Protocol Servers

👆この🌟 Reference Servers に登録されているので比較的安全だと思われるサイトです。

🌟 Reference Servers  
Anthropic自身が実装したMCPサーバー

Anthropic はMCPを開発した会社であり、MCPの開発に関与しています。

### 前提条件

- 一応無料
- Stripe経由でカード情報と住所が必要
- サブスクライブ0$でのカード支払い登録が必要

### 出来ること

サブスクライブ0$で出来ること

- 画像検索
- ビデオ検索
- ニュース検索
- 提案
- スペルチェック  
	他

### 導入

アカウント作成します。  
サブスクライブに入会します。  
Stripe経由でカードの情報を登録します  
👆自己責任で！  
メールでやり取りをしてアカウントを作成します。

サイトからAPIキーを生成します。  
適当な場所に保存しておきます。

settings.json

```json
...
    "brave-search": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-brave-search"],
        "env": {
            "BRAVE_API_KEY": "[BRAVE_API_KEY]"
        }
    }
```

### インストール

```terminal
npx -y @modelcontextprotocol/server-brave-search

Error: BRAVE_API_KEY environment variable is required
is required
```

👆と出てきますが、このErrorは無視してよいようです。  
起動ボタンを押して起動を確認します。  
起動できれば、実行中と出ます。

### 使用例

- 設定ファイル(`settings.json`)で、Brave-searchを起動させます。
- GitHub Copilotをエージェントモードにします。

※GitHub Copilotの「質問する」、「編集」 モードでは検索しません。

エージェントモードに切り替えて指示をします。

例

```text
Brave Searchで Brave Search について検索し、その概要を \`Brave Search-summary.md\` というファイル名で保存してください。
```

このように👆具体的に指示することが大切です。  
使用するMCPと目的と、出力先等の詳細な情報をはっきり書いてください。

「\*\*\*\*\*について教えてください」と指示が曖昧ではGitHub Copilotは使ったり使わなかったりします。  
GitHub Copilotは空気を読みません、行間を読みません。  
前後の文脈から連想して回答しているだけです。

青い「続行」ボタンが表示されるので、押します。

続行ボタンのドロップダウンメニューで「常に許可」を選ぶと、毎回確認しなくなります。

実行の許可が求められるので  
自分の責任で良いと思えれば  
実行してもらいます。

青い「保持」ボタンを押します。

出力されたファイルを確認して、検索データが取れていればOKです。

使い終わったら有料、回数制限があるので、 `settings.json` で停止させておきます。

## filesystem

指定した範囲のローカルファイルにアクセスが出来るようになります。

modelcontextprotocol/servers: Model Context Protocol Servers

### インストール

```terminal
npm install -g @modelcontextprotocol/server-filesystem
```

### 設定例

settings.json

```json
...
    "file-system": {
        "command": "npx",
        "args": [
            "-y",
            "@modelcontextprotocol/server-filesystem",
            "[アクセスしたいローカルファイルのフォルダ、パス]"
        ]
    }
```

```text
[アクセスしたいローカルファイルのフォルダ、パス]

    👇このように置き換えます。

"C:\\2025_src"
```

必要な資料を入れたフォルダを指定したりします。  
複数のプロジェクトのパスが設定できます。

### 起動

起動ボタンを押して、エラーが出なければ成功です。

---

## MCP 関連URL

## 探す

Smithery - Model Context Protocol Registry

VeyraX

MCP Servers for Cursor

modelcontextprotocol/servers: Model Context Protocol Servers

🌟 Reference Servers  
Anthropic自身が実装したMCPサーバー

🤝 Third-Party Servers  
Anthropic以外が実装したMCPサーバー

## VSCodeのMCP Issues

Issues · microsoft/vscode

## MCP

Introduction - Model Context Protocol

---

## Gemini API キー

VSCode のGitHub CopilotでGemini系を使うのに登録が必要になりました。

無料で簡単に取得できます。

Geminiを使用するには、GeminiのAPIキーが必要です。  
キーは Google AI Studio で数回のクリックで作成できます。

Gemini API キーを取得する | Google AI for Developers

Get API key | Google AI Studio

2025年4月10日現在  
gemini 2.5 pro exp 3-25  
gemini 2.0 flash  
などが使えます。

---

## GitHub Copilotに最新の情報を取得させ、認識させる方法

GitHub Copilotは、使用する基盤モデルによって提案内容が左右されます。  
最新モデルであっても、常に最新のライブラリやフレームワークに対応しているとは限りません。  
そのため、GitHub Copilotに最新情報を認識させるには、以下の方法があります。

1. 人力
2. 1ファイル化してGitHub Copilotに認識させる
3. RAGに最新の情報を取得させ、認識させる
4. MCPを使用する
5. エージェントモードでLLMが自分で調べる

※下に行くほど人間が楽になります。  
※5のエージェントモードはGitHub Copilot Agent modeのことではありません。  
LLMが自律行動した場合のことを指します。

### 1 人力 (とてもたいへん)

- ドキュメントを参照する: 使用しているライブラリやフレームワークの公式ドキュメントを確認し、最新の情報を把握します。  
	最新のバージョンの情報を追いかけます。  
	学習して使い方を覚えて組み込めるようにします。
- GitHubリポジトリを確認する: 使用しているライブラリやフレームワークのGitHubリポジトリを確認し、最新のコードや変更履歴を把握します。

AIが流行る前の一般常識でした。

AIが流行った後は、ある程度最新の情報は仕方ないと自分で調べて1の方法を採用していました。  
数カ月後にはいろいろな手法が生み出され、👇2. 以降の技術が出てきました。

### 2 1ファイル化してGitHub Copilotに認識させる

- 関連する情報を1つのファイルにまとめ、GitHub Copilotにそのファイルを読み込ませて、コードを提案するように指示します（チャットやプロンプト、指示書を使用）。
- GitHubリポジトリを1ファイル化するツール
- uithub GitHubのサイトのリポジトリを1ファイル化
- repomix CLIのコマンド一つで1ファイル化

その他多数

- MCPクライアントアプリ、AI搭載IDE

VSCode  
Cursor  
Windsurf

1ファイル化したものをMCPクライアントアプリに読み込ませることで、GitHub Copilotが最新の情報を認識しやすくなります。

### 3 RAGに最新の情報を取得させ、認識させる方法

- RAG（Retrieval-Augmented Generation）を使用して、最新の情報を取得し、LLMに認識させる方法です。
- RAGは、LLMが情報を生成する際に、外部の情報源から取得した情報を組み合わせて生成する手法です。
- 共通の規格など無く、各開発者が自由に開発していました。

### 4 MCPを使用する

- 共通の規格が採用され、GitHub Copilotに取り込むのが非常に楽になりました。それがMCPと呼ばれるAIコネクト共通規格です。これがUSBコネクターと言われる所以です。
- Context7 ＜＜今回のBlogで紹介したMCP

事前準備された最新バージョンの情報等を用意しておき、LLMが自律的に最新の情報、コード、スニペットを収集できるようにします。

### 5 エージェントモードでLLMに自律的に調べさせる

- エージェントモードを搭載したエディタなどを使用し、LLMが自律的に情報を収集し、それに基づいてコードを生成する環境が実現しつつあります。

※このエージェントモードはGitHub Copilot Agent modeとは別です。

## 最後に

👆 5に関して、  
現在のところ、エージェントという言葉のとおり代理人になるのが限界です。  
自律タイプのAIはまだ出てきていません。

AIには、弱いAIと強いAIという分類があります。

- 弱いAI 人からの指示に基づいて動作するAI
- 強いAI 自律的に行動し、自分で情報を収集や判断できるAI

もし強いAIが現れれば、6. 自律調査ができる。が可能になれば、技術的特異点（シンギュラリティ）が起こったと言えるかもしれません。  
擬似的に自律的に動いているように見せることは出来ます。

[0](https://qiita.com/masakinihirota/items/#comments)

[169](https://qiita.com/masakinihirota/items/8971aa8ccead3193e77f/likers)

168