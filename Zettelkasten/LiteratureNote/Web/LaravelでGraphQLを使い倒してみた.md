---
title: LaravelでGraphQLを使い倒してみた
source: https://qiita.com/TsukasaGR/items/1a8b0020e5e83e7a46c7
author:
  - "[[TsukasaGR]]"
published: 2018-12-09
created: 2025-05-22
description: はじめにLaravel Advent Calendar 2018 - Qiitaの10日目の記事です！1年前くらいから聞く機会がぐっと増え、最近懐疑的な意見がだんだん増えてきた印象を個人的に持っ…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から5年以上が経過しています。

## はじめに

[Laravel Advent Calendar 2018 - Qiita](https://qiita.com/advent-calendar/2018/laravel) の10日目の記事です！

1年前くらいから聞く機会がぐっと増え、最近懐疑的な意見がだんだん増えてきた印象を個人的に持っているGraphQLですが、

- RestfulライクなAPIの代替案として有効なのか自分で触って確かめてみたかった
- Laravelで使い倒す記事を見かけていなかった

のでこの機会に使い倒してみました(と言っても多少踏み込んだ程度ですが😅)。  
※実装例は順次追加していきます。

なお、この記事では

- GraphQLとは何か
- GraphQLの諸々(クエリ、ミューテーション)の説明
- クエリの記述方法の説明

などは記載しません。  
(記述方法についての説明はしませんが、使い倒す中で実行するクエリは記載していきます)

## 前提

php: 7.1.3  
laravel/framework: 5.7.16  
nuwave/lighthouse: 2.6

## つくるもの

記事投稿&SNS(News◯icksみたいな感じです)のAPIを題材に使い倒してみようと思います。  
ER図はこんな感じです。  
[![スクリーンショット 2018-12-10 7.45.55.png](https://qiita-image-store.s3.amazonaws.com/0/173370/99b6b78b-5274-2938-aa1e-c7c8d7fb81c1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2F99b6b78b-5274-2938-aa1e-c7c8d7fb81c1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1fa4005defab2b032b94a45df119749e)

## 使うライブラリ

LaravelでGraphQLを使う場合いくつかライブラリはありますが、今回は前提にある [lighthouse](https://lighthouse-php.com/2.6/getting-started/installation.html) にしました。

lighthouseは [こちら](https://qiita.com/acro5piano/items/b9e3acd4af5ec14d7bb1) の記事が詳しく書かれていてわかりやすいと思います。

## 成果物

[こちら](https://github.com/TsukasaGR/laravel-lighthouse-example) に今回の成果物を置いてあります。  
※追記分は反映出来てません

## 下準備

上記ER図のような構成をつくる為、マイグレーション、リレーション、シーダーを作成します。  
説明は割愛します。  
[成果物](https://github.com/TsukasaGR/laravel-lighthouse-example) 内にありますので気になる方はそちらをご覧ください。

## Lighthouse導入

ここからが本題です。  
[こちら](https://lighthouse-php.com/master/getting-started/installation.html) を参考に入れていきます。

まずはライブラリのインストールします。

```sh
$ composer require nuwave/lighthouse
```

続いて、vendor:publishで初期設定をします。

```sh
$ php artisan vendor:publish --provider="Nuwave\Lighthouse\Providers\LighthouseServiceProvider" --tag=schema
```

routes/graphql/schema.graphqlが出来上がっていると思います。

DevToolsであるlaravel-graphql-playgroundも入れます。  
(フロントエンド等で別途GraphQLクライアントを用意されている場合でもスキーマ定義確認など使いどころはあるかなと思います。)

```sh
$ composer require mll-lab/laravel-graphql-playground
```

最後に、playgroundもvendor:publishして初期設定をします。  
playgroundは [こちら](https://github.com/mll-lab/laravel-graphql-playground) に説明があります。

```sh
php artisan vendor:publish --provider="MLL\GraphQLPlayground\GraphQLPlaygroundServiceProvider"
```

以上で導入は完了です。  
ファサードは今回は使わなかったので設定していません。

## クエリを実行する

## シンプルなクエリ

まずは簡単なクエリを実行してみます。  
routes/graphql/schema.graphqlにクエリが登録されているのでそちらを実行します。

```graphql
type Query {
    user(id: ID @eq): User @find(model: "App\\User")
}
```

laravel-graphql-playgroundで実行するのでアクセスします。  
[こちら](https://github.com/mll-lab/laravel-graphql-playground) のUsageにある通り、/graphql-playgroundというURLにアクセスすると表示されると思います。  
(`http://127.0.0.1:8000` の環境の場合は `http://127.0.0.1:8000/graphql-playground` です)

アクセスするとこんな画面が表示されると思います。  
[![スクリーンショット 2018-12-09 2.38.09.png](https://qiita-image-store.s3.amazonaws.com/0/173370/0dc83f87-bdd0-5671-0f4b-5b262cab6b12.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2F0dc83f87-bdd0-5671-0f4b-5b262cab6b12.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=dd9e17462334ebe6eb81c00e99dd996a)

アクセス出来たら早速クエリを発行してみます。

```graphql
query {
  user(id: 1) {
    id
    name
    email
  }
}
```

無事発行して取得出来ました！  
[![スクリーンショット 2018-12-09 2.41.23.png](https://qiita-image-store.s3.amazonaws.com/0/173370/1a3b705d-3543-2d40-ec4e-743421881e3e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2F1a3b705d-3543-2d40-ec4e-743421881e3e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4100b7ab610cb5cd97eac4f557f9c02e)

## リレーションを使ったクエリ

次に、リレーションを使ったクエリも実行してみます。  
Userに紐付いているArticlesを取得してみます。

まずはArticleのTypeを作成します。

```graphql
type Article {
    id: ID!
    userId: Int! @rename(attribute: user_id)
    title: String!
    body: String!
    createdAt: DateTime! @rename(attribute: created_at)
    updatedAt: DateTime! @rename(attribute: updated_at)
}
```

実際にGraphQLを利用するのはフロントエンドがほとんどかなと思うので `@rename` ディレクティブでカラム名をリネームしてキャメルケースに変換しています。

続いて、UserのTypeにarticlesを追加します。

```diff
type User {
    id: ID!
    name: String!
    email: String!
+   articles: [Article] @hasMany
    created_at: DateTime! @rename(attribute: created_at)
    updated_at: DateTime! @rename(attribute: updated_at)
}
```

準備出来たのでリレーションを使ったクエリを実行します。

```graphql
query {
  user(id: 2) {
    id
    name
    email
    articles {
      id
      userId
      title
    }
  }
}
```

無事articlesも取得出来ました！  
[![スクリーンショット 2018-12-09 2.57.06.png](https://qiita-image-store.s3.amazonaws.com/0/173370/81eb9a4a-9131-2f9b-5e1d-ce45ef5b0b1f.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2F81eb9a4a-9131-2f9b-5e1d-ce45ef5b0b1f.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=23fd32d35d642fc07378b3c147f8ef00)

## paginatorのデータを取得するクエリ

最後に、複数件のデータを取得します。  
こちらもroutes/graphql/schema.graphqlにクエリが登録されているのでそちらを実行します。

```graphql
type Query {
    users: [User!]! @paginate(type: "paginator" model: "App\\User")
}
```

paginatorではcountの引数が必須なので入れてあげます。

```graphql
query {
  users(
    count: 10
  ) {
    data {
      id
      name
      email
      articles {
        id
        userId
        title
      }
    }
    paginatorInfo {
      currentPage
    }
  }
}
```

ページネーションでも無事取得出来ました！  
[![スクリーンショット 2018-12-09 9.13.05.png](https://qiita-image-store.s3.amazonaws.com/0/173370/b302f3e2-5fa8-47ff-a95b-dbb1baf22f6c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2Fb302f3e2-5fa8-47ff-a95b-dbb1baf22f6c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=07a124ac5478f6a2242d1ee18d9fe539)

複数件データを取得する際は、

- ベースとなるモデルのhasMany等で取得する
- paginatorで取得する(typeは２種類)

の２択しかないのかなと思います。  
ベースとなるモデルをpaginatorなしの純粋なCollectionとして引っ張ってきたいなぁと思われる方いらっしゃるのかなと思いますが、その場合は自作のresolver登録になるのかなと思います。(resolver作らなくても出来るよ、という情報お持ちでしたらぜひコメント頂けますと幸いです)

playgroundはソースコードをもとにスキーマ情報を出力してくれているので、

- どんなクエリがある？
- このクエリの引数、戻り値は？

と悩んだ時はスキーマ情報を見てもらえればと思います。

Typeはドリルダウンして見ることが出来ます。  
[![スクリーンショット 2018-12-09 9.18.42.png](https://qiita-image-store.s3.amazonaws.com/0/173370/e6ccc516-2d07-6f52-2fe6-e7b690cd4e96.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2Fe6ccc516-2d07-6f52-2fe6-e7b690cd4e96.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=dc07857b45db7f009e31105ff67ab6a9)

サジェストが効いてくれるのもありがたいですね。  
[![スクリーンショット 2018-12-09 9.26.31.png](https://qiita-image-store.s3.amazonaws.com/0/173370/115d6455-e848-5023-7336-38a35cebb03c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2F115d6455-e848-5023-7336-38a35cebb03c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d02a61fa5ef31aa834c8c5660e553791)

ただし、ホットリロードには対応していないので、ソースコードを修正したら画面をリロードする必要があります。

## ミューテーションを実行する

## 登録

クエリが出来たところでミューテーションも実行してみます。  
まずはUserを登録します。

routes/graphql/schema.graphqlに登録されているミューテーションを少し加工します。

```graphql
type Mutation {
    createUser(
        name: String
            @rules(apply: ["required"])
        email: String
            @rules(apply: ["required",
                "email",
                "unique:users,email"]
            )
        password: String
            @bcrypt
            @rules(apply: ["required"])
    ): User
        @create(model: "App\\User")
}
```

登録するミューションはこんな感じです。

```graphql
mutation {
  createUser(
    name: "taro"
    email: "graphql@example.com"
    password: "secret"
  ) {
    id
    name
    email
  }
}
```

無事登録出来ました！  
[![スクリーンショット 2018-12-09 9.36.17.png](https://qiita-image-store.s3.amazonaws.com/0/173370/c4014633-339b-45d2-9a5e-de5d7ef50fa2.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2Fc4014633-339b-45d2-9a5e-de5d7ef50fa2.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=325e04f65bf361de5099f6fa69f091b9)

## 更新

更新処理も行きます。  
こちらはもともとあるものをそのまま使います。

```graphql
type Mutation {
    updateUser(
        id: ID
            @rules(apply: ["required"])
        name: String
        email: String
            @rules(apply: ["email"])
    ): User
        @update(model: "App\\User")
}
```

更新してみます。

```graphql
mutation {
  updateUser(
    id: 51
    name: "hanako"
  ) {
    id
    name
  }
}
```

更新も無事出来ました！  
[![スクリーンショット 2018-12-09 9.36.17.png](https://qiita-image-store.s3.amazonaws.com/0/173370/c4e814fd-c0bf-db86-c2fe-e2196fa0fb56.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2Fc4e814fd-c0bf-db86-c2fe-e2196fa0fb56.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3e740068781906ea8615657b6614e9c9)

## 削除

最後に削除もします。

こちらはもともとあるものをそのまま使います。

```graphql
type Mutation {
    deleteUser(
        id: ID @rules(apply: ["required"])
    ): User @delete(model: "App\\User")
}
```

削除してみます。

```graphql
mutation {
  deleteUser(
    id: 51
  ) {
    id
    name
  }
}
```

`@delete` ディレクティブではNonNullな引数を指定してね、と怒られてしまいます。  
[![スクリーンショット 2018-12-09 9.44.34.png](https://qiita-image-store.s3.amazonaws.com/0/173370/7d51ff2c-187f-6bdb-813f-1bec2236ad12.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2F7d51ff2c-187f-6bdb-813f-1bec2236ad12.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=deed477a12293302d92718b36568d0cc)

メッセージに従って、戻り値を必須に変更して再実行します。

```diff
type Mutation {
    deleteUser(
-       id: ID @rules(apply: ["required"])
+       id: ID! @rules(apply: ["required"])
    ): User @delete(model: "App\\User")
}
```

無事削除出来ました！  
[![スクリーンショット 2018-12-09 9.47.06.png](https://qiita-image-store.s3.amazonaws.com/0/173370/48579765-87e0-c481-f25a-47aa5db02998.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2F48579765-87e0-c481-f25a-47aa5db02998.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9a0cae4b23f3dee8587d71bdc9e7ee63)

## クエリ、ミューテーションを追加する

使い倒すために、いくつかクエリ、ミューテーションを追加してみます。

## LIKE検索を使ったクエリ

定義はこうやって、

```graphql
type Query {
    usersByEmail(email: String @where(operator: "like")): [User!]!
        @paginate(type: "paginator" model: "App\\User")
}
```

こんな感じで実行します。

```graphql
query {
  usersByEmail(
    email: "%r%"
    count: 10
  ) {
    data {
      id
      email
    }
  }
}
```

`@where` ディレクティブ、公式の [masterブランチ](https://lighthouse-php.com/master/api-reference/directives.html) には記載されているのですが [ver2.6ブランチ](https://lighthouse-php.com/2.6/api-reference/directives.html) には記載されていないので少し戸惑いました。

## ID以外を指定した削除ミューテーション

[ソースコード](https://github.com/nuwave/lighthouse/blob/master/src/Schema/Directives/Fields/DeleteDirective.php#L46-L57) を見るとID以外を指定した `@delete` ディレクティブの実行は出来なそうなので、resolverを自作します。

まずはartisanでファイルを生成します。

```sh
$ php artisan lighthouse:mutation DeleteUsersByEmail
```

今回は

- 指定した文字列でEmailのLIKE検索を検索をし、該当したユーザーを削除する
- 戻り値に適当な値をセットする

を満たすresolverを作ってみました。

app/Http/GraphQL/Mutations/DeleteUsersByEmail.php

```php
<?php

namespace App\Http\GraphQL\Mutations;

use GraphQL\Type\Definition\ResolveInfo;
use Nuwave\Lighthouse\Support\Contracts\GraphQLContext;
use App\User;

class DeleteUsersByEmail
{
    /**
     * Return a value for the field.
     *
     * @param null $rootValue Usually contains the result returned from the parent field. In this case, it is always \`null\`.
     * @param array $args The arguments that were passed into the field.
     * @param GraphQLContext|null $context Arbitrary data that is shared between all fields of a single query.
     * @param ResolveInfo $resolveInfo Information about the query itself, such as the execution state, the field name, path to the field from the root, and more.
     *
     * @return mixed
     */
    public function handle(
        $rootValue,
        array $args,
        GraphQLContext $context = null,
        ResolveInfo $resolveInfo
    ) {
        $email = $args['email'];
        User::where('email', 'like', $email)
            ->delete();

        $res = 'ok';

        return compact('res');
    }
}
```

続いて、自作したresolverを使って定義します。

```graphql
type DummyResponse {
    res: String!
}

type Mutation {
    deleteUsersByEmail(
        email: String!  @rules(apply: ["required"])
    ): DummyResponse
        @field(resolver: "App\\Http\\GraphQL\\Mutations\\DeleteUsersByEmail@handle")
}
```

最後に、こんな感じで実行します。

```graphql
mutation {
  deleteUsersByEmail(
    email: "%r%"
  ) {
    res
  }
}
```

## 定義ファイルを複数に分割する

routes/graphql/schema.graphqlだけでやり過ごすのは厳しいのでTypeごとに分割してみます。

まずはroutes/graphql/schema.graphqlをこんな感じにします。

routes/graphql/schema.graphql

```graphql
scalar DateTime @scalar(class: "Nuwave\\Lighthouse\\Schema\\Types\\Scalars\\DateTime")

scalar Date @scalar(class: "Nuwave\\Lighthouse\\Schema\\Types\\Scalars\\Date")

# その他ファイルをextend typeする為にここに空のベースとなる定義を配置しておく
type Query
type Mutation

# import models/*.graphql
```

続いて、分割したファイルはこんな感じにします。

routes/graphql/models/user.graphql

```graphql
extend type Query {
    users: [User!]! @paginate(type: "paginator" model: "App\\User")
    user(id: ID @eq): User @find(model: "App\\User")
    usersByEmail(email: String @where(operator: "like")): [User!]!
        @paginate(type: "paginator" model: "App\\User")
}

extend type Mutation {
    createUser(
        name: String
            @rules(apply: ["required"])
        email: String
            @rules(apply: ["required",
                "email",
                "unique:users,email"]
            )
        password: String
            @bcrypt
            @rules(apply: ["required"])
    ): User
    @create(model: "App\\User")
    updateUser(
        id: ID
            @rules(apply: ["required"])
        name: String
        email: String
            @rules(apply: ["email"])
    ): User
        @update(model: "App\\User")
    deleteUser(
        id: ID! @rules(apply: ["required"])
    ): User
        @delete(model: "App\\User")
    deleteUsersByEmail(
        email: String!  @rules(apply: ["required"])
    ): DummyResponse
        @field(resolver: "App\\Http\\GraphQL\\Mutations\\DeleteUsersByEmail@handle")
}

type DummyResponse {
    res: String!
}

type User {
    id: ID!
    name: String!
    email: String!
    password: String!
    articles: [Article] @hasMany
    createdAt: DateTime! @rename(attribute: created_at)
    updatedAt: DateTime! @rename(attribute: updated_at)
}

type Article {
    id: ID!
    userId: Int! @rename(attribute: user_id)
    title: String!
    body: String!
    createdAt: DateTime! @rename(attribute: created_at)
    updatedAt: DateTime! @rename(attribute: updated_at)
}
```

`type Query` `type Mutation` 等は一度しか呼べないので、schema.graphql側で

```graphql
# その他ファイルをextend typeする為にここに空のベースとなる定義を配置しておく
type Query
type Mutation
```

空の定義をしておいて、分割したファイルでは常に `extend type Query` 等extendするようにしています。

## エラーメッセージをカスタマイズする

実際に利用する場合、エラーメッセージのカスタマイズが必要になると思うので、Userを更新する処理でやってみます。

```graphql
extend type Mutation
    updateUser(
        id: ID
            @rules(
                apply: ["required"],
                messages: { required: "idは必須です" }
            )
        name: String
        email: String
            @rules(apply: ["email"])
    ): User
        @update(model: "App\\User")
}
```

messageでなく、extensions.validation内にカスタマイズしたメッセージが入ってきます。  
[![スクリーンショット 2018-12-09 10.44.51.png](https://qiita-image-store.s3.amazonaws.com/0/173370/1bdac9f2-449f-6a9a-f11f-5895fd1ed42a.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2F1bdac9f2-449f-6a9a-f11f-5895fd1ed42a.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c696c5bba1e41660c29c2e980aa8cdf4)

## graphql-playgroundが動かなくなったときの対処法

ミスした記述のクエリを実行したりしていると、ふとgraphql-playgroundが動かなくなるときがあります。

結構詰まって悩んでTwitterでつぶやいたところ、助けて頂きました！

誰しも一度は遭遇すると思うのでお気をつけください。

## 【2018/12/20追記】Datetimeを使う際の注意点

デフォルトで定義されている

```graphql
scalar DateTime @scalar(class: "Nuwave\\Lighthouse\\Schema\\Types\\Scalars\\DateTime")
```

こちらのDatetimeですが、

```graphql
type User {
    id: ID!
    ...省略
    emailVerifiedAt: DateTime @rename(attribute: email_verified_at)
    createdAt: DateTime @rename(attribute: created_at)
    updatedAt: DateTime @rename(attribute: updated_at)
}
```

typeで利用する際には注意が必要です。  
$datesを指定しておかないとエラーを吐いてしまうので必ずModel側で$ datesを指定しておきましょう。  
(created\_at, updated\_atだけであれば明示的に付加しなくても良いですが他の項目もあるので全て付加しておいたほうが良いと思います)

app/User.php

```php
class User extends Authenticatable
{
    // 省略

    // $datesで指定しておく
    protected $dates = [
        'email_verified_at', 'created_at', 'updated_at'
    ];
}
```

## 【2018/12/27追記】複数のクエリをまとめて実行する

GraphQLの最大のメリットである `エンドポイントをまとめられる` について記載していなかったので追記します。  
いきなり答えですが、

```graphql
query multiQuerySample(
  $user1Id: ID
  $user2Id: ID
) {
  user1: user(id: $user1Id) {
    id
  }
  user2: user(id: $user2Id) {
    id
  }
}
```

こうすれば複数のクエリをまとめて実行できます。

play graoundで実行するとこんな感じになります。  
[![スクリーンショット 2018-12-27 15.55.17.png](https://qiita-image-store.s3.amazonaws.com/0/173370/e81d73a3-eabe-52c6-7111-bf1107964713.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2Fe81d73a3-eabe-52c6-7111-bf1107964713.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=63a73dc0406b3202110a020b85014c8c)

RestAPIを何度も叩く苦痛がこれで一気に解消出来ますね！

## 【2019/01/10追記】複数のデータを一括で登録する

複数件のデータを一括で登録したいケースもあるのかなと思います。  
この方法が正解かどうかはわかりませんが追記します。

## 最終的なゴール

今回は以下のようなクエリを実行してユーザーを複数件登録出来るようにしたいと思います。

```graphql
mutation {
    createUsers(users: [
        {name: "taro", email: "taro@example.com", password: "xxxxxxxx"},
        {name: "hanako", email: "hanako@example.com", password: "xxxxxxxx"}
    ])
    {
      res
    }
}
```

## 配列型のデータを受け取る

複数件登録するには配列型のデータをGraphQLに渡してあげる必要があると思いますが、おそらくはScalarを定義しないと取得出来ないと思います。(間違っていたらご指摘頂けますと幸いです)

ですので、まずは配列の型を設定する為の独自のスカラーを定義します。

CLIからスカラーを作成します。

```bash
$ php artisan lighthouse:scalar Users
```

実行すると、 `app/GraphQL/Scalars` 配下にUsers.phpというファイルが作成されます。

続いて中身を実装していきます。

まず、作成したてのファイルの中身がこちらです。

app/GraphQL/Scalars/Users.php

```php
<?php

namespace App\Http\GraphQL\Scalars;

use GraphQL\Language\AST\Node;
use GraphQL\Type\Definition\ScalarType;

/**
 * Read more about scalars here http://webonyx.github.io/graphql-php/type-system/scalar-types/
 */
class Users extends ScalarType
{
    /**
     * Serializes an internal value to include in a response.
     *
     * @param string $value
     * @return string
     */
    public function serialize($value)
    {
        // Assuming the internal representation of the value is always correct
        return $value;
       
        // TODO validate if it might be incorrect
    }
    
    /**
     * Parses an externally provided value (query variable) to use as an input
     *
     * @param mixed $value
     * @return mixed
     */
    public function parseValue($value)
    {
        // TODO implement validation
        
        return $value;
    }
    
    /**
     * Parses an externally provided literal value (hardcoded in GraphQL query) to use as an input.
     *
     * E.g.
     * {
     *   user(email: "user@example.com")
     * }
     *
     * @param Node $valueNode
     * @param array|null $variables
     *
     * @return mixed
     */
    public function parseLiteral($valueNode, array $variables = null)
    {
        // TODO implement validation
        
        return $valueNode->value;
    }
}
```

補足のコメントはありますがざっくりそれぞれの関数について説明しておきます。

- serialize
	- DBから取得する際に値を変換する為の関数
- parseValue
	- 引数を変数で受け取った値を検証、加工する為の関数
- parseLiteral
	- 引数を直接受け取った値(変数定義せずクエリに直接指定する場合)を検証、加工する為の関数

最初から作成されている `Nuwave\Lighthouse\Schema\Types\Scalars\Datetime` を見ると参考になるかもと思ったので念の為載せておきます。

vendor/nuwave/lighthouse/src/Schema/Types/Scalars/DateTime.php

```php
<?php

namespace Nuwave\Lighthouse\Schema\Types\Scalars;

use Carbon\Carbon;
use GraphQL\Error\Error;
use GraphQL\Utils\Utils;
use GraphQL\Type\Definition\ScalarType;
use GraphQL\Language\AST\StringValueNode;

class DateTime extends ScalarType
{
    public function serialize($value): string
    {
        return $value->toAtomString();
    }

    public function parseValue($value): Carbon
    {
        try {
            $dateTime = Carbon::createFromFormat(Carbon::DEFAULT_TO_STRING_FORMAT, $value);
        } catch (\Exception $e) {
            throw new Error(Utils::printSafeJson($e->getMessage()));
        }

        return $dateTime;
    }

    public function parseLiteral($valueNode, array $variables = null): Carbon
    {
        if (! $valueNode instanceof StringValueNode) {
            throw new Error('Query error: Can only parse strings got: '.$valueNode->kind, [$valueNode]);
        }

        try {
            $dateTime = Carbon::createFromFormat(Carbon::DEFAULT_TO_STRING_FORMAT, $valueNode->value);
        } catch (\Exception $e) {
            throw new Error(Utils::printSafeJson($e->getMessage()));
        }

        return $dateTime;
    }
}
```

では、実際に中身を実装していきます。  
インターネット上にあるサンプルでは、日付型、メールアドレス等プリミティブな値に何かしらの制御を加えるケースが多いのかなと思うのですが、今回は配列の為GraphQL内部の型を意識する必要がありました。  
いきなり最終的な結果ですが、今回はこんな感じにしてみました。

app/Http/GraphQL/Scalars/Users.php

```php
<?php

namespace App\Http\GraphQL\Scalars;

use GraphQL\Language\AST\Node;
use GraphQL\Type\Definition\ScalarType;
use GraphQL\Error\Error;
use GraphQL\Language\AST\ListValueNode;
use GraphQL\Language\AST\ObjectValueNode;
use GraphQL\Language\AST\ObjectFieldNode;

class Users extends ScalarType
{
    public function serialize($value)
    {
        return $value;
    }

    public function parseValue($value)
    {
        return $value;
    }

    public function parseLiteral($valueNode, array $variables = null)
    {
        // 今回はデータが配列であるかどうかだけをチェックしていますが、実際はもう少ししっかりチェックすることになると思います
        if (!$valueNode instanceof ListValueNode) {
            throw new Error('Query error: Can only parse List got: '.$valueNode->kind, [$valueNode]);
        }

        return $this->argValue($valueNode);
    }

    /**
     * GraphQLが持つ型から最終的な配列データを取得する
     * 
     * @param $arg mixed
     * @return array
     */
    private function argValue($arg)
    {
        if ($arg instanceof ListValueNode) {
            return collect($arg->values)->map(function ($node) {
                return $this->argValue($node);
            })->toArray();
        }

        if ($arg instanceof ObjectValueNode) {
            return collect($arg->fields)->mapWithKeys(function ($field) {
                return [$field->name->value => $this->argValue($field)];
            })->toArray();
        }

        if ($arg instanceof ObjectFieldNode) {
            return $this->argValue($arg->value);
        }

        return $arg->value;
    }
}
```

上記例を見てわかる通り、argValueの内容が複雑で結構面倒でした。。  
`Nuwave\Lighthouse\Support\Traits\HandlesDirectives` というTraitにあるargValueを参考に少し整形して作成しました。

ここまでで引数が受け取れるようになると思います。

## 複数件データを登録する為のミューテーションを作成する

こちらも自作のResolverを使わないでミューテーションを実装する方法はないのかなと思うのでミューテーションを作成します。  
作成手順は上記に記載してあるので細かいことは割愛して、

```bash
$ php artisan lighthouse:mutation CreateUsers
```

上記コマンドでファイルを作成して、以下のように実装してみました。

app/Http/GraphQL/Mutations/CreateUsers.php

```php
<?php

namespace App\Http\GraphQL\Mutations;

use GraphQL\Type\Definition\ResolveInfo;
use Nuwave\Lighthouse\Support\Contracts\GraphQLContext;
use App\User;

class CreateUsers
{
    public function resolve($rootValue, array $args, GraphQLContext $context = null, ResolveInfo $resolveInfo)
    {
        User::insert($args['users']);

        $res = 'ok';
        return compact('res');
    }
}
```

ミューテーションはシンプルにこれだけでOKです。

最後に実際のミューテーションの定義を記載すれば完成です。

routes/graphql/models/user.graphql

```graphql
scalar Users @scalar(class: "App\\Http\\GraphQL\\Scalars\\Users")

extend type Mutation {
    createUsers(
        users: Users
    ): DummyResponse
    @field(resolver: "App\\Http\\GraphQL\\Mutations\\CreateUsers@resolve")
}
```

以上で実装は完了したので実際に実行してみます！

[![スクリーンショット 2019-01-10 17.00.45.png](https://qiita-image-store.s3.amazonaws.com/0/173370/aa9bf247-84e5-7dcf-3585-71974db8908b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2Faa9bf247-84e5-7dcf-3585-71974db8908b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ac877e3ea1273773023a6708c50910da)

無事複数件データを登録することが出来ました！

## 【2019/01/28追記】Order Byをする

早く公式で実装してほしいところですが現状order byをするには自作でdirectiveを作る(or resolverで対処)する必要があると思います。  
ディレクティブ追加方法含めてやっていきます。

## ゴール

こんな感じのクエリでソートを掛けられるようにしてみます。

```graphql
query {
  users(
    count: 3
    queryOrder: {
      column: "id",
      order: DESC
    }
  ) {
      data {
      id
    }
  }
}
```

## カスタムディレクティブを作成する

まずはカスタムディレクティブを作成します。

今までQueryやMutationはartisanコマンドから生成していましたが、Directiveは現状まだ実装されていないようです。  
ドキュメントは見当たらなかったんですが、config/lighthouse.phpを見ると、

app/config/lighthouse.php

```php
/*
    |--------------------------------------------------------------------------
    | Directives
    |--------------------------------------------------------------------------
    |
    | List directories that will be scanned for custom server-side directives.
    |
    */
    'directives' => [__DIR__.'/../app/Http/GraphQL/Directives'],
```

とあるので、指定されているディレクトリに作成します。  
`@eq` のもとになっている `Nuwave\Lighthouse\Schema\Directives\Args\EqualsFilterDirective` をコピペしていきます。

見てみるとこんな感じになっています。

vendor/nuwave/lighthouse/src/Schema/Directives/Args/EqualsFilterDirective.php

```php
\`\`\`<?php

namespace Nuwave\Lighthouse\Schema\Directives\Args;

use Nuwave\Lighthouse\Schema\Values\ArgumentValue;
use Nuwave\Lighthouse\Schema\Directives\BaseDirective;
use Nuwave\Lighthouse\Support\Contracts\ArgMiddleware;
use Nuwave\Lighthouse\Support\Traits\HandlesQueryFilter;

class EqualsFilterDirective extends BaseDirective implements ArgMiddleware
{
    use HandlesQueryFilter;

    /**
     * Name of the directive.
     *
     * @return string
     */
    public function name(): string
    {
        return 'eq';
    }

    /**
     * Resolve the field directive.
     *
     * @param ArgumentValue $argument
     * @param \Closure       $next
     *
     * @return ArgumentValue
     */
    public function handleArgument(ArgumentValue $argument, \Closure $next): ArgumentValue
    {
        $this->injectFilter(
            $argument,
            function ($query, string $columnName, $value) {
                return $query->where($columnName, $value);
            }
        );

        return $next($argument);
    }
}
```

↑の `injectFilter` だけ変えてあげれば行けそう？って予想がつくと思います。  
ので、それっぽく作ってみます。

app/Http/GraphQL/Directives/OrderByDirective.php

```php
<?php

namespace App\Http\GraphQL\Directives;

use Nuwave\Lighthouse\Schema\Values\ArgumentValue;
use Nuwave\Lighthouse\Schema\Directives\BaseDirective;
use Nuwave\Lighthouse\Support\Contracts\ArgMiddleware;
use Nuwave\Lighthouse\Support\Traits\HandlesQueryFilter;

class OrderByDirective extends BaseDirective implements ArgMiddleware
{
    use HandlesQueryFilter;

    /**
     * Name of the directive.
     *
     * @return string
     */
    public function name()
    {
        return 'orderBy';
    }

    /**
     * Resolve the field directive.
     *
     * @param ArgumentValue $argument
     * @param \Closure       $next
     *
     * @return ArgumentValue
     */
    public function handleArgument(ArgumentValue $argument, \Closure $next)
    {
        $this->injectFilter(
            $argument,
            function ($query, string $columnName, $value) {
                // ここでOrder Byをする
                return $query->orderBy($value['column'], array_key_exists('order', $value) ? $value['order'] : 'asc');
            }
        );

        return $next($argument);
    }
}
```

ここまででディレクティブ側の対応は完了です。

## 引数を受け取る

今回は `@eq` と同様に `ArgMiddleware` インターフェースを利用して引数で受け取るようにします。  
今回は単一項目でのOrder byを対象としますが、それでも `カラム` `昇順/降順` の2つが必要になります。  
そこで、2つの値を受け取れるよう設定します。

routes/graphql/models/user.graphql

```diff
- users: [User!]! @paginate(type: "paginator" model: "App\\User")
+ users(queryOrder: QueryOrder @orderBy): [User!]! @paginate(type: "paginator" model: "App\\User")
+ 
+ input QueryOrder {
+     column: String!
+     order: Order = ASC
+ }
+ 
+ enum Order {
+     ASC @enum(value: "ASC")
+     DESC @enum(value: "DESC")
+ }
```

これで引数として受け取れるようになりましたので実行してみます！

きちんとソート順を変えた状態でデータ取得出来ました！！！

[![スクリーンショット 2019-01-28 18.45.38.png](https://qiita-image-store.s3.amazonaws.com/0/173370/60a46497-5bd6-633a-5257-12cdc1bbd7fa.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2F60a46497-5bd6-633a-5257-12cdc1bbd7fa.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2018e82881778e711a4d15a44fe6403a)

## 【2019/02/07追記】ID型の注意点

僕も先日初めて知ったのですが、IDは [こちら](https://facebook.github.io/graphql/April2016/#sec-ID) にある通り、 `String` です。  
Laravel標準のidはIntですので、サンプルに惑わされずidにはIntを設定しておきましょう。

## さいごに

使い倒すと言っておきながら大して使い倒さぬままさいごになってしまいました😵

今回、LaravelのGraphQLライブラリの中では一番活発そうなLighouthouseを使ってみましたが、それでもまだまだ枯れてはいないなという印象を受けました。  
困った時は [このあたり](https://github.com/nuwave/lighthouse/tree/master/src/Schema/Directives) のソースコードを見ることで解決出来ると思います。  
解決出来ないならresolverを作成する方向で良いと思いますが、公式サイトに

[![スクリーンショット 2018-12-09 10.48.03.png](https://qiita-image-store.s3.amazonaws.com/0/173370/a9200273-7932-205a-2cb0-e37cc7e323a7.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F173370%2Fa9200273-7932-205a-2cb0-e37cc7e323a7.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8e82c23843dac11d4e1cb262c36d1678)

こうある通り、自らも一緒にOSSを育てていくくらいの感覚でいると面白みが出てくるかなと思います！(という自分もまだPRを上げられてませんが😣)

RestfulライクなAPIの代替案になりえるかという話、触る前は

- まとめて操作出来るのでAPI発行回数が少なくて済むってメリットがあるから複数のAPIを叩かなきゃいけないところだけGraphQL使って、それ以外は今まで通りRestfulライクなAPIを叩く感じになるかぁ

と思っていましが、いざ触ってみると

- SwaggerDocのように鬼のようなコードを書かずにドキュメントが生成出来る
- DevToolsも使いやすい

という感動があったので、認証系のAPIを除いてGraphQLに統合してしまうのもアリかなと思いました。

まだCodeジェネレーター周りを触っていないのですが、codegenが整備されてくるとTypescriptとの相性も良くなって一気にRestfulAPIの代替案としての可能性が出てくるのかも？と考えています。  
[こちら](https://qiita.com/ja_rascal/items/cebc0829f7407f6f6eda) の記事等に記載されているので、私もどこかのタイミングで触れてみたいと思います。

Lighouthouse自体はまだまだ枯れていないので成長の余地が大いにあると思いますが、Slackワークスペースに自由に参加出来ますし、PRも大歓迎なようなので少しでも成長に寄与出来たら良いなぁと少しだけ思ったりしています🙇

私自身、まだまだGraphQL初心者の域ですので、当記事の内容に対して何かありましたらぜひコメント頂けますと幸いです🙇🙇🙇！！！

[2](https://qiita.com/TsukasaGR/items/#comments)

[136](https://qiita.com/TsukasaGR/items/1a8b0020e5e83e7a46c7/likers)

92

ストックを更新しました