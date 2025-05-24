---
title: LaravelでGraphQLを使うために学んだメモ
source: https://zenn.dev/yum3/articles/t_search_laravel_graphql#graphql%E3%81%A8%E3%81%AF
author:
  - "[[Zenn]]"
published: 2022-07-02
created: 2025-05-15
description: 
tags:
  - web
  - GraphQL
  - Laravel
---
26[tech](https://zenn.dev/tech-or-idea)

## GraphQLとは

### 概要

- GraphQLは仕様
- GraphQLは言語
- GraphQLはサービス

情報のグラフ構造に着目してクエリ言語を用いて情報を操作しようという考え方。  
クライアント/サーバー通信のための言語仕様でもある。

### GraphQLはRESTの代替品ではない

GraphQLは翻訳者である。  
GrapQLはWebが発達するにつれてRESTが適合しない状況が出てきたため、誕生した。

#### GraphQLとRESTを同等と誤解する図

![](https://storage.googleapis.com/zenn-user-upload/759b6339d7b6-20220702.png)

GraphQLはインターフェースであり、 **RESTの代替ではない** 。  
RESTからデータを取ってくるようにできるし、DBからデータを取ってくるようにもできる。

#### GraphQLの実体図

![](https://storage.googleapis.com/zenn-user-upload/4a06079a52e3-20220702.png)

### RESTとの比較

#### エンドポイント数が増えない

クライアント側から見ると `/graphql` だけである

#### バージョン管理が必要ない

常にクライアント側からは同じバージョンに見える

#### パフォーマンス面で無駄なケースを解消できる

RESTの過剰なレスポンスを必要最低限に抑えることでスピードをあげたり、多数アクセスによる情報取得を1回のクエリで済ませる。

また、フロントエンド〜バックエンド間のすり合わせで開発速度が低下するのを避ける

### GraphQLの問題

- リソース枯渇攻撃
	- 深いネストや同じフィールドを何度も要求するよくない実装が引き起こす
- クライアントによるデータのキャッシュが難しい
- N+1問題がある

### GraphQLのスキーマ設計

APIはRESTのエンドポイントの集合ではなく、型の集合と捉えることができる。  
そのデータ型の集合を **スキーマ** と呼ぶ。

スキーマファーストで設計していくのであればGraphQLは輝く。  
スキーマファーストで考えるということは、「フロントエンドのためのバックエンド（BFF）」である。  
「クラアント側から何ができるか」という観点から設計を始めていく必要がある。UI駆動スキーマ設計のことである。

### GraphQLスキーマガイド

#### 命名規則

##### Query

- リソース名（名詞）とその複数形
- `repository`, `repositories`

##### Mutation

- 動詞＋名詞の形 `addStar`, `createIssue`
	- Mutationの名前+Input
	- Mutationの名前+Payload

#### Global Object Identification

- 全データをIDにより一意に特定できるようにする仕様
- users tableのidを一意にしたければ `User:1` など

```graphql
type Query {
    node(id: ID!): Node
}

interface Node {
    id: ID!
}

type User implements Node {
    id: ID!
    name: String
}
```

- Nodeに型指定をするとデータが取得する
- `nodes(ids: [ID!]!): [Node]!`もついでに定義するとよい

#### Cursor Connections

- カーソルを使ってリストを順にたどるための仕様
	- `Repository` のリストを返す場合、 `[Repository]` を返す代わりに `RepositoryConnection` を返す
1. リスト型のフィールドの代わりに、Connectionサフィックスの型を作る
2. first: Int!とafter: Stringを引数に設ける
3. Connectionはcursorを持てるEdgeとページング情報をもつPageInfoをもつ

```graphql
type Query {
    repositories(
        first: Int
        after: String
        
        last: Int
        before: String
    ): RepositoryConnection
}

type RepositoryConnection {
    pageInfo: PageInfo
    edges: [RepositoryEdge]
}

type PageInfo {
    hasPreviousPage: Boolean!
    startCursol: String
    hasNextPage: Boolean!
    endCursol: String
}

type RepositoryEdge {
    cursor: String!
    node: Repository
}

type Repository {
    id: ID!
    name: String!
}
```

#### Queryでinputはアンチパターン

- フロント側的には `input` でまとめなくても `variables` でまとまる
- 引数を明示的に指定するのは2 ~ 3がほとんどであまり多くならない
- デフォルト値の指定が活用できずつらい

### GraphQLサーバー

GraphQLサーバーには **リゾルバ** を実装する。  
**リゾルバ** はREST API, データベース, その他のサービスからデータを取得したり更新したりできる。

### GraphQLの操作

RESTに対応付けて表現するとGraphQLにはルートリゾルバと呼ばれる操作が３つある。

- Query (GET)
- Mutation (POST, PUT, DELETE)
- Subscription(データのSubscribe)

これらの操作でデータの集合を扱っていく

詳解の説明は [こちら](https://speakerdeck.com/nagano/phptographql)

### まとめ

- GraphQLは翻訳者としての立ち位置なので応用が効きそうだが、銀の弾丸ではないことを理解する
- LaravelでGraphQLを扱う際にもスキーマ設計が必要になるので、設計する必要がある

### 参考リンク

<iframe src="https://www.youtube-nocookie.com/embed/_U6Ft8op9Ik" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

## Laravel x GraphQL

LaravelでGraphQLを扱うために [Lighthouse](https://lighthouse-php.com/) という便利なライブラリを使う。  
Eloquentと連動していて、スキーマ定義にディレクティブを付けることで補完してくれる、というライブラリである。

### Install

1. `composer require nuwave/lighthouse`
2. `php artisan vendor:publish --tag=lighthouse-config`
	1. [CORS設定](https://lighthouse-php.com/5/getting-started/configuration.html#cors)
3. スキーマ定義する

### Dev Tools

#### PlayGround

```shell
composer require mll-lab/laravel-graphql-playground
```

#### IDE Plugin

### Lighthouse UseCase

## Schema

### Type

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  created_at: String!
  updated_at: String
}
```

#### Object Type

Eloquentモデルと密接に関連していて、一意の名前を持ち、一連のフィールドを含んでいる必要がある。

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  created_at: String!
  updated_at: String
}

type Query {
  users: [User!]!
  user(id: ID!): User
}
```

#### Scalar

- [Lighthouseデフォルトのスカラー型](https://lighthouse-php.com/5/api-reference/scalars.html)
- `php artisan lighthouse:scalar <Scalar name>` で独自のスカラー型を定義することができる
- サードパーティによる [mll-lab/graphql-php-scalars](https://github.com/mll-lab/graphql-php-scalars) スカラーが型も使用できる
- [独自のスカラー型を実装する](https://webonyx.github.io/graphql-php/type-definitions/scalars/)

#### Enum

`@enum` ディレクティブを使用して値を定義できる

```graphql
enum EmploymentStatus {
  INTERN @enum(value: 0)
  EMPLOYEE @enum(value: 1)
  TERMINATED @enum(value: 2)
}
```

### Root Type

#### Query

```graphql
type Query {
  me: User
  users: [User!]!
  userById(id: ID): User
}
```

#### Mutation

```graphql
type Mutation {
  createUser(name: String!, email: String!, password: String!): User
  updateUser(id: ID, email: String, password: String): User
  deleteUser(id: ID): User
}
```

#### Subscription

```graphql
type Subscription {
  newUser: User
}
```

### Fields

- デフォルトでは、Lighthouseは `App\GraphQL\Queries` また `App\GraphQL\Mutations` の名前空間下でフィールドの大文字の名前を持つクラスを検索し、 `__invoke` を使い、 [通常のリゾルバ引数](https://lighthouse-php.com/5/api-reference/resolvers.html#resolver-function-signature) を使用して呼び出す。
- `php artisan lighthouse:query <QueryName>` で生成できる
- `$args` はクエリに渡される引数の連想配列

### ディレクティブ

- `@` で始まり、一意の名前が続く
- [Directives API](https://lighthouse-php.com/5/api-reference/directives.html)

## Eloquent

- READ
	- `@all` でモデルの名前が解決しているフィールドの戻りタイプと同じであると想定し、Eloquentで自動解決する
	- `@paginate` でページネーション
	- `@eq` で追加制約をクエリに追加できる
	- `@orderBy` でソート
- CREATE
	- `@create` でデータ作成
	- `@update` でモデル更新
	- `@upsert`
	- `@delete`

## スキーマ構成

- `schema.graphql` からスキーマを読み取る
- インポートにワイルドカードも使える

```graphql
type Query {
  user: User
}

#import user.graphql
```

#### Settings

##### Security

```
'security' => [  
    'max_query_complexity' => \GraphQL\Validator\Rules\QueryComplexity::DISABLED,  
    'max_query_depth' => \GraphQL\Validator\Rules\QueryDepth::DISABLED,  
    'disable_introspection' => \GraphQL\Validator\Rules\DisableIntrospection::DISABLED,  
],
```

- クエリの深さを制限する場合は、 `max_query_depth` を設定する
- クエリの複雑性で制限する場合は、 `max_query_complexity` を設定する
- イントロスペクションを無効にする場合は、 `disable_introspection` を設定する

[GitHubで編集を提案](https://github.com/A-yum3/zenn-post/blob/main/articles/t_search_laravel_graphql.md)

26