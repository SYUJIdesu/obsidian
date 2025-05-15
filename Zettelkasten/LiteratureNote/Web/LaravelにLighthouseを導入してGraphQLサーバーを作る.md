---
title: "LaravelにLighthouseを導入してGraphQLサーバーを作る"
source: "https://tech.fusic.co.jp/posts/2019-12-08-laravel-lighthouse-graphql-server/"
author:
published:
created: 2025-05-15
description: "この記事ではLaravelでGraphQLサーバーの開発はどんなんだろうという方向けにセットアップから開発の流れ、テストまでざっくり紹介したいと思います。"
tags:
  - "web"
  - "lighthouse"
---
![Author](https://d3t2k53lg8fjy7.cloudfront.net/public/fdb95e707b9319f0957b90be9c3ac3d6d4f3f273.jpeg) Daiki Urata

2019/12/08

## セットアップ

### 1\. インストール

```
$ composer require nuwave/lighthouse
```

### 2.デフォルトスキーマ生成

```
$ php artisan vendor:publish --provider="Nuwave\Lighthouse\LighthouseServiceProvider" --tag=schema
```

デフォルトで生成されるスキーマは以下のようになりました。

```
# graphql/schema.graphql

"A datetime string with format \`Y-m-d H:i:s\`, e.g. \`2018-01-01 13:00:00\`."

scalar DateTime @scalar(class: "Nuwave\\Lighthouse\\Schema\\Types\\Scalars\\DateTime")

"A date string with format \`Y-m-d\`, e.g. \`2011-05-23\`."

scalar Date @scalar(class: "Nuwave\\Lighthouse\\Schema\\Types\\Scalars\\Date")

type Query {

    users: [User!]! @paginate(defaultCount: 10)

    user(id: ID @eq): User @find

}

type User {

    id: ID!

    name: String!

    email: String!

    created_at: DateTime!

    updated_at: DateTime!

}
```

### 3\. IDEヘルパー導入

Lighthouseでは独自で定義されたディレクティブを多用するので、PhpStormやVSCodeで認識させるためのスキーマを生成します。

```
$ composer require --dev haydenpierce/class-finder

$ php artisan lighthouse:ide-helper
```

コマンド実行後は `schema-directives.graphql` というファイルが作成されます。

### 4\. GraphQL開発ツールの導入

このツールを導入するとGraphQLクエリを実行して確認しながら開発を進めることができます。

```
$ composer require mll-lab/laravel-graphql-playground
```

インストール完了後、ブラウザで `http://[アプリURL]/graphql-playground` へアクセスすると以下のような画面が出てくると思います。

![GraphQL Playground Preview](https://tech.fusic.co.jp/uploads/laravel-lighthouse-1.png "GraphQL Playground Preview")

実装したAPIは基本ここでデバッグしていくことになります。

右タブの「DOCS」を開くとすでに定義されいるスキーマ情報が確認できます。

そのためクライアントアプリ開発者はここの情報を見て開発を進めることができ、とても便利だと思います。

これでLighthouseのセットアップは一通り完了です。

## クエリを実行してみる

すでに生成されたデフォルトのスキーマではユーザー一覧が取得できる users とユーザー単体の情報が取得できる user というクエリが定義されています。

```
# graphql/schema.graphql

type Query {

    users: [User!]! @paginate(defaultCount: 10)

    user(id: ID @eq): User @find

}
```

開発ツールで表示されるスキーマ情報は以下のようになります。

![GraphQL Playground Schema](https://tech.fusic.co.jp/uploads/laravel-lighthouse-2.png "GraphQL Playground Schema")

Laravelのプロジェクト作成時にはUserテーブルのMigrationファイルはすでにあると思いますが、Seedファイルはないので自分で作成してください。

今回は省略しますが詳しくは [ドキュメント](https://tech.fusic.co.jp/posts/2019-12-08-laravel-lighthouse-graphql-server/laravel.com/docs/6.x/seeding) を見てください。

Userテーブルにデータがある前提で users クエリを先ほど導入した開発ツールで実行してみたいと思います。

以下のように実行してみました。

![Users Query Example](https://tech.fusic.co.jp/uploads/laravel-lighthouse-3.png "Users Query Example")

Lighthouseで提供している [@paginate](https://lighthouse-php.com/4.7/api-reference/directives.html#paginate) ディレクティブを追加することで firstとpageというパラメータを受け取ることができ、クエリ結果にもpaginatorInfoというページネーション情報が返ってくるようになりました。

このようにディレクティブを追加するだけでユーザー一覧を取得するビジネスロジック部分を書かなくていいのがLighthouseの特徴です。

## CRUDを実装する

それではLighthouseのディレクティブを使用してCRUDを実装してみます。

### 1\. Create

スキーマ:

@createディレクティブを使います。

```
# graphql/schema.graphql

# ...省略

type Mutation {

    createUser(name: String!, email: String!, password: String!): User @create

}
```

実行画面:

![Create User Mutation Example](https://tech.fusic.co.jp/uploads/laravel-lighthouse-4.png "Create User Mutation Example")

### 2\. Read

こちらはすでにデフォルトで定義されているので省略します。

@findや@paginateを使うことで実現できます。

リレーションに関しては @hasManyや@belongsToというディレクティブがあるのでそれらを使用します。

詳しくは [ドキュメント](https://lighthouse-php.com/4.7/eloquent/relationships.html) を見てください。

### 3\. Update

スキーマ:

@updateディレクティブを使います。

```
# graphql/schema.graphql

# ...省略

type Mutation {

    # ...省略

    updateUser(id: ID!, name: String): User @update

}
```

実行画面:

![Update User Mutation Example](https://tech.fusic.co.jp/uploads/laravel-lighthouse-5.png "Update User Mutation Example")

### 4\. Delete

スキーマ:

@deleteディレクティブを使います。

```
# graphql/schema.graphql

# ...省略

type Mutation {

    # ...省略

    deleteUser(id: ID!): User @delete

}
```

実行画面:

![Delete User Mutation Example](https://tech.fusic.co.jp/uploads/laravel-lighthouse-6.png "Delete User Mutation Example")

### おまけ Upsert

さらにv4.5.0から@upsertのディレクティブも追加されました。

```
# graphql/schema.graphql

# ...省略

type Mutation {

    # ...省略

    upsertUser(id: ID, name: String!, email: String): User @upsert

}
```

ここまで単純なCRUDを実装してみましたが、一切PHPを書かずにディレクティブだけで実現できました。

## スキーマを分割

ここまでデフォルトで生成されたschema.graphqlに全て記述してきましたが、開発が進んでくるとスキーマが膨らんできてファイルをモデルごとに分割したくなってきます。

分割方法は簡単で schema.graphqlに

```
#import **/*.graphql
```

を追加するだけOKです。

ディレクトリ構成は色々できますが、今回は以下のようにしました。

```
- graphql
    - schema.graphql
    - models
        - user.graphql
```

分割後のファイルは以下のようになります。

```
# graphql/schema.graphql

"A datetime string with format \`Y-m-d H:i:s\`, e.g. \`2018-01-01 13:00:00\`."

scalar DateTime @scalar(class: "Nuwave\\Lighthouse\\Schema\\Types\\Scalars\\DateTime")

"A date string with format \`Y-m-d\`, e.g. \`2011-05-23\`."

scalar Date @scalar(class: "Nuwave\\Lighthouse\\Schema\\Types\\Scalars\\Date")

type Query

type Mutation

#import **/*.graphql
```
```
# models/user.graphql

extend type Query {

    users: [User!]! @paginate(defaultCount: 10)

    user(id: ID @eq): User @find

}

extend type Mutation {

    createUser(name: String!, email: String!, password: String!): User @create

    updateUser(id: ID!, name: String): User @update

    deleteUser(id: ID!): User @delete

    upsertUser(id: ID, name: String!, email: String): User @upsert

}

type User {

    id: ID!

    name: String!

    email: String!

    created_at: DateTime!

    updated_at: DateTime!

}
```

このように分割することでモデルごとにスキーマ情報を整理することができます。

ちなみに自分の書いたスキーマの構文をチェックするコマンドがあります。

```
$ php artisan lighthouse:validate-schema
```

スキーマに間違いがある場合はエラーが出力されます。

## Resolverによるカスタムロジックの導入

単純なCRUDではLighthouseの提供するディレクティブで対応できました。

しかし複雑なロジックが関わってくるとどうしてもPHPを書きたい箇所が出てくると思います。

そんな時はResolverを定義することで解決できます。

コマンドが用意されてるので実行します。

```
$ php artisan lighthouse:query UserResolver
```

生成されたファイルが以下のようになります。

```
<?php

//  GraphQL/Queries/UserResolver.php

namespace App\GraphQL\Queries;

use GraphQL\Type\Definition\ResolveInfo;

use Nuwave\Lighthouse\Support\Contracts\GraphQLContext;

class UserResolver

{

    /**

     * Return a value for the field.

     *

     * @param  null  $rootValue Usually contains the result returned from the parent field. In this case, it is always \`null\`.

     * @param  mixed[]  $args The arguments that were passed into the field.

     * @param  \Nuwave\Lighthouse\Support\Contracts\GraphQLContext  $context Arbitrary data that is shared between all fields of a single query.

     * @param  \GraphQL\Type\Definition\ResolveInfo  $resolveInfo Information about the query itself, such as the execution state, the field name, path to the field from the root, and more.

     * @return mixed

     */

    public function __invoke($rootValue, array $args, GraphQLContext $context, ResolveInfo $resolveInfo)

    {

        // TODO implement the resolver

    }

}
```

デフォルトの関数名は\_\_invokeとなっていますが、これをfindに変更して以下のように書いてみます。

```
public function find($rootValue, array $args, GraphQLContext $context, ResolveInfo $resolveInfo)

{

    $user = \App\User::findOrFail($args['id']);

    return $user;

}
```

その後、スキーマを編集します。

```
# models/user.graphql

extend type Query {

    # ...省略

    user(id: ID): User @field(resolver: "App\\GraphQL\\Queries\\UserResolver@find")

}
```

このように @find、@eqディレクティブを利用していたものを@fieldに変更して先ほど用意したUserResolverを指定します。

実行してみると動きとしては同じですが、findの関数内で独自のロジックを組み込むことが可能となります。

## APIテスト

最後にGraphQLのAPIテストについて軽く説明したいと思います。

まずはセットアップです。

```
<?php

namespace Tests;

use Nuwave\Lighthouse\Testing\MakesGraphQLRequests; // 追加

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase

{

    use CreatesApplication;

    use MakesGraphQLRequests; // 追加

}
```

テストファイルを作成します。

```
$ php artisan make:test UserTest
```

試しにユーザー一覧を取得するクエリのテストを書いてみました。

```
<?php

// tests/Feature/UserTest.php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;

use Illuminate\Foundation\Testing\WithFaker;

use Tests\TestCase;

use App\User;

class UserTest extends TestCase

{

    use RefreshDatabase;

    /**

     * test users query

     *

     * @return void

     */

    public function testQueriesUsers()

    {

        $user = factory(User::class)->create();

        $this->graphQL('

            query GetUsers {

                users {

                    data {

                        id

                        name

                        email

                    }

                }

            }

        ')->assertJson([

            'data' => [

                'users' => [

                        "data" => [

                            [

                                'id'     => $user->id,

                                'name'     => $user->name,

                                'email' => $user->email,

                            ]

                        ]

                ]

            ]

        ]);

    }

}
```

## 最後に

ここまででLighthouseのセットアップから簡単なCRUDの実装、テストまで書いてみました。

複雑なサーバー処理がなければほとんどスキーマを書いていくだけでLaravel製のGraphQLサーバーができてしまうのはかなり魅力的だなと思います。

まだまだ紹介できないディレクティブがあるので興味のある方は [公式ドキュメント](https://lighthouse-php.com/4.7/api-reference/directives.html) を参照してみてください。

明日の記事は [@kozo](https://qiita.com/kozo) となります。お楽しみにー。

![Daiki Urata](https://d3t2k53lg8fjy7.cloudfront.net/public/fdb95e707b9319f0957b90be9c3ac3d6d4f3f273.jpeg)

## Daiki Urata

フロントエンド/モバイルアプリなどを主に開発しています。

## Related Posts

[LaravelとVueで翻訳ファイルを共通化する](https://tech.fusic.co.jp/posts/2023-05-09-shared-translation-files-laravel-vue) ![Daiki Urata](https://d3t2k53lg8fjy7.cloudfront.net/public/fdb95e707b9319f0957b90be9c3ac3d6d4f3f273.jpeg)

Daiki Urata

2023/05/09

---

[Laravel9 + Inertia.js + Vue3 + TypeScript環境を構築する](https://tech.fusic.co.jp/posts/2023-01-05-laravel-inertia-vue3-typescript) ![Daiki Urata](https://d3t2k53lg8fjy7.cloudfront.net/public/fdb95e707b9319f0957b90be9c3ac3d6d4f3f273.jpeg)

Daiki Urata

2023/01/05

---

[Stripe Payment Links Webhooks For PHPer](https://tech.fusic.co.jp/posts/2022-12-04-2022-12-04-stripe-payment-links-webhook) ![shiro seike / せいけ しろう / 清家 史郎](https://tech.fusic.co.jp/uploads/seike460.png)

shiro seike / せいけ しろう / 清家 史郎

2022/12/04

---

[始めよう！PHP AWS Lambda with Laravel！](https://tech.fusic.co.jp/posts/2022-12-01-php-lambda-laravel) ![shiro seike / せいけ しろう / 清家 史郎](https://tech.fusic.co.jp/uploads/seike460.png)

shiro seike / せいけ しろう / 清家 史郎

2022/12/01

---

[LaravelにReact + TypeScriptを導入する](https://tech.fusic.co.jp/posts/2022-08-02-laravel-vite-react-typescript) ![Daiki Urata](https://d3t2k53lg8fjy7.cloudfront.net/public/fdb95e707b9319f0957b90be9c3ac3d6d4f3f273.jpeg)

Daiki Urata

2022/08/02