---
title: Cursor MCPの設定方法・使い方(mac)  - 仕事で使えるAI｜中村修三(ShuzoN)
source: https://note.com/shuzon__/n/na2aafacf7324
author:
  - "[[note（ノート）]]"
published: 2025-02-15
created: 2025-05-15
description: Cursor の使い方を知る     CursorでもMCP対応が来ましたね👏  https://www.cursor.com/ja/changelog/-cursor-rules-better-codebase-understanding-new-tab-model  Cursor MCPの設定方法・使い方についてまとまっている記事を見かけなかったので書きます。初めて設定したのでめちゃくちゃハマりました。  まだ英語でも情報が少ないため、まとめておきます。  MCPとは  簡単に言うと、AIが外部データを取得・更新する際に、個別の実装に依存せず統一された方法でやりとりできるようにす
tags:
  - web
  - AI
---
![見出し画像](https://assets.st-note.com/production/uploads/images/174690077/rectangle_large_type_2_134f9ca66aa6dbe31943543b492ce34e.png?width=1200)

## Cursor MCPの設定方法・使い方(mac) - 仕事で使えるAI

[中村修三(ShuzoN)](https://note.com/shuzon__)

## Cursor の使い方を知る

---

CursorでもMCP対応が来ましたね👏

[https://www.cursor.com/ja/changelog/-cursor-rules-better-codebase-understanding-new-tab-model](https://www.cursor.com/ja/changelog/-cursor-rules-better-codebase-understanding-new-tab-model)

Cursor MCPの設定方法・使い方についてまとまっている記事を見かけなかったので書きます。初めて設定したのでめちゃくちゃハマりました。

まだ英語でも情報が少ないため、まとめておきます。

## MCPとは

簡単に言うと、AIが外部データを取得・更新する際に、個別の実装に依存せず統一された方法でやりとりできるようにするための「共通言語」のようなものです。これにより、さまざまなシステムやツールと容易に連携し、柔軟かつ安全に情報のやり取りが可能になります。

MCP（Model Context Protocol）とは、AIが外部のデータソース（ファイル、データベース、ウェブサービスなど）にアクセスするための、共通の通信ルールを定めたプロトコルです。

CursorもMCPをサポートしているので、これを使うことでCursorからコンピュータの外側にあるデータにアクセスしたり、検索することができるようになります。

## 今回扱うMCP

世の中に出回っているMCPはにリスト化されています。

今回接続を試してみたのは以下です。おすすめの3つを選んでいます。

- notion
- brave-search
- sequetial-thiking

## 設定方法

日本語や英語で調べても一発で設定できる方法がまとまった記事を見かけなかったので設定方法をまとめておきます。

## 前提

ちなみに \`nodejs\` を使っているので、\`nodejs\` がインストールされていることが前提です。入れ方はいくつかあるので調べてみてください。

いちばん簡単なのは \`brew install node\` です。

## Cursorの基本設定

### Cursorのinstall

今回利用するエディタ Cursor のinstall方法です。

- ここからインストール: [https://www.cursor.com/ja](https://www.cursor.com/ja)

### Cursor Proの人

このまま \`CursorのMCP設定\`に飛んで、作業を続けてください。  
Cursor Proの人は特に設定不要でLLMを利用できるため、ここでは設定が要りません。面倒が嫌いな人はお金を払って解決するのがおすすめです。

### Anthoropic Claude API の設定

お金をあまり払いたくない人は、ClaudeのAPIを使ってみましょう。  
今回はAnthoropicのLLM Claudeを利用します。ClaudeをCursorで使うためにはAPI Keyが必要です。

[https://console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys) に飛んでAPI Keyを取得します。

API Keyとは、コンピュータ間で通信を行うために使う鍵だと思ってください。LLMを利用したり、データにアクセスしたり、更新したりするときに使うものです。 **家の鍵を人に渡さないように、非常に大切なものなので絶対にWeb上に公開しないでください。** LLMのAPIを公開した場合、かなり高額の金額を請求される可能性があるため、必ずRate Limitを設定しておいてください($20/monthくらいがおすすめ。)

扱いが難しいものなので、お試しだとしてもCursor Proを使うことをおすすめします。

### CursorのMCP設定

- MCPサーバーを追加する
- menu bar > cursor > settings > Cursor Settings > Features > MCP > Add MCP Server

ここにいろんなMCP Serverを追加します。基本はここで作業していると思ってください。  

![画像](https://assets.st-note.com/img/1739580440-o3RecPfbLGEW2jvVgd80UQxC.png?width=1200)

![画像](https://assets.st-note.com/img/1739580443-SaByTpElXCPWHF9hf764MGOg.png?width=1200)

## Notion

ここからはNotionのMCP設定をしていきます。

### NotionのAPI Keyの取得

[https://www.notion.so/my-integrations](https://www.notion.so/my-integrations) に飛んでAPI Keyを取得します。

- 名前は \`MCP\` とか \`MCP-Notion\` など適当につけてください。
- Typeは \`Internal Integration\` を選択します。
- Capabilityは全て選択します。
	- 不要なら\`Comment\` は選択しなくても大丈夫です。
- Use Capabilityは \`No User Information\` を選択します。
	- これ以外はユーザのメアドなども読まれるので強い権限です
	- \`No User Information\`は取得不可なのでご安心ください

これでAPI Keyが取得できます。こちらも絶対にWeb上に公開しないでください。

### Notionの記事にアクセス権限を付ける

API Keyを取得したら、Notionの記事にアクセス権限を付けます。

- Cursorから取得したい記事のページにアクセスします。
- ページの右上にある \`⋯\` ボタンをクリックします。
- \`Connections\` > \`Search for connections\` > 先ほどつけたAPI Keyの名前を選択します。

これでAPI Keyを使って対象記事と配下にアクセスできるようになりました。

![画像](https://assets.st-note.com/img/1739580529-TY8iI4OBpJQgZxAf3e75P1bX.png?width=1200)

### Notion MCP Server のインストール

では、Notion MCPを設定していきましょう。

[https://note.com/noujoujin/n/nfb627bc17194](https://note.com/noujoujin/n/nfb627bc17194)

  

を参考にします。多分 \`TypeScriptがない\` と言われるので先に入れておきましょう。

まずは \`mcp-notion-server\` をおきたいディレクトリに移動します。

> cd /Users/your\_id/<どこかのディレクトリ>  
> git clone [https://github.com/suekou/mcp-notion-server.git](https://github.com/suekou/mcp-notion-server.git)  
> cd mcp-notion-server  
> cd mcp-notion-server/notion  
> npm install tsc  
> npm install typescript  
> npm run build  
> npm link

in Terminal

これで \`mcp-notion-server\` というコマンドが使えるようになります。

### Notion MCP Server を Cursor に設定

CursorのMCP設定に戻って、\`Add MCP Server\` をクリックします。

- 名前は \`MCP-Notion\` など適当につけてください。
- コマンドは \`mcp-notion-server\` とします。
- 環境変数は \`NOTION\_API\_TOKEN\` とします。値は先ほど取得したAPI Keyを設定します。
- typeは \`command\` とします。

Server URLは以下のように設定します。

> env NOTION\_API\_TOKEN=<your notion api token> node /Users/<Username>/work/mcp-notion-server/notion/build/index.js

これでNotion MCP Server が設定できました。緑のランプがついているのがわかると思います。※ no tool available という表示が出ている場合は、設定がうまく行っていないので、設定を見直してください。  

![画像](https://assets.st-note.com/img/1739580619-6MlqhBF32EgNVGsKoCZRWkaS.png?width=1200)

成功するとこんな感じ

## Brave Search

Brave searchはCursorをつかってWeb検索をするときに使うものです。

同じくAPI Keyを取得します。

- [https://brave.com/search/api/](https://brave.com/search/api/) にアクセス
- アカウントを使って free subscription に登録
- 登録後、 [https://api-dashboard.search.brave.com/app/keys](https://api-dashboard.search.brave.com/app/keys) からAPI Keyを取得

このAPIも絶対にWeb上に公開しないでください。

### Brave-Search MCP Server を Cursor に設定

CursorのMCP設定に戻って、\`Add MCP Server\` をクリックします。

- 名前は \`MCP-Brave-Search\` など適当につけてください。
- コマンドは \`brave-search-server\` とします。
- 環境変数は \`BRAVE\_API\_KEY\` とします。値は先ほど取得したAPI Keyを設定します。
- typeは \`command\` とします。

brave-searchは nodeに同梱の \`npx\` を使うため、特に設定は不要です。

Server URLは以下のように設定します。

> env BRAVE\_API\_KEY=<your brave search api key> npx -y @modelcontextprotocol/server-brave-search

これでBrave Search MCP Server が設定できました。緑のランプがついているのがわかると思います。  

![画像](https://assets.st-note.com/img/1739580650-qENTYWtRvJPknGXO9CKdj7IU.png)

## Sequential-Thinking

Sequential-Thinkingは ChatGPT o1のように垂直思考を行ってくれるMCPです。  
つまり、一つの質問に対して何度も自律的に質問を繰り返して、最終的に答えを出してくれます。

### Sequential-Thinking MCP Server を Cursor に設定

Sequential-ThinkingはCursorのMCP設定に戻って、\`Add MCP Server\` をクリックします。

- 名前は \`MCP-Sequential-Thinking\` など適当につけてください。
- typeは \`command\` とします。

Server URLは以下のように設定します。

> npx -y @modelcontextprotocol/server-sequential-thinking

これでSequential-Thinking MCP Server が設定できました。緑のランプがついているのがわかると思います。  

![画像](https://assets.st-note.com/img/1739580677-yXTIlprDk91BO2nVwzd4scN0.png)

## 実際に使ってみる

これで設定ができたら、実際に使ってみましょう。

[**Cursor – Model Context Protocol** *Connect external tools and data sources to Cursor using the M* *docs.cursor.com*](https://docs.cursor.com/context/model-context-protocol)

によると

> MCP tools may not work with all models. MCP tools are only available to the Agent in Composer.

とあるので、Cursor Composerを使って実際に使ってみましょう。

Cursor内でCmd+I を押してComposerを表示します。今回はComposerのモデルに\`Claude 3.5 Sonnet\` を選択します。

## notion

> 僕のnotionから最新5本持ってきて内容を教えてください。

のようにテキストを入力します。するとCalled MCP tool notion\_search と表示されます。  
accept を押すと、notionのデータが取得できます。

## brave-search

> brave mcpをつかって今日の東京の天気を教えて下さい

と聞くと calling MCP tool brave\_web\_search と表示されます。  
accept を押すと、brave searchのデータが取得できます。

## sequential-thinking

例えばNotionで日記を渡しているならば以下のように聞いてみましょう。

> 僕(ハンドルネーム)の性格について教えて、必要ならばnotionやbrave searchをmcp経由で使っていいよ。そしてこの人の人格について再帰的に考えてみてください

すると何度も自分で質問を繰り返して、最終的に答えを出してくれます。  

![画像](https://assets.st-note.com/img/1739580781-0rwvYDCFSJbyqQ8fs3z6EoOe.png?width=1200)

notionを読みbrave searchで検索して垂直思考をしている

## まとめ

MCPはCursorの使い道を大きく広げてくれます。  
非常に便利なので、ぜひ使ってみてください。

Cursor MCPの設定方法・使い方(mac) - 仕事で使えるAI｜中村修三(ShuzoN)