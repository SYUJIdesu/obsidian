---
title: "「MCP？聞いたことあるけど使ってない…😅」人向けに初歩から少し踏み込んだ内容まで解説"
source: "https://zenn.dev/yamada_quantum/articles/465c4993465053"
author:
  - "[[Zenn]]"
published: 2025-03-13
created: 2025-05-15
description:
tags:
  - "web"
  - "MCP"
---
391

168

今回は一気に「MCPなにもわからない」から「MCP完全に理解した」に一気にレベルアップすることを目的に書いています。

そのために以下をモリモリに解説していきます。

- ModelContextProtocol(MCP)とは？
- MCPがあることでできること
- MCPを実装するライブラリmodelcontextprotocolを使ってチュートリアル
	- 実装のためのネゴシエーションや通信プロトコルの説明も踏まえてのチュートリアルです。
- CursorへのMCPサーバーの登録方法
- MCPがどのように動作してツールが使われるのか？
- FunctionCallingとの違い

MCPって単語聞きすげてわからないままに嫌になっている人はこれを読むことで解放されてください。

## ModelContextProtocol(MCP)とは？

まずはイメージを見てもらうとわかりやすいと思います。  
![](https://storage.googleapis.com/zenn-user-upload/c39470a5b2b1-20250308.png)  
([https://modelcontextprotocol.io/introductionより引用](https://modelcontextprotocol.io/introduction%E3%82%88%E3%82%8A%E5%BC%95%E7%94%A8))  
この図は、MCPクライアント(ClaudeなどのAIモデル、CursorなどのIDEである)が **MCPサーバーを通してツールを利用できるよ** ということを表しています。  
なるほど。ただイメージはわかりますが、MCPとはなんでしょうか？  
MCPというのはAIモデルにツールを使わせるために標準化したルールを決めようというものです。

すこし難しいのでもう少し噛み砕きます。  
MCPを提唱したAnthoropic社が出しているOSSのmodelcontextprotocolのドキュメントを引用します。

> MCP は、アプリケーションが LLM にコンテキストを提供する方法を標準化するオープン プロトコルです。MCP は、AI アプリケーション用の USB-C ポートのようなものです。USB-C がデバイスをさまざまな周辺機器やアクセサリに接続するための標準化された方法を提供するのと同様に、MCP は AI モデルをさまざまなデータ ソースやツールに接続するための標準化された方法を提供します。  
> [https://modelcontextprotocol.io/introduction](https://modelcontextprotocol.io/introduction)

めちゃくちゃわかりやすいですね。  
MCPは **AIモデルにいろんなツールを繋げるためのUSBポート** なのです。  
Xなどで「〜なMCPサーバーを作った！」などを見かけた時は、  
「標準化したルールでAI用のツールを作ったんだな！」と思ってもらえるといいと思います！

## MCPがあることで以下のようなことが簡単にできる

ではMCPがあることでどんないいことがあるのでしょうか？  
CursorやClineに

- **Figma API** を使わせてデザインからコードに変換してもらえる
- **GoogleDocument** にチャット内容を保存してもらえる
- **GitHubAPIで** PRを作らせることができる  
	etc...

めちゃくちゃ嬉しいですね...  
「ただAIモデルにツールを使わせることができるってことが言いたいなら、今までFunctionCallingなどのAIモデルにツールを使わせる方法があったじゃないか！MCPのいいところじゃない！」  
って反論があるかもですが、それは違います。  
MCPのいいところは **ツールを使わせる方法が標準化したこと** なのです。  
これのおかげでいろんなツールを作っている会社などがMCPサーバーを公開することで、我々の使うAIモデルがその恩恵を受けることができるのです。  
本当にありがたい...

例えば、MCPができたことで以下のようなMCPサーバーを提供する企業があります。

ClineでのMCPマーケットプレイスなどもできています。

あとこんなのもあります。

## MCPサーバー作成チュートリアル（TypeScript+bun）

ではみなさんそんなMCPサーバーは **Anthoropic社が出しているOSSのmodelcontextprotocol** を使うことで簡単に自作できます。

今回は簡単に **modelcontextprotocolのTypeScript SDK + Bun** を使ったチュートリアルを作りました。

### プロジェクトのセットアップ

まず、新しいプロジェクトディレクトリを作成し、Bunを使って初期化します。

```bash
mkdir mcp
cd mcp
bun init
```

### 依存関係のインストール

次に、必要な依存関係をインストールします。

```bash
bun add @modelcontextprotocol/sdk zod
```

### MCPサーバーの実装

では **「現時刻を表示する」MCPサーバー** を実装しましょう。  
実装手順を簡単に言語化すると、以下になります。

1. `@modelcontextprotocol/sdk` からMCPサーバーを用意する。
2. MCPサーバーの機能ネゴシエーションを準備する
	- サーバに対して、toolを定義する。(今回は「現時刻を表示する」toolを定義します。)
3. MCPサーバーのTransportLayerを準備する
	- StdioServerTransportを使って、MCPサーバーを標準の入力および出力ストリームを介した通信をできるようにする。

#### 1\. @modelcontextprotocol/sdkからMCPサーバーを用意する。

まずはこの部分を実装しましょう。  
![](https://storage.googleapis.com/zenn-user-upload/d85bf141a0cf-20250312.png)

コードは簡単で以下です。

```typescript
// MCPサーバーを作成
const server = new McpServer({
    name: "時間表示サーバー",
    version: "1.0.0"
});
```

#### 2\. MCPサーバーの機能ネゴシエーションを用意する

機能ネゴシエーションを用意しましょう。イメージがつきやすいものに置き換えるとHTTPメソッドみたいなやつです。  
大きく3種類あり、以下です。

| 項目 | 説明 |
| --- | --- |
| [Prompt](https://modelcontextprotocol.io/docs/concepts/prompts) | サーバーは再利用可能なプロンプト テンプレートとワークフローを定義でき、クライアントはそれをユーザーや LLM に簡単に表示できます。基本はユーザーが制御するもので、ユーザーが明示的に選択する |
| [Resource](https://modelcontextprotocol.io/docs/concepts/resources) | MCP サーバーがクライアントに提供したいあらゆる種類のデータを提供する。アプリケーションが制御するもの。 |
| [Tool](https://modelcontextprotocol.io/docs/concepts/tools) | アクションを実行するためにLLMに公開される関数。基本モデルが制御するもので、世の中の人がMCPと思っているのもこれ |

~~HTTPメソッドとの比べてイメージする場合は  
Resource →　GET  
Tool →　POSTに近いです。  
Promptは当てはまるものないですね-!~~  
あんまり正しくない説明なので取り消します。

この辺りはMCPの仕様書を見てみてください

今回は時間表示なのでToolを使いましょう。実装は以下です。

```typescript
server.tool(
    "get-current-time",
    { format: z.enum(["full", "date", "time"]).optional() },
    async ({ format = "full" }) => {
        const now = new Date();
        let timeString = "";

        switch (format) {
            case "date":
                timeString = now.toLocaleDateString("ja-JP");
                break;
            case "time":
                timeString = now.toLocaleTimeString("ja-JP");
                break;
            case "full":
            default:
                timeString = now.toLocaleString("ja-JP");
                break;
        }

        return {
            content: [{
                type: "text",
                text: \`現在の時刻は ${timeString} です。\`
            }]
        };
    }
);
```

#### 3.MCPサーバーのTransportLayerを準備する

以下画像のこの部分を用意しないといけません。  
![](https://storage.googleapis.com/zenn-user-upload/8bdd73b129cf-20250312.png)

基本MCPサーバーとMCPクライアントのやり取りはJSON-RPC2.0というプロトコルでやりとりしています。少し噛み砕くとあらかじめ決められたJSONのルールに沿って、リクエストや回答、通知といったメッセージのやり取りをしてます！

そして、MCPには2つの通信方法（トランスポート）が用意されています。

- 標準入出力（stdio）  
	これは、パソコンのキーボードや画面のような「標準の入力と出力」を使ってデータをやり取りする方法で、コマンドラインツールなど、シンプルなローカル環境で利用します。ローカルMCPサーバーの場合は基本これ
- サーバー送信イベント（HTTP＋SSE）  
	こちらは、HTTP POSTリクエスト+SSEエンドポイントの2つを使って、サーバーからクライアントへデータを連続的に送る方法です。  
	LLMをツールとして使いたいとかならこれですね！  
	最近Streamable HTTPが出てこれはレガシー機能になりました。

詳しくはこちらを見てみてください！

では実装です。今回はstdinでやりましょう！

```typescript
const transport = new StdioServerTransport();
await server.connect(transport);
console.log("時間表示MCPサーバーが起動しました。");
```

### MCPサーバー実装まとめ

これだけの実装でMCPサーバーを作れます。簡単ですね。  
最後に実装だけまとめておきます。

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// MCPサーバーを作成
const server = new McpServer({
    name: "時間表示サーバー",
    version: "1.0.0"
});

server.tool(
    "get-current-time",
    { format: z.enum(["full", "date", "time"]).optional() },
    async ({ format = "full" }) => {
        const now = new Date();
        let timeString = "";

        switch (format) {
            case "date":
                timeString = now.toLocaleDateString("ja-JP");
                break;
            case "time":
                timeString = now.toLocaleTimeString("ja-JP");
                break;
            case "full":
            default:
                timeString = now.toLocaleString("ja-JP");
                break;
        }

        return {
            content: [{
                type: "text",
                text: \`現在の時刻は ${timeString} です。\`
            }]
        };
    }
);

// サーバーを起動
const transport = new StdioServerTransport();
await server.connect(transport);

console.log("時間表示MCPサーバーが起動しました。");
```

## MCPクライアントとしてCursorを利用する

MCPサーバー作成したらあとは登録です。私がCursorをよく使っているので、Cursorを例に書いていきます。

基本は以下のドキュメントに書いてることなので、こちらを見てもらえるといいです。ただいくつかはまった瞬間もあるのでトラブルシュートがてら書いておきます！

1. Cursor Settings > Features > MCP に移動して 「+ Add New MCP Server」をクリックする  
	![](https://storage.googleapis.com/zenn-user-upload/ab206511f6a0-20250309.png)
2. LLMが使えるMCPサーバーを登録する  
	![](https://storage.googleapis.com/zenn-user-upload/b59f3512d36c-20250310.png)
- Name
	- あなたの好きな名前を入れてください。
- Type
	- Command
	- サーバーからクライアントへのストリーミングのみ必要な場合は **SSE** を使ってください。LLMをMCPサーバーで使う場合とかはSSEですね・
- Command
	- `bun run ~/Users/username/dev/mcp/index.ts`
	- ファイルの場所は基本絶対パスで入れてください。そうじゃないと認識してくれないことがあります。それに加えてbunコマンドも絶対パスを指定しないとうまく認識してくれないことがありましたので、うまくいかなかった場合は絶対パスで指定してください
1. チャットで実行！！  
	![](https://storage.googleapis.com/zenn-user-upload/7375730beb4d-20250310.png)

夢が...夢が広がりますね...

## MCPがどのように動作してツールが使われるのか？

![](https://storage.googleapis.com/zenn-user-upload/a37527f9db8d-20250312.png)

1. MCPクライアント：MCPサーバーに「どんなツールがある？」と問い合わせ。
2. MCPサーバ：MCPクライアント経由で「現在時刻取得ツール」などの一覧を返す。
3. LLMアプリ：AIモデルに「今の時間を知りたい」とリクエスト。
4. AIモデル:「現在時刻取得ツールを使えば？」と提案。
5. MCPクライアント:MCPサーバーの「現在時刻取得ツール」を実行
6. MCPサーバー:ツールを実行し、「今は12時です」をMCPクライアントに返却
7. LLMアプリ:結果をAIモデルに伝える（「今は12時ですよ」）。
8. AIモデル:ツールの実行結果をもとに回答を生成

大まかな流れはこんな感じで、アプリがMCPサーバーを通してツールを使い、その結果をAIモデルにわたし回答を生成しています。  
FunctionCallingも実は同じですね。

また最近MCPの動作原理がClineを例に書かれているやつがあったので、  
MCPがエージェントで使われるときのイメージなどをもっと詳しく理解したい方はこちらを見るとより理解が深まると思います。

## FunctionCallingとは何が違うの？

「LLMにツールを使わせる」という点では今までにOpenAIのFunctionCallingが出ていて有名です。MCPというのはAIモデルにツールを使わせるために標準化したルールを決めようというものでした。FunctionCallingと何が違うのでしょうか？まずはFunctionCallingのフローを見てみましょう。  
![](https://storage.googleapis.com/zenn-user-upload/bc1e4c6d684a-20250312.png)

MCPサーバーとMCPクライアントがないだけで実はやっていることは同じなんですよね。  
じゃあ何が違うかというと **MCPサーバーとMCPクライアントがあるかないかです。** （真剣）

それを理解するために実際にChatGPTにFunctionCallingをさせるときのPythonコードを見てみましょう。

```python
import openai
import json

# OpenAI APIキーの設定
openai.api_key = 'YOUR_OPENAI_API_KEY'

# 関数の定義
def get_current_weather(location):
    # ここで実際の天気情報を取得する処理を実装
    return {
        "location": location,
        "temperature": "22°C",
        "description": "晴れ"
    }

# 関数の仕様を定義
functions = [
    {
        "name": "get_current_weather",
        "description": "指定された場所の現在の天気を取得します。",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "天気を取得したい場所の名前"
                }
            },
            "required": ["location"]
        }
    }
]

# ユーザーからのメッセージ
messages = [
    {"role": "user", "content": "東京の現在の天気は？"}
]

# ChatCompletionの呼び出し
response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=messages,
    functions=functions,
    function_call="auto"  # LLMが適切な関数を自動的に呼び出す
)

# レスポンスから関数呼び出しの情報を取得
response_message = response["choices"][0]["message"]

if "function_call" in response_message:
    # 関数名と引数を取得
    function_name = response_message["function_call"]["name"]
    function_args = json.loads(response_message["function_call"]["arguments"])

    # 関数を実行
    if function_name == "get_current_weather":
        function_response = get_current_weather(function_args.get("location"))

    # 関数の実行結果をメッセージに追加
    messages.append(response_message)  # 関数呼び出しのメッセージを追加
    messages.append({
        "role": "function",
        "name": function_name,
        "content": json.dumps(function_response)
    })

    # 関数の実行結果を元に再度ChatCompletionを呼び出す
    second_response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=messages,
    )

    # 最終的なアシスタントからの応答を取得
    assistant_reply = second_response["choices"][0]["message"]["content"]
    print(assistant_reply)
else:
    # 関数呼び出しが不要な場合の応答を取得
    assistant_reply = response_message["content"]
    print(assistant_reply)
```

ここで抑えて欲しいのは以下のような定義がChatGPT固有のものであることです。

```bash
{
        "name": "get_current_weather",
        "description": "指定された場所の現在の天気を取得します。",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "天気を取得したい場所の名前"
                }
            },
            "required": ["location"]
        }
    }
```

これ実はAIモデルが別になると全然違うのです。  
例えばVercel AI SDKの実装などをみると以下のようなモデルごとのFunctionCallingのインターフェイスの違いを頑張って吸収しようとしているのがわかります。

- OpenAI版：functionオブジェクトの中にname、description、parametersを含む。
- Anthropic版：同様にname、descriptionを定義しますが、パラメータはinput\_schemaというキー名で扱います。  
	[https://github.com/vercel/ai/tree/main/packages](https://github.com/vercel/ai/tree/main/packages)

FunctionCallingはそのツールとLLMを繋げるインターフェイスがOpenAIのようなベンダーに依存しているのです。  
MCPの場合は、MCPサーバーとMCPクライアントが存在し、一定のプロトコルに沿ってやりとりするため、特定のAIモデルに依存せずいい感じにやり取りができます。

違いをまとめると

- FunctionCalling:各AIモデル（OpenAI、Anthropicなど）の仕様に依存し、インターフェイスがモデルごとに異なる。
- MCP：MCPサーバーとMCPクライアント間の共通プロトコルに基づいており、特定のAIモデルに依存しません。

FunctionCallingとMCPの違いについて書いてる記事がさがすと他にも色々とあり面白かったのでぜひ読んでみてください。

## 最後に

Xやってるのでぜひフォローお願いします。とはいえ最近全然更新してない...

391

168