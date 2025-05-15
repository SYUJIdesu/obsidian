---
title: "POST だけ？こんな馬鹿げた API デザインの議論を終わらせよう · Logto ブログ"
source: "https://blog.logto.io/ja/post-only-debate"
author:
  - "[[Logto ブログ]]"
published: 2024-07-15
created: 2025-05-15
description: "\"POST だけ\" API の神話を打ち破り、なぜそれが API デザイン原則の誤解に基づいているのかを説明し、RESTful と RPC のアーキテクチャスタイルの適切な使用事例を明確にします。"
tags:
  - "web"
  - "API"
---
ユーザー認証に何週間も費やすのはもうやめましょう

Logto でより速く安全なアプリをリリース。数分で認証を統合し、コア製品に集中できます。

[

始めましょう

](https://cloud.logto.io/?sign_up=true&utm_source=blog&utm_medium=subscribeUs&utm_campaign=post-only-debate)

![Product screenshot](https://blog.logto.io/assets/product-features-Bk9JAhsM.png)

最近、"POST だけ" を使って API を設計するべきかどうかの議論が私の注意を引きました。この議論を掘り下げてみると、人々が争っている問題が無意味であるばかりでなく、多くの開発者が API デザインの本質を誤解していることも明らかになりました。今日は、API デザインの核心的な考え方に深く迫り、この議論が最初から存在するべきではない理由を見ていきましょう。

## "POST だけ" の誤解

"POST だけ" を使って RESTful API 仕様を置き換えることを支持する開発者は、明らかに API デザインの最も重要な点を理解していません。彼らの主張は通常、以下のようなものです:

1. 設計の簡素化: 1 つのメソッドですべてを処理できる
2. セキュリティ: POST パラメータは URL に表示されない
3. 柔軟性: POST は任意のデータ構造を送信できる

一見すると、これらの主張には一理あるように見えます。しかし、実際には、これは HTTP メソッドの選択と API デザインスタイルの混同、すなわち異なるレベルの問題を混同しているに過ぎません。POST は HTTP プロトコルの 1 つのメソッドに過ぎませんが、REST は API デザインのスタイルです。

## API デザインの本質

具体的な API スタイルについて議論する前に、まず API デザインの核心的な目的が何であるかを理解する必要があります。優れた API は次のようであるべきです:

1. 明確で理解しやすい: 他の開発者（将来の自分自身も含む）が、各エンドポイントの目的を直感的に理解できる
2. 一貫性がある: 一定の仕様に従い、学習コストを削減する
3. 拡張性がある: バージョン管理や機能の拡張を簡単に行える
4. 効率的である: パフォーマンスやリソースの利用効率を考慮する

## RESTful API: HTTP メソッドの選択だけではない

RESTful API は多くの API デザインスタイルの中の 1 つであり、リソースとリソースに対する操作に焦点を当てています。簡単なブログシステムを例にして、RESTful APIがどのように設計されているかを見てみましょう:

1. すべての記事を取得:
	```
	GET /api/articles
	```
2. 特定の記事を取得:
	```
	GET /api/articles/{id}
	```
3. 新しい記事を作成:
	```
	POST /api/articles
	Content-Type: application/json
	{
	  "title": "REST vs RPC",
	  "content": "これは記事の内容です...",
	  "authorId": 12345
	}
	```
4. 記事を更新:
	```
	PUT /api/articles/{id}
	Content-Type: application/json
	{
	  "title": "更新されたタイトル",
	  "content": "更新された内容..."
	}
	```
5. 記事を削除:
	```
	DELETE /api/articles/{id}
	```

この例では次のことがわかります:

- API は "記事" リソースを基盤に設計されています
- 異なる操作を表すために異なる HTTP メソッドが使用されています
- URL 構造は操作されているリソースを示しており、明確です

この設計アプローチによって、API はより直感的で自己説明的になり、開発者が各エンドポイントの機能を理解しやすくなります。

## RPC: "POST だけ" の背後にある API スタイルを理解する

RPC (Remote Procedure Call) スタイルの API デザインの目的は、リモートサービス呼び出しをローカル機能の呼び出しのようにシンプルに見せることです。

興味深いことに、"POST だけ" を支持する人々は、実際には RPC スタイルを説明していることに気づいていないかもしれません。

RESTful API に比べて、RPC はリソースよりも操作そのものに重点を置いています。これが、RPC スタイルの API が通常 "動詞 + 名詞" 形式を使用する理由です。例えば `getProduct(productId)` や `createUser(userData)` です。

多くの RPC 実装では、すべての操作が通常同じエンドポイントに POST リクエストとして送信され、リクエストボディ内で特定の操作とパラメータが指定されます。これが "POST だけ" の考えが実際には RPC に近い理由です。

例えば、HTTP ベースの RPC スタイルでの "商品の取得" リクエストは次のようになります:

```
POST /api/rpc/getProduct

Content-Type: application/json

{

    "productId": 12345

}
```

現代の RPC フレームワーク（例えば gRPC）は、より強力で効率的な実装を提供しています。これを例にして RPC スタイルを説明しましょう:

まず、サービスとメッセージ形式を定義します（Protocol Buffers を使用）:

```
syntax = "proto3";

package blog;

service BlogService {

  rpc GetArticle (GetArticleRequest) returns (Article) {}

  rpc CreateArticle (CreateArticleRequest) returns (Article) {}

}

message GetArticleRequest {

  int32 article_id = 1;

}

message Article {

  int32 id = 1;

  string title = 2;

  string content = 3;

  int32 author_id = 4;

}

message CreateArticleRequest {

  string title = 1;

  string content = 2;

  int32 author_id = 3;

}
```

次に、Node.js クライアントでこのサービスを使用するのは、ローカル関数を呼び出すように簡単です:

```
const grpc = require('@grpc/grpc-js');

const protoLoader = require('@grpc/proto-loader');

const PROTO_PATH = __dirname + '/blog.proto';

const packageDefinition = protoLoader.loadSync(PROTO_PATH, {

  keepCase: true,

  longs: String,

  enums: String,

  defaults: true,

  oneofs: true,

});

const blog_proto = grpc.loadPackageDefinition(packageDefinition).blog;

// gRPC メソッドを Promise に変換

function promisify(client, methodName) {

  return (request) => {

    return new Promise((resolve, reject) => {

      client[methodName](request, (error, response) => {

        if (error) reject(error);

        else resolve(response);

      });

    });

  };

}

async function main() {

  const client = new blog_proto.BlogService('localhost:50051', grpc.credentials.createInsecure());

  const getArticle = promisify(client, 'getArticle');

  const createArticle = promisify(client, 'createArticle');

  try {

    // 記事を取得

    const article = await getArticle({ article_id: 12345 });

    console.log('取得した記事:', article.title);

    // 記事を作成

    const newArticle = await createArticle({

      title: 'gRPC vs REST',

      content: 'gRPC は社内サービスに最適です...',

      author_id: 67890,

    });

    console.log('作成した新しい記事の ID:', newArticle.id);

  } catch (error) {

    console.error('エラー:', error);

  }

}

main();
```

この RPC スタイルの例では、次のことがわかります:

1. サービス定義が利用可能なすべての操作を明確にリストしています（この簡略化された例では `GetArticle` と `CreateArticle` ）。
2. 各操作には明確に定義されたリクエストとレスポンスのタイプがあります。
3. クライアントコードは、非同期のローカル関数を呼び出すように見えます。 `await` を使用して結果を待機し、ネットワーク通信の複雑さをさらに隠しています。
4. 手動で HTTP リクエストを構築したり、JSON レスポンスを解析したりする必要はありません。

潜在的に HTTP/2 をトランスポートプロトコルとして使用している場合がありますが、RPC フレームワーク（例えば gRPC）は、リモート呼び出しをローカル関数呼び出しのように見せる抽象化層を提供します。

したがって、"POST だけ" と RESTful API に関する議論の多くは、実質的にこれら 2 種類の API スタイル、すなわち REST と RPC に関する議論であることがわかります。しかし、重要なのは、これら 2 種類のスタイルがそれぞれの適用シナリオを持ち、選択はプロジェクトの具体的なニーズに基づいて行われるべきだということです。個人的な好みで選んではいけません。

## REST vs RPC: 絶対的な優劣はない

REST と RPC の違いを理解したところで、それぞれの適用シナリオを見てみましょう:

1. REST は次のような場合に適しています:
	- リソース指向のアプリケーション（例えば、コンテンツ管理システム、ブログプラットフォーム、E コマースウェブサイト）
	- 優れたキャッシュサポートが必要なシナリオ（GET リクエストは自然にキャッシュ可能です）
	- HTTP のセマンティクスを有効活用したいプロジェクト（適切なステータスコードの使用など）
	- 発見性や自己説明が求められる公開 API
2. RPC は次のような場合に適しています:
	- アクション指向のアプリケーション（例えば、複雑なデータ処理操作、ワークフローコントロール）
	- 高性能で低遅延が求められるシステム（例えばリアルタイム取引システム）
	- 内部マイクロサービス間の通信（より柔軟なパラメータ伝達が必要かもしれません）
	- 操作が単純に CRUD（Create, Read, Update, Delete）操作にマッピングできない場合

スタイルの選択はあなたの具体的なニーズに基づくべきです。場合によっては、違うパーツのニーズに応じて、同じシステム内でこれら 2 種類のスタイルを混在させることさえあります。

## 結論

1. API デザインの核心は、特定のメソッドやスタイルを固守することではなく、明確さ、一貫性、拡張性、効率にあります。
2. RESTful と RPC のどちらも成熟した API デザインのパラダイムです。選択はプロジェクトの要件に基づいて行われるべきで、個人的な好みに基づくべきではありません。
3. "POST だけ" を使用することを決定した場合は、RPC スタイルに従って API を設計してください。RESTful のように、それは業界でよく使用されている API 仕様ですが、適切なシナリオで使用されるべきです。
4. 表面的な技術的細部に惑わされないでください。重要なのは、あなたの API が効果的にユーザーやビジネスニーズに応えることができるかどうかです。
5. API を設計する際には、どの HTTP メソッドを使用するかにこだわるのではなく、リソースモデル、ビジネスロジック、ユーザーのニーズについてもっと考える時間を費やしましょう。

こうした無意味な議論から注意を逸らし、本当に優れた API を設計することに集中しましょう。結局のところ、テクノロジーは問題を解決するためのものであり、それを作り出すためのものではありません。[Logto Cloud を無料で試す](https://logto.io/?utm_source=blog&utm_medium=cta&utm_campaign=%2Fja%2Fpost-only-debate)