---
title: GraphQL(gqlgen) + Go(gin)のファイルアップロード(Postman)
source: https://zenn.dev/ygsn/articles/e047d74c5870ca
author:
  - "[[Zenn]]"
published: 2021-12-17
created: 2025-05-15
description: 
tags:
  - web
  - GraphQL
---
14[tech](https://zenn.dev/tech-or-idea)

## 概要

GraphQLサーバに `mutation` でファイルをアップロードしたいことがあり、備忘録も兼ねてまとめます。  
私は平時の業務ではサーバサイドの実装を行っており APIの動作確認やテストでPostmanを使用しております。

過去の実装時にPostmanからの動作確認方法がわからず、調べたら少々複雑＋当時日本語の記事があんまり無かった印象だったのを思い出したので記事にしました。

この記事のために **わざわざサンプルまで作った** ので、スクショを見たり文字を読むのが面倒な方はソースコードを見て頂いても良いかもしれません。  
ローカルで動作確認可能(のはず)です。 (Windows環境では多分動かないです😋)

動作確認環境

- macOS
- Go v1.17
	- gqlgen v0.14.0
	- gin v1.7.7
- Postman Version 9.4.1

## この記事で伝えたい嬉しかったこと

- GraphQL(gqlgen)+Go(gin)でファイルの受け取りが簡単にできるよ！
- Postmanで動作確認ができるので、開発フローの統一化や、API側の結合テストや負荷テストなどについて考えないといけないことが減る(可能性がある)よ！

**以下の内容はすでに素晴らしい記事やドキュメントが豊富にあるので割愛します。**

- gqlgenの設定などの使用方法 (`.gqlgen.yml` の設定値とか)
- Postmanとは？など基本的なところ

## GraphQL(gqlgen)+Go(gin)でファイルの受け取り

1. Schemaの定義
2. 自動生成されたソースの確認
3. 実装

の順番で見ていきます。

### Schemaの定義

#### Upload型の定義

schema/scalar.graphql

```graphql
"""
upload type
"""
scalar Upload
```

後にも出てくるのでざっくりと説明すると `multipart/form-data` で送信されたデータが格納されます。gqlgenがよしなにしてくれるのでとっても楽です。

#### mutationの定義

schema/file.graphql

```graphql
"""
upload mutation input
"""
input UploadInput {
  """
  upload file data
  """
  data: Upload!
}

"""
upload mutation output
"""
type UploadPayload {
  """
  uploaded file path
  """
  path: String!
}
```

schema/mutation.graphql

```graphql
type Mutation {
  """
  upload one file
  """
  uploadFile(input: UploadInput!): UploadPayload!
}
```

今回はサンプルなのでとてもシンプルにしています。  
入力として何かしらのファイルを受け取り、ファイルが格納されたパスを返すようにしています。  
弊社ではGitHub GraphQL APIの [ミューテーションについて](https://docs.github.com/ja/graphql/guides/forming-calls-with-graphql#about-mutations) を参考にし　 `input` `payload` をそれぞれの `mutation` で定義するスタイルを採用しています。

### 自動生成されたソースの確認

graphql/mutation.resolvers.go

```
func (r *mutationResolver) UploadFile(ctx context.Context, input model.UploadInput) (*model.UploadPayload, error) {
    panic(fmt.Errorf("not implemented"))
}
```

`generate` した結果がこちらです。想定通りのファイルが生成されていました。

### 実装

今回のサンプルの実装は以下です。

大きく分けて

1. インプットのデータを `io.ReadAll()` で読み込み
2. そのまま受け取ったデータを書き出し
3. データを書き出したファイルパスをペイロードとして返却

の3段構成です。

graphql/mutation.resolvers.go

```
func (r *mutationResolver) UploadFile(ctx context.Context, input model.UploadInput) (*model.UploadPayload, error) {
    const (
        fileDirName string      = "file"
        filePerm    fs.FileMode = 0o666
    )

    // io.Reader provided in input.File.File, so it is easy to throw it to S3 or something.
    file, err := io.ReadAll(input.Data.File)
    if err != nil {
        return nil, err
    }

    filePath := path.Join(fileDirName, input.Data.Filename)
    if _, err := os.Create(filePath); err != nil {
        return nil, err
    }
    if err := os.WriteFile(filePath, file, filePerm); err != nil {
        return nil, err
    }

    pwd, err := os.Getwd()
    if err != nil {
        return nil, err
    }
    return &model.UploadPayload{
        Path: path.Join(pwd, filePath),
    }, nil
}
```

## Postmanで動作確認

1. PostmanでのファイルアップロードのQuery発行
2. 動作結果の検証

の順番で見ていきます。

### Queryの発行

PostmanのBodyで `GraphQL` を選択してガリガリとQueryを書いていければ良いのですが、今回は「ファイルをアップロードしたい」ので何かしらのファイルを指定したいです。  
先にも言ったとおり今回の Upload型 は `multipart/form-data` である必要があります。そのため、Bodyの `form-data` を選択して以下のように入力する必要があります。

これでローカルのファイルも簡単に選択できますし、サーバ側の実装ではgqlgenがよしなにしてくれるのでとても楽です。 **嬉しいですね！**

| KEY | VALUE |
| --- | --- |
| `operations` (Text) | `{"query": "mutation ($i: Upload!){ uploadFile( input:{ data:$i } ){ path } }", "variables": { "i": null }}` |
| `map` (Text) | `{ "0": ["variables.i"] }` |
| `0` (File) | \[ファイルを選択する\] |

⬇こんな感じです

![](https://storage.googleapis.com/zenn-user-upload/c404b94c3e94-20211204.png)

※サンプルでは CONTENT TYPE は `AUTO` でも動作します。必要に応じて設定して下さい。

`curl` で見るとこんな感じです。

### 動作結果の検証

ファイルを選択すれば後は `Send` を押すだけです。  
サンプルでも以下のレスポンスでファイルパスを取得できました。

```json
{
    "data": {
        "uploadFile": {
            "path": "/Users/shinsakuyagi/src/gql-upload-sample/file/sample.png"
        }
    }
}
```

`mutation` 関数内に以下のような雑なプリントデバッグを仕込んで出力したログを記載します。

```
fmt.Printf("%v\n", input.Data)
```

```shell
[GIN-debug] Listening and serving HTTP on :8080
{{0xc0002dcb10} sample.png 6727 image/png}
[GIN] 2021/12/04 - 18:21:18 | 200 |    1.184674ms |             ::1 | POST     "/query"
```

ファイル自体のバイト列は **io.Readerインターフェースで提供される** ため、そのままAWSなどのSDKに後処理を投げるもよし、何かしら加工してからこねくり回すのもとても簡単に＆柔軟にできます。 **嬉しいですね！**  
ファイル名やファイルサイズ、ContentTypeなどもいい感じに取得格納してくれているのがわかるかと思います。

## まとめ

いかがだったでしょうか。ファイル名・ファイルサイズなども格納しておいてくれるのが地味に嬉しいですよね。  
PostmanのQuery作成部分が、調べて書いてみたら「なるほど」なのですが、当時は調べるのも書くのも自分的にはすこし ~~ダルい~~ 辛かったと言った所感でした。

今は色々文献や解説が出ていたりもしますし、Postmanで無くても良いのかもしれませんが、少しでもなにかの役に立てば幸いですし、Postmanはきちんと使うと便利です。個人的には推していきたい所存です。

## 補足・参考

### サンプルソースコード

### 参考・補足記事

#### gqlgen, ginのインテグレーション

#### gqlgenのスカラ型(Upload Type)

#### GraphQLのvariablesについて

#### Postmanでのファイルアップロード方法

14