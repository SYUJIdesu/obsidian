---
title: 5年間 Laravel を使って辿り着いた，全然頑張らない「なんちゃってクリーンアーキテクチャ」という落としどころ
source: https://zenn.dev/mpyw/articles/ce7d09eb6d8117#%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89%E5%90%8D%E3%81%8C%E8%A1%9D%E7%AA%81%E3%81%99%E3%82%8B
author:
  - "[[Zenn]]"
published: 2020-12-24
created: 2025-05-15
description: 
tags:
  - web
  - Laravel
---
1156

23[tech](https://zenn.dev/tech-or-idea)

この記事は [Laravel Advent Calendar 2020 - Qiita](https://qiita.com/advent-calendar/2020/laravel) 最終日の記事です。

## TL;DR

- DDD や "真の" クリーンアーキテクチャは， Web 業界における大抵の現場ではオーバースペックだし，導入しても全員がついてこれるとは限らない
- `app/UseCases` ディレクトリだけ切って，ドメインごとに単一責務なクラスを置くと使いやすいよ
- ActiveRecord 指向のフレームワークで Repository パターンを無理に導入すると死ぬので， UseCase で Eloquent Model の機能を使うことを恐れるな

## はじめに

Zenn では初投稿です。日本の Laravel コミュニティではもうお馴染みのようで実はあまり顔を出していない（？） [@mpyw](https://github.com/mpyw) と申します。オンラインサロンの火付け役となった [Synapse](https://synapseam.github.io/) が最初の仕事でしたが，就職後すぐ会社が DMM に買収されて人員ごと移動することになりました。そこから会社を行ったり来たり彷徨っている時期がありましたが，最終的に現在は株式会社ゆめみで PHP テックリードとして勤務しています。

![Yumemi](https://res.cloudinary.com/zenn/image/fetch/s--lE8d-V5n--/https://www.yumemi.co.jp/images/logo_yumemi_02.svg)

5年ほど Laravel を使った結果，「結局 Laravel ってどう使えばいいんだ？」という自身の疑問と，ずばりそれに対する絶妙な落としどころとして， **「なんちゃってクリーンアーキテクチャ」** についてまとめます。クリーンアーキテクチャ要素は少しだけ取り込むけど，全然ガチでやらないよ，ぐらいの温度感のものです。

例を考えるにあたって， Laravel を JSON API アプリケーションとして使う前提で話を進めます。 `User` `Community` `Post` という3つのエンティティが存在し，ユーザがコミュニティのタイムラインに投稿するようなアプリケーションを考えます。コミュニティは CLI から管理者のみ作成可能であるとします。

## Laravel はここから始まる

[laravel/installer](https://github.com/laravel/installer) を使ってインストールし， `artisan` コマンドにクラス生成を任せる場合，典型的には `app` ディレクトリは以下のような形になると思います。（ `Models` ディレクトリが Laravel 8.x から復活したことは記憶に新しいですね）

```
app/
├─┬ Console/
│ ├─┬ Commands/
│ │ └── CommunityCommand.php
│ └── Kernel.php
├─┬ Exceptions/
│ └── Handler.php
├─┬ Http/
│ ├─┬ Controllers/
│ │ ├── UserController.php
│ │ ├── CommunityController.php
│ │ ├── PostController.php
│ │ └── Controller.php
│ ├── Kernel.php
│ └── Middleware/
├─┬ Models/
│ ├── User.php
│ ├── Community.php
│ └── Post.php
└── Providers/
```

- `User` `Community` `Post` にリレーションメソッドを実装し， `@property` アノテーションでカラム情報を書く。
- `UserController` `CommunityController` `PostController` に CRUD のうち必要なものや，コミュニティへの入退会処理などを実装。ロジックは `User` `Community` `Post` モデルを利用してコントローラに直接書く。
- `CommunityCommand` に CRUD のうち必要なものを実装。ロジックは `Community` モデルを利用してコマンドに直接書く。

ドメインロジックへの入り口として

- **HTTP** による JSON API へのアクセス
- **CLI** による `artisan` コマンド実行

の２通りがあり，それを介してドメインロジックが実行される流れです。例として， `PostController::store()` メソッドとしてありがちな実装を示してみます。

app/Http/Controllers/PostController.php

```php
class PostContoller
{
    /**
     * POST communities/{community}/posts 
     *
     * というエンドポイントで， {community} に指定された ID でルートモデルバインディング
     */
    public function store(Request $request, Community $community): Post
    {
        // 認可
    $userBelongsToCommunity = $community
        ->users()
        ->wherePivot('user_id', $request->user()->id)
        ->exists();
    if (!$userBelongsToCommunity) {
        throw new AccessDeniedHttpException('ユーザは指定されたコミュニティに所属していません');
    }
    
        // フォーマットバリデーション
    // （失敗時に ValidationException がスローされ，ハンドラによって UnprocessableEntityHttpException に変換される）
        $validator = Validator::make($request->all(), [
            'title' => 'required|string|max:30',
            'body' => 'required|string|max:10000',
        ]);
    $validator->validate();
    
    // ドメインバリデーション
    $userPostsCountToday = $request
        ->user()
        ->posts()
        ->where('community_id', $community->id)
        ->where('created_at', '>=', Carbon::midnight())
        ->count();
    if ($userPostsCountToday >= 200) {
        throw new TooManyRequestsHttpException(null, '本日の投稿可能な回数を超えました。');
    }
    
    // ドメインロジック
        $post = new Post($validator->validated());
    $post->user()->associate($request->user());
    $post->community()->associate($community);
    $post->save();
    
    // レスポンス返却
    return $post;
    }
}
```

…さて，こんなコードを書いていてはすぐコントローラが肥大化して破綻します。コントローラやコマンドにオールインワンで書くのを脱するための第一歩として， Laravel の機能を使ったリファクタリングを行いましょう。

## Laravel の機能を使ってコード分割

この章での最終形は以下のようになります。

```
app/
├─┬ Console/
│ ├── Commands/
│ └── Kernel.php
├─┬ Exceptions/
│ ├─┬ Post/
│ │ └── PostLimitExceededException.php
│ └── Handler.php
├─┬ Http/
│ ├─┬ Requests/
│ │ ├─┬ User/
│ │ │ ├── StoreRequest.php
│ │ │ └── UpdateRequest.php
│ │ └─┬ Post/
│ │   ├── IndexRequest.php
│ │   ├── StoreRequest.php
│ │   └── UpdateRequest.php
│ ├─┬ Resources/
│ │ ├── UserResource.php
│ │ ├── CommunityResource.php
│ │ └── PostResource.php
│ ├─┬ Controllers/
│ │ ├── UserController.php
│ │ ├── CommunityController.php
│ │ ├── PostController.php
│ │ └── Controller.php
│ ├── Kernel.php
│ └── Middleware/
├─┬ Models/
│ ├── User.php
│ ├── Community.php
│ └── Post.php
└── Providers/
```

## ポリシーで認可処理を切り出す

まず先頭にある認可処理を切り出してみます。 Laravel の **ポリシー** という機能を使うと， Eloquent Model に対する操作単位でメソッドに切り出して，判定ロジックをコントローラの外に出すことができます。

app/Policies/PostPolicy.php

```php
class PostPolicy
{
    use HandlesAuthorizations;

    public function store(User $user, Community $community): Response
    {
    $userBelongsToCommunity = $community
        ->users()
        ->wherePivot('user_id', $user->id)
        ->exists();

    return $userBelongsToCommunity
        ? $this->allow()
        : $this->deny('ユーザは指定されたコミュニティに所属していません');
    }
}
```

app/Http/Controllers/PostController.php

app/Http/Controllers/PostController.php

```php
use App\Http\Controllers\Controller;

class PostContoller extends Controller;
{
    public function store(Request $request, Community $community): Post
    {
        // 認可 ← 1行になった！
    $this->authorize('store', [Post::class, $community]);
    
        $validator = Validator::make($request->all(), [
            'title' => 'required|string|max:30',
            'body' => 'required|string|max:10000',
        ]);
    $validator->validate();
    
    $userPostsCountToday = $request
        ->user()
        ->posts()
        ->where('community_id', $community->id)
        ->where('created_at', '>=', Carbon::midnight())
        ->count();
    if ($userPostsCountToday >= 200) {
        throw new TooManyRequestsHttpException(null, '本日の投稿可能な回数を超えました。');
    }
    
        $post = new Post($validator->validated());
    $post->user()->associate($request->user());
    $post->community()->associate($community);
    $post->save();
    
    return $post;
    }
}
```

認可として典型的な権限判定処理はドメインロジックのノイズになりやすいので，ポリシーとして切り出すことによって **「結局何がしたいのか」** を明確にできるメリットがあります。

しかし，認可は後述の「ドメインバリデーション」というものと紙一重で，区別が非常に難しいものです。認可に分類すべき典型的な特徴としては，以下のようなものがあります。

- データベースアクセスが無い，あるいは軽量なクエリ
- ユーザのステータスや所属状態に依存した判定
- ドメインロジックに含めるにはあまりにも自明すぎる判定

これらも常に当てはまるものではなく，結局はケースバイケースというところです。

## フォーマットバリデーションを FormRequest に切り出す

次にバリデータを使って検証しているフォーマットバリデーションを **フォームリクエスト** という機能を使って切り出してみましょう。

app/Http/Requests/Post/StoreRequest.php

```php
class StoreRequest extends FormRequest
{
    public function authorize(Gate $gate): bool
    {
        // 認可処理もここで行うことができる
        $community = $this->route()->parameter('community');
    return $gate->authorize('store', [Post::class, $community]);
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|max:30',
            'body' => 'required|string|max:10000',
        ];
    }

    public function makePost(): Post
    {
        // バリデーションした値で埋めた Post を取得
        return new Post($this->validated());
    }
}
```

app/Http/Controllers/PostController.php

app/Http/Controllers/PostController.php

```php
class PostContoller
{
    public function store(StoreRequest $request, Community $community): Post
    {
        // 認可 + フォーマットバリデーション + 埋める処理
    $post = $request->makePost();
    
    $userPostsCountToday = $request
        ->user()
        ->posts()
        ->where('community_id', $community->id)
        ->where('created_at', '>=', Carbon::midnight())
        ->count();
    if ($userPostsCountToday >= 200) {
        throw new TooManyRequestsHttpException(null, '本日の投稿可能な回数を超えました。');
    }
    
    $post->user()->associate($this->request->user());
    $post->community()->associate($community);
    $post->save();
    
    return $post;
    }
}
```

認可処理とバリデーションを外出しした結果，だいぶスッキリしてきました。認可処理をフォームリクエストに含むべきかどうかについては賛否あると思いますが，個人的には **フォーマットバリデーションよりも前に認可処理が行われるべき** だと考えているため，ここに含めることを推奨しています。

最後のほうにある `FormRequest::makePost()` というメソッドの書き方に関しては，見慣れない方も多いかもしれませんが，コントローラ側での処理を隠蔽するために書いています。

## レスポンス整形処理を Resource に切り出す

飛んで，一番最後に書いてある Eloquent Model をリターンしている部分に注目してください。

```php
return $post;
```

Eloquent Model は `Jsonable` `Arrayable` 等を実装しているため，コントローラでリターンするだけで整形処理をフレームワークに丸投げすることが出来ます。但し，逆に言えば **HTTP がデータベースレイヤに強く依存した実装** となっており，柔軟性を大きく損なうものです。

これを解決するために， Laravel 5.5 から **API リソース** という機能が導入されました。

- [Eloquent: APIリソース 8.x Laravel](https://readouble.com/laravel/8.x/ja/eloquent-resources.html)

リソースクラスでラップすることで，柔軟に整形を行うことができます。また， **API インタフェースを分かりやすくすることにも繋がる** ため，可能な限り導入しておくことをおすすめします。

app/Http/Resources/PostResource.php

```php
class PostResource extends JsonResource
{
    public function toArray($request): array
    {
        // API 仕様がひと目で分かる！
        return [
        'id' => $this->resource->id,
        'title' => $this->resource->title,
        'body' => $this->resource->body,
        'created_at' => $this->resource->created_at,
        'updated_at' => $this->resource->updated_at, 
    ];
    }
}
```

app/Http/Controllers/PostController.php

app/Http/Controllers/PostController.php

```php
class PostContoller
{
    public function store(StoreRequest $request, Community $community): PostResource
    {
    $post = $request->makePost();
    
    $userPostsCountToday = $post
        ->user
        ->posts()
        ->where('community_id', $community->id)
        ->where('created_at', '>=', Carbon::midnight())
        ->count();
    if ($userPostsCountToday >= 200) {
        throw new TooManyRequestsHttpException(null, '本日の投稿可能な回数を超えました。');
    }
    
    $post->user()->associate($this->request->user());
    $post->community()->associate($community);
    $post->save();
    
    // ラップする！
    return new PostResource($post);
    }
}
```

さて，フレームワークとして導線を敷いてくれているのはここまでです。ここから先は，人によって大きく構成が変わってくる部分になります。

## モデルにドメインロジックを集約

ここでのドメインロジックとは，ドメインバリデーションを含みます。 Eloquent Model のような ActiveRecord パターンを採用しているフレームワークにおいて，コントローラからロジックが移植される先として挙げられやすいのはもちろんモデルです。まずここに移植してみましょう。同時に， HTTP に依存している例外があるため， HTTP 非依存の例外を定義してモデルで使用し，これをコントローラで変換することにします。

app/Http/Controllers/PostController.php

```php
class PostContoller
{
    public function store(StoreRequest $request, Community $community): PostResource
    {
    $post = $request->makePost();

    try {
        // ドメインバリデーションを呼び出す
            return new PostResource($post->storeAfterDomainValidation($user, $community));
    } catch (PostLimitExceededException $e) {
        // 捕まえた例外はスタックトレースに積む
        throw new TooManyRequestsHttpException(null, $e->getMessage(), $e);
    }
    }
}
```

app/Exceptions/Post/PostLimitExcededException.php

```php
class PostLimitExceededException extends Exception
{
}
```

app/Models/Post.php

```php
class Post extends Model
{
    public function storeAfterDomainValidation(User $user, Community $community): self
    {
    $userPostsCountToday = $user
        ->posts()
        ->where('community_id', $community->id)
        ->where('created_at', '>=', Carbon::midnight())
        ->count();
    if ($userPostsCountToday >= 200) {
        throw new PostLimitExceededException('本日の投稿可能な回数を超えました。');
    }
    
    $this->user()->associate($user);
    $this->community()->associate($community);
    $this->save();

    return $this;
    }
}
```

`storeAfterDomainValidation` という名前をつけて雑に移植してみました。一見これで十分に思えますが，実はモデルに移植することによって問題は山積みになります。

## メソッド名が衝突する

ActiveRecord パターンの宿命ですが，モデルのインスタンスがさまざまなアクションの起点になるため，モデルには無数のメソッドが生えています。更に Laravel が再利用性に乏しいメソッド分割でも行う傾向にあるため，すべてのメソッドの実装は誰も把握できないレベルです。

ここでは `storeAfterDomainValidation` という名前にしましたが，単に `store` とした場合はどうでしょう？これもまだギリギリ大丈夫です。では `delete` `update` などの場合は…？

そうです。 **モデルがもともと持っているメソッドと衝突してしまう** のです。Rails 全盛期の頃には「ファットコントローラつらい。ファットモデル最高！」と叫ばれている時期もありましたが，少なくとも Laravel でこれをやると常にメソッド命名に悩み続けなければなりません。

## 実装が肥大化する

コントローラからモデルに移して喜んだのも束の間。結局モデルが太ってしまっては意味がありません。

Rails 由来の文化ですが，苦肉の策として `Concerns` ディレクトリを切ってトレイトとして括りだす策があるでしょう。これも一見コードがスッキリする気はするのですが， １ファイルが読みやすくなっただけで， **複数のファイル間に共有するプロパティやメソッドの依存関係が分散してしまう** 大きな欠点があります。結果的にそれで失敗しているのが Laravel の基底 Eloquent Model クラス自身です。

- [database/Model.php at master · illuminate/database](https://github.com/illuminate/database/blob/3aea683cf1066bbeaeaf54f80eb3a11bfb27aee8/Eloquent/Model.php#L25-L32)

これは反面教師にして，真似しないようにしましょう。

## 例外クラスを置く場所がモデルから遠くなる

ドメインロジックに関する例外なのに，ドメインでまとめるディレクトリが存在しないため，仕方なく `app/Exceptions` あたりに置くしか無くなります。

## サービスクラスにドメインロジックを集約

モデルの代案として，「とりあえずサービスクラスを切ろうよ」という声も聞きます。 `app/Services` ディレクトリを作り，この中にドメインの関心を集めてみましょう。

```
app/
├─┬ Console/
│ ├── Commands/
│ └── Kernel.php
├─┬ Exceptions/
│ └── Handler.php
├─┬ Http/
│ ├─┬ Requests/
│ │ ├─┬ User/
│ │ │ ├── StoreRequest.php
│ │ │ └── UpdateRequest.php
│ │ └─┬ Post/
│ │   ├── IndexRequest.php
│ │   ├── StoreRequest.php
│ │   └── UpdateRequest.php
│ ├─┬ Resources/
│ │ ├── UserResource.php
│ │ ├── CommunityResource.php
│ │ └── PostResource.php
│ ├─┬ Controllers/
│ │ ├── UserController.php
│ │ ├── CommunityController.php
│ │ ├── PostController.php
│ │ └── Controller.php
│ ├── Kernel.php
│ └── Middleware/
├─┬ Models
│ ├── User.php
│ ├── Community.php
│ └── Post.php
├── Providers/
└─┬ Services/
  ├─┬ User/
  │ └── UserService.php
  ├─┬ Community/
  │ └── CommunityService.php
  └─┬ Post/
    ├── PostService.php
    └─┬ Exceptions/
      └── PostLimitExceededException.php
```

例として，投稿の作成に関する部分だけ見てみましょう。

app/Http/Controllers/PostController.php

app/Http/Controllers/PostController.php

```php
class PostContoller
{
    public function store(StoreRequest $request, PostService $service): PostResource
    {
    $post = $request->makePost();

    try {
        // ドメインバリデーションを呼び出す
            return new PostResource($service->store($user, $community, $post));
    } catch (PostLimitExceededException $e) {
        // 捕まえた例外はスタックトレースに積む
        throw new TooManyRequestsHttpException(null, $e->getMessage(), $e);
    }
    }
}
```

app/Services/Post/PostService.php

```php
class PostService
{
    public function store(User $user, Community $community, Post $post): Post
    {
        // バグを防ぐために簡易的にアサーションを書く
        assert($user->exists);
        assert($community->exists);
        assert(!$post->exists);

    $userPostsCountToday = $user
        ->posts()
        ->where('community_id', $community->id)
        ->where('created_at', '>=', Carbon::midnight())
        ->count();
    if ($userPostsCountToday >= 200) {
        throw new PostLimitExceededException('本日の投稿可能な回数を超えました。');
    }
    
    $post->save();
    return $post;
    }
}
```

これにより，以下のようなメリットが得られるようになりました！

- モデルからロジックが完全に追い出された
- コントローラも HTTP + Service + Resource のオーケストレーションをしているだけでシンプル
- **ドメインロジックを HTTP に依存しない形で分離することができた**

HTTP に依存しないことによって， HTTP だけでなく CLI からでも使い回せる可能性が見えてきました。但し，またここで問題があります。

## サービスクラスの肥大化

ここまで

「コントローラ → モデル → サービス」

と流れてきただけで，１クラスに全部詰め込んでいては本質的解決になっていません。この問題を解決するために，これ以降は **1アクション 1クラス** という思想を取り込みます。クリーンアーキテクチャで **「ユースケース」** と呼ばれている概念に近いです。

## 実はサービスには2種類ある

ここでは `app/Services` 直下にドメインロジックを扱うクラスの名前空間を並べました。ここでもし

「AWS S3 に画像をアップロードするためのラッパー的なクライアントが欲しい。画像のリサイズなどもいい感じにやってほしい。」

と言われたらどうしますか？ `app/Services/Aws` とか `app/Services/Image` とか `app/Services/Uploader` とか切りますか？ドメインロジックであるものと， AWS 固有のロジックが伴うものを果たして同じ階層に置いて良いのでしょうか？

私は，サービスという言葉で分離する場合，以下の2つ（ないし3つ）の種類が存在すると考えています。

| 種別 | 説明 |
| --- | --- |
| **ドメイン** サービス | ビジネスで実現したいことに関して，中核的なロジックを担うサービス。 |
| **インフラ** サービス | ネットワークやファイルストレージへのアクセスを担うサービス。 |
| アプリケーションサービス | HTTP アクセスを捌くサービス。要するにルーティング+コントローラ+コントローラに近い雑多なロジックもろもろ。 |

アプリケーションサービスに関しては， Laravel の通常のディレクトリ構成においては **`app` よりも上の `src` ディレクトリ配下すべて** に相当すると考えられるので，今回の文脈における分類では実質2種類であると言ってもいいかもしれません。またちょうどこの考え方は，次章で説明するクリーンアーキテクチャにも通じる部分があります。

## クリーンアーキテクチャをもし Laravel に厳格に適用したら？

![Clean Architecture](https://storage.googleapis.com/zenn-user-upload/s4ynb39huwkppx189nibez98o2go)  
*インターネッツでよく見かける図*

ようやく本題のクリーンアーキテクチャというところですが，この記事では厳格なクリーンアーキテクチャを **「たいていの Web 業界の現場ではオーバースペックすぎるよ」** という趣旨で反面教師とさせていただきます。実装内容について具体的に触れると記事が肥大化しますので詳細は割愛します。背景知識に乏しい方は，下記のような記事を参考にしてください。

- [Laravelで実践クリーンアーキテクチャ - Qiita](https://qiita.com/nrslib/items/aa49d10dd2bcb3110f22)

実装パターンは無数にありますが，記事の例に倣ってここでは以下のような構成を示してみます。今までは `app/` 配下ですべて管理していましたが，この例ではフレームワークの枠組みの外に置いて別パッケージ化していますね。

```
packages/
├─┬ Domain/
│ ├─┬ User/
│ | ├── User.php
│ | └── UserRepositoryInterface.php
│ ├─┬ Community/
│ | ├── Community.php
│ | └── CommunityRepositoryInterface.php
│ └─┬ Post/
│   ├── Post.php
│   ├── PostRepositoryInterface.php
│   └─┬ Exceptions/
│     └── PostLimitExceededException.php
├─┬ UseCases/
│ ├─┬ User/
│ │ ├── StoreAction.php
│ │ └── UpdateAction.php
│ ├─┬ Community/
│ │ ├── StoreAction.php
│ │ └── UpdateAction.php
│ └─┬ Post/
│   ├── IndexAction.php
│   ├── StoreAction.php
│   ├── UpdateAction.php
│   └── DestroyAction.php
└─┬ Infrastructure/
  ├─┬ EloquentModels/
  │ ├── User.php
  │ ├── Community.php
  │ └── Post.php
  └─┬ Repositories/
    ├── UserRepository.php
    ├── CommunityRepository.php
    └── PostRepository.php

app/
├─┬ Console/
│ ├── Commands/
│ └── Kernel.php
├─┬ Exceptions/
│ └── Handler.php
├─┬ Http/
│ ├─┬ Requests/
│ │ ├─┬ User/
│ │ │ ├── StoreRequest.php
│ │ │ └── UpdateRequest.php
│ │ └─┬ Post/
│ │   ├── IndexRequest.php
│ │   ├── StoreRequest.php
│ │   └── UpdateRequest.php
│ ├─┬ Resources/
│ │ ├── UserResource.php
│ │ ├── CommunityResource.php
│ │ └── PostResource.php
│ ├─┬ Controllers/
│ │ ├── UserController.php
│ │ ├── CommunityController.php
│ │ ├── PostController.php
│ │ └── Controller.php
│ ├── Kernel.php
│ └── Middleware/
└── Providers/
```

さて， `app/` に存在していた Eloquent Model と Service が外部に飛び出した形になりました。 `packages/` の中のファイル数がエグいことになっていますが，これでもまだかなり控えめなほうです。厳格に実施することがいかに大変か，これだけでも伝わりますね。

大雑把にディレクトリの用途を確認しましょう。

| ディレクトリ | 用途 |
| --- | --- |
| packages/Domain | **「ビジネスに出てくる登場人物とその設定事項」** |
| packages/UseCase | Domain を用いて **「ビジネスで実現したいこと」** を表現したもの |
| packages/Infrastructure | UseCase に **「どうやって実現するか」** として具体性を与えるもの |
| app | フレームワーク上で UseCase に Infrastructure を注入して，ユーザに価値を提供する |

さらに，この中に出てくる何種類かのクラスについて，その役割を確認しておきましょう。

| クラス | 用途 |
| --- | --- |
| packages/Domain/Post/Post | 投稿を意味する抽象的なエンティティ |
| packages/Domain/Post/PostRepositoryInterface | `Post` の取得や保存の呼び出し方だけを定める |
| packages/UseCases/Post/StoreAction | ドメイン層の `Post` と `PostRepositoryInterface` を使って保存する手順を定める |
| packages/Infrastructure/EloquentModels/Post | `posts` テーブルの操作が可能な Laravel の Eloquent Model |
| packages/Infrastructure/Repositories/PostRepository | `PostRepositoryInterface` を実装し， Eloquent Model を使ってデータベースを操作する具体的なメソッドを定義する |

これらの依存関係を図にまとめると，以下のようになります。

![Laravel におけるクリーンアーキテクチャの依存関係](https://storage.googleapis.com/zenn-user-upload/hvt3rwrdt015ob2gok5z4c5snuch)  
*Laravel におけるクリーンアーキテクチャの依存関係*

ではこの実装の問題点を挙げていきましょう。

## ファイル数が多い

今回示した例はクリーンアーキテクチャと呼べるほぼ最小限の実装ですが，これでかったるいと感じる人に厳格に適用することは向いていないでしょう。

## Eloquent Model の機能を使うことが出来ない

Eloquent Model を Repository が返してしまえば，例えば UseCase で `->save()` を実行することができますが， UseCase は外側にある Infrastructure に依存してはならないため，原理主義的なクリーンアーキテクチャにおいてこの設計は許されません。

そのため， **ActiveRecord 由来の機能を持たない Domain Model (Entity) に詰め替えた上で返し** ，また何か別の処理が必要なときはもう一度 Eloquent Model に詰めてクエリを実行する，という面倒な責務を Repository が責任を持って担当しなければなりません。

クリーンアーキテクチャとしては当たり前ですが，普段の Laravel のコードに慣れている人からすると見慣れない書き方だと感じられてしまうかもしれないし，教育が徹底されていないとルールを破った実装をする人が出てくるかもしれません。

## Eloquent Builder の機能を使うことが出来ない

上記と似ていますが，こちらはクエリビルディングに関する機能を指します。

Laravel の Eloquent Builder は `->where()` やスコープメソッドなどを連ねて柔軟に条件を組み立てることができますが，同じように UseCase でこの機能を使うことができなくなり，結果的に **Repository に条件の組み立て方 1 つごとに 1 メソッド用意** することで， Repository クラスが肥大化してしまうことが懸念されます。

これを回避するための Criteria パターンというものもありますが， **Eloquent のスコープ機能と目的が完全に被ります** 。無理に Laravel に導入するとなると， **「スコープ機能は使わない」** あるいは **「Eloquent を完全に捨てて Query Builder だけで縛る」** など厳しい制約が必要になってきます。

仮にそんなコードがあるとしましょう。何故あなたは Laravel を使うんですか？

「Laravel だと書ける人が多くて採用がラク」

が理由の1つになるかもしれませんが，標準から逸脱した書き方をした場合，新入りの人に責任を持ってあなたが全てを教える必要があります。

## UseCase だけを取り入れた， Laravel 向けに妥協したクリーンアーキテクチャ

Laravel にクリーンアーキテクチャを持ち込む上で，最も障壁になっている ActiveRecord 実装としての Eloquent。ぶっちゃけ Eloquent は設計が腐っているけど，それを承知でクリーンアーキテクチャを使いたいんだ！ということで，最終回答として示したいのがこの実装。

- [Laravel で Request, UseCase, Resource を使いコントロールフローをシンプルにする - Qiita](https://qiita.com/nunulk/items/5297cce4545ac3c16822)
- [Controller, UseCase, Service (および Model) の役割分担についての考察 - Qiita](https://qiita.com/nunulk/items/bc7c93a3dfb43b01dfab)

上記リンクを参考にして実務に導入してみましたが， **学習コストに対してコードの治安を保つ効果が十分に高い** と感じられたため，「迷ったらこれを使っておけ」とオススメしたいアーキテクチャです。 Web 系ベンチャー案件なら全然これで対応できるんじゃないかと思います。「サービスにドメインロジックを集約」の段階からの変化としては，

**「ドメインサービスを単一責務にしてユースケースとして扱う」**

と表現することができます。

クリーンアーキテクチャにおいて本来なら禁忌とされる形です。実態として「MVC + Service だよね」という声もありましたが，単一責務な Service は UseCase と呼ぶほうがしっくり来るので，これもクリーンアーキテクチャの亜種の1つとして存在を認めてもいいかなと考えています。

```
app/
├─┬ Console/
│ ├── Commands/
│ └── Kernel.php
├─┬ Exceptions/
│ └── Handler.php
├─┬ Http/
│ ├─┬ Requests/
│ │ ├─┬ User/
│ │ │ ├── StoreRequest.php
│ │ │ └── UpdateRequest.php
│ │ └─┬ Post/
│ │   ├── IndexRequest.php
│ │   ├── StoreRequest.php
│ │   └── UpdateRequest.php
│ ├─┬ Resources/
│ │ ├── UserResource.php
│ │ ├── CommunityResource.php
│ │ └── PostResource.php
│ ├─┬ Controllers/
│ │ ├── UserController.php
│ │ ├── CommunityController.php
│ │ ├── PostController.php
│ │ └── Controller.php
│ ├── Kernel.php
│ └── Middleware/
├─┬ Models
│ ├── User.php
│ ├── Community.php
│ └── Post.php
├── Providers/
└─┬ UseCases/
  ├─┬ User/
  │ ├── StoreAction.php
  │ └── UpdateAction.php
  ├─┬ Community/
  │ ├── StoreAction.php
  │ └── UpdateAction.php
  └─┬ Post/
    ├── IndexAction.php
    ├── StoreAction.php
    ├── UpdateAction.php
    ├── DestroyAction.php
    └─┬ Exceptions/
      └── PostLimitExceededException.php
```

app/Http/Controllers/PostController.php

```php
class PostContoller
{
    public function store(StoreRequest $request, StoreAction $action): PostResource
    {
    $post = $request->makePost();

    try {
        // ドメインバリデーションを呼び出す
            return new PostResource($action($user, $community, $post));
    } catch (PostLimitExceededException $e) {
        // 捕まえた例外はスタックトレースに積む
        throw new TooManyRequestsHttpException(null, $e->getMessage(), $e);
    }
    }
}
```

app/UseCases/Post/StoreAction.php

```php
class StoreAction
{
    public function __invoke(User $user, Community $community, Post $post): Post
    {
        // バグを防ぐために簡易的にアサーションを書く
        assert($user->exists);
        assert($community->exists);
        assert(!$post->exists);

    $userPostsCountToday = $user
        ->posts()
        ->where('community_id', $community->id)
        ->where('created_at', '>=', Carbon::midnight())
        ->count();
    if ($userPostsCountToday >= 200) {
        throw new PostLimitExceededException('本日の投稿可能な回数を超えました。');
    }
    
    $post->save();
    return $post;
    }
}
```

通常のクリーンアーキテクチャの実装が，この亜種にどう対応するか比較してみましょう。

| 通常 | 亜種 |
| --- | --- |
| packages/Domain/Post/Post | app/Models/Post |
| packages/Domain/Post/PostRepositoryInterface | app/Models/Post |
| packages/UseCases/Post/StoreAction | app/UseCases/Post/StoreAction |
| packages/Infrastructure/EloquentModels/Post | app/Models/Post |
| packages/Infrastructure/Repositories/PostRepository | app/Models/Post |

はい！たった2クラスしかありません。 以下の変更が起こっています。

- Eloquent Model と Domain Model (Entity) の区別をやめました
- Eloquent Builder と Repository の区別をやめました
- データベースへの依存を抽象化するのをやめました
- アプリケーションが直接 UseCase を持つようになりました

これにより， Domain Model や Repository に関する記述はすべて Eloquent Model に統合され， UseCase だけが残る形になりました。

![Eloquent を認めたクリーンアーキテクチャ](https://storage.googleapis.com/zenn-user-upload/nu9y70g1x8ps7ncxw22ib54p7d7m)  
*Eloquent を認めたクリーンアーキテクチャ*

## テストどうするの？

Repository 至上主義の方から真っ先に突っ込まれるのはこの質問だと思います。答えとしては， **「ユニットテストは諦めて機能テストを書け」** となります。 `RefreshDatabase` トレイト万歳。

ユニットテストは純粋なロジック以外はすべてモックされている必要がありますが， UseCase が Eloquent Model を使う以上は厳しいので，諦めて機能テストに分類させましょう。 **但し完全に全てモックを諦めるのではなく，あくまでデータベースアクセスに限定する話です。** 外部 API の呼び出し，イベントのディスパッチ処理などは引き続きモックした上でテストを書くといいと思います。

一応過去にこの辺をどうにかしようと頑張ったことはありますが，学習コストや保守性での費用対効果があまりにも悪かったので諦めました。 [^1] [^2]

## データベース変わったらどうするの？

Web 業界の一般的な案件で，「アプリケーションのコードはそのままで RDBMS を入れ替えたい」という事例をほとんど聞いたことがありません。こちらに関しては 99% 杞憂だと考えても良いでしょう。

## フレームワーク変わったらどうするの？

データベースよりは変わる可能性はありますが，それでもビジネスサイドからすれば「品質に問題なく動いているもののアーキテクチャを無理に変えなくていい」とされる場合がほとんどだと思います。このアーキテクチャで品質をしっかり保つことができていれば，そうそうそんな機会はないでしょう。

もしそれがあるとすれば，もっと大掛かりに言語ごと選択を変えてゼロベースでリプレイスするときです。例えば，「PHP から Golang に置き換えてマイクロサービスにするぞ」といった案件は至るところで耳にします。

## テーブルに対応しないエンティティが出てきたらどうするの？

**該当する UseCase と同じ階層に置きましょう。**

- 「 `app/UseCases` には Controller などから呼ばれる UseCase 以外置いてはいけない」という決まりは無いので，自由に置きましょう。個人的には「Domain のために UseCase が存在する」ではなく **UseCase の抽象的な共通処理をまとめるために Domain が存在する** と考えており， UseCase によるディレクトリ分割を軸にして Domain を添えるという形はむしろあるべき姿だと考えています。
- 但し，上記の UseCase を司令塔 **（Transaction Script）** と考えるアーキテクチャはいわゆる **「ドメインモデル貧血症」** を引き起こしてしまう可能性があります。似たような処理が多くの UseCase に分散してしまってモデリングが足りていないように感じた場合，上の注意書きに述べたように， **Eloquent Model を内包する Domain Model を作成しても構いません** 。 **最初から UseCase 内でデータベースアクセスの副作用を抽象化することは諦めているためです** 。この際， Domain Model のコンストラクタが複数の Eloquent Model や，それに付随する何らかの付加情報を一緒に受け取っても良いため，応用範囲は非常に広いと考えられます。
😢 UseCase 向けトレイトによる Transaction Script 的な方法

```php
// UseCase 向けトレイトによる Transaction Script 的な方法
// いわゆる「ドメインモデル貧血症」の疑い

trait DetectsUsersDailyPostLimit
{
    public function userExceedsDailyPostLimit(User $user, Community $community): bool
    {
        $userPostsCountToday = $user
        ->posts()
        ->where('community_id', $community->id)
        ->where('created_at', '>=', Carbon::midnight())
        ->count();

    return $userPostsCountToday >= 200;
    }
}

// UseCase
public function __invoke(User $user, Community $community): void
{
    $exceeded = $this->userExceedsDailyPostLimit($user, $community);
    // ...
}
```

😢 Eloquent Model 向けトレイト+インタフェースによる方法

```php
// Eloquent Model 向けトレイト+インタフェースによる方法
// Eloquent Model に引っ張られるため自由度が低い

/**
 * @mixin User
 */
trait HasDailyPostLimitTrait
{
    public function exceedsDailyPostLimit(Community $community): bool
    {
        $userPostsCountToday = $this
        ->posts()
        ->where('community_id', $community->id)
        ->where('created_at', '>=', Carbon::midnight())
        ->count();

    return $userPostsCountToday >= 200;
    }
}
interface HasDailyPostLimit
{
    public function exceedsDailyPostLimit(Community $community): bool;
}

// UseCase
public function __invoke(HasDailyPostLimit $user, Community $community): void
{
    $exceeded = $user->exceedsDailyPostLimit($community);
    // ...
}
```

😊 Domain Model が Eloquent Model をラップする方法

```php
// Domain Model が Eloquent Model をラップする
// OOP を正しく実装している感じがする
// ($user と $community を一緒に持ち回りたいというニーズに応える雑な例)
class UserInCommunity
{
    public function __construct(
        protected User $user,
    protected Community $community,
    ) {
    }
    
    public function exceedsDailyPostLimit(): bool
    {
        $userPostsCountToday = $this
        ->user
        ->posts()
        ->where('community_id', $this->community->id)
        ->where('created_at', '>=', Carbon::midnight())
        ->count();

    return $userPostsCountToday >= 200;
    }
}

// UseCase
public function __invoke(UserInCommunity $userInCommunity): void
{
    $exceeded = $userInCommunity->exceedsDailyPostLimit();
    // ...
}
```

## まとめ

TL;DR の繰り返しになりますが

> - DDD や "真の" クリーンアーキテクチャは， Web 業界における大抵の現場ではオーバースペックだし，導入しても全員がついてこれるとは限らない
> - `app/UseCases` ディレクトリだけ切って，ドメインごとに単一責務なクラスを置くと使いやすいよ
> - ActiveRecord 指向のフレームワークで Repository パターンを無理に導入すると死ぬので， UseCase で Eloquent Model の機能を使うことを恐れるな

## 余談

## UseCase のクエリスナップショットテスト

Eloquent の副作用を許す UseCase パターンに対しては，クエリスナップショットテストが特に有用です。十分に書きやすく，安全性を維持する効果が高いです。

1. `DB::enableQueryLog()` でログ記録開始
2. UseCase を実行
3. `DB::getQueryLog()` で実行された SQL と付随するバインドパラメータを検証 **（クエリスナップショットテスト）**
4. 起こった副作用を検証

といっても，そのままでは些か可読性が下がるので，以下のようなユーティリティを用意するといいでしょう。ほぼそのまま使えるように `use` 宣言も記載しておきます。

tests/SqlFormatter.php

tests/SqlFormatter.php

```php
<?php

namespace Tests;

use Illuminate\Support\Str;
use NilPortugues\Sql\QueryFormatter\Formatter;

class SqlFormatter
{
    /**
     * @param  string[] $sqls
     * @return string
     */
    public static function concatFormattedSqls(array $sqls): string
    {
        $results = [];
        foreach ($sqls as $offset => $sql) {
            $results[] = implode("\n", [
                "Logged SQL ($offset)",
                '',
                static::formatSql($sql),
            ]);
        }
        return implode("\n------------\n", $results);
    }

    /**
     * @param  string $sql
     * @return string
     */
    public static function formatSql(string $sql): string
    {
        return (new Formatter())->format($sql);
    }

    /**
     * @param  string $sql
     * @param  array  $bindings
     * @return string
     */
    public static function replacePlaceholders(string $sql, array $bindings): string
    {
        return Str::replaceArray(
            '?',
            array_map(fn ($v) => static::replacePlaceholderValue($v), $bindings),
            $sql
        );
    }

    /**
     * @param  mixed  $value
     * @return string
     */
    protected static function replacePlaceholderValue($value): string
    {
        if ($value === null) {
            return 'null';
        }

        return sprintf("'%s'", addcslashes((string)$value, "\\'"));
    }
}
```

tests/AssertsSql.php

tests/AssertsSql.php

```php
<?php

namespace Tests;

use Illuminate\Database\Eloquent\Builder as EloquentBuiler;
use Illuminate\Database\Eloquent\Relations\Relation;
use Illuminate\Database\Query\Builder as QueryBuilder;
use Illuminate\Support\Facades\DB;
use PHPUnit\Framework\Assert as PHPUnit;

trait AssertsSql
{
    /**
     * @param string[]    $expected
     * @param null|string $connection
     * @param string      $message
     */
    public static function assertLoggedSqlsEqual(array $expected, ?string $connection = null, string $message = ''): void
    {
        $actual = array_map(
            fn (array $entry) => SqlFormatter::replacePlaceholders($entry['query'], $entry['bindings']),
            DB::connection($connection)->getQueryLog()
        );

        PHPUnit::assertSame(
            SqlFormatter::concatFormattedSqls($expected),
            SqlFormatter::concatFormattedSqls($actual),
            $message ?: 'Failed asserting that SQLs are equivalent.'
        );
    }

    /**
     * @param  string                                                                                                                           $expected
     * @param  \Illuminate\Database\Eloquent\Builder|\Illuminate\Database\Eloquent\Relations\Relation|\Illuminate\Database\Query\Builder|string $actual
     * @param  string                                                                                                                           $message
     * @throws \PHPUnit\Framework\AssertionFailedError
     */
    public static function assertSqlEquals(string $expected, $actual, string $message = ''): void
    {
        if ($actual instanceof Relation) {
            $actual = $actual->getQuery();
        }
        if ($actual instanceof EloquentBuiler) {
            $actual = (clone $actual)->toBase();
        }
        if ($actual instanceof QueryBuilder) {
            static::assertSqlWithBindingEquals(
                $expected,
                (clone $actual)->toSql(),
                (clone $actual)->getBindings()
            );
            return;
        }

        assert(is_string($actual));
        PHPUnit::assertSame(
            SqlFormatter::formatSql($expected),
            SqlFormatter::formatSql($actual),
            $message ?: 'Failed asserting that SQLs are equivalent.'
        );
    }

    /**
     * @param  string                                  $expected
     * @param  string                                  $actualSql
     * @param  array                                   $actualBindings
     * @param  string                                  $message
     * @throws \PHPUnit\Framework\AssertionFailedError
     */
    public static function assertSqlWithBindingEquals(string $expected, string $actualSql, array $actualBindings, string $message = ''): void
    {
        static::assertSqlEquals(
            $expected,
            SqlFormatter::replacePlaceholders($actualSql, $actualBindings),
            $message
        );
    }
}
```

- `NilPortugues\Sql\QueryFormatter\Formatter` が SQL 文字列をある程度正規化してくれるので，人間が読みやすいように改行を自由に入れても問題有りません。
- クエリパラメータも，文字列と整数に関しては， SQL 文字列中に埋め込めるようになっています。

活用例

```php
DB::enableQueryLog();

$post = Post::where('title', 'タイトル')->first();

$this->assertLoggedSqlsEqual([
    <<<EOD
        select
            *
        from
            \`posts\`
        where
            \`posts\`.\`title\` = 'タイトル'
        and \`posts\`.\`deleted_at\` is null
    limit
        1
    EOD
]);

$this->assertSame('タイトル', $post->title);
```

## UseCase 以外のテストをどう書くか

### FormRequest

UseCase のテストは必須，次点でここもテストすることが望ましい部分になります。基本的にユニットテストの扱いで書くことが可能ですが，少し工夫が必要です。やや書き方に癖があるので，改善案を探しています。

FormRequest のテスト

tests/Unit/Http/Requests/Post/StorRequestTest.php

```php
class StoreRequestTest extends TestCase
{
    protected function setUp(): void
    {
        parent::setUp();

        // Gate インタフェースをモックして認可処理をスキップする
        $this->app->instance(Gate::class, $gate = Mockery::mock(Gate::class));
        $gate->shouldReceive('authorize->allowed')->andReturnTrue();
    }

    protected function createRequest(array $params, ?Authenticatable $user = null): StoreRequest
    {
        // 「コンテナ」「リダイレクタ」が無いと FormRequest が動かないためセット
    // （このあたりは Mockerey でモック化しても OK）
        return StoreRequest::create('', 'POST', $params)
            ->setContainer($this->app)
            ->setRedirector($this->app[Redirector::class])
            ->setUserResolver(fn () => $user);
    }

    public function testPassing(): void
    {
        $request = $this->createRequest([
            'title' => 'タイトル',
            'body' => '本文',
        ]);
    
    // FormRequest が解決されたときと同じ処理を実行
        $request->validateResolved();

        // fill() されたものがリクエストパラメータと一致するか検証
        $post = $request->makePost();
        $this->assertEquals([
            'title' => 'タイトル',
            'body' => '本文',
        ], $post->getDirty());
    }

    public function testMissingBody(): void
    {
        $request = $this->createRequest([
            'title' => 'タイトル',
        'body' => '',
        ]);

        try {
        // FormRequest が解決されたときと同じ処理を実行
            $request->validateResolved();
        
        // ValidationException が飛ぶはずなのでここには来ない
            $this->assertTrue(false);
        } catch (ValidationException $e) {
        // body のチェックに引っかかっていることを簡易的に検証
            $this->assertEqualsCanonicalizing(
                ['body'],
                $e->validator->getMessageBag()->keys()
            );
        }
    }
}
```

### API Resource

API リソースはほとんどテストを書く必要がないレベルですが，一応書いておくなら例えば以下のようになるでしょう。リクエストに依存しない場合は `Request::create('')` として雑にインスタンスを渡しておけばいいです。

もちろん，モデルのフィールドが出力の際に変化する場合はそれを踏まえたテストを書きましょう。

API Resource のテスト

tests/Unit/Http/Resources/PostResourceTest.php

```php
class PostResourceTest extends TestCase
{
    public function testToArray(): void
    {
        $attributes = [
            'id' => 123,
            'title' => 'タイトル',
        'body' => '本文',
            'created_at' => '2020-01-01 00:00:00',
            'updated_at' => '2020-01-01 00:00:00',
        ];

        $post = (new Post())->setRawAttributes($attributes, true);
    
    // 入出力が等しいことを検証
    $expected = $attributes;
        $actual = (new PostResource($post))->toArray(Request::create(''));

        $this->assertEquals($expected, $actual);
    }
}
```

### Controller

コントローラに関しては，ユニットテストはほぼ不要で， Laravel の標準的な疑似リクエストを行うテストのみで十分です。実際， IDE や PHPStan での静的解析が通っていたらほとんどテストは要らないかと思いますが，アプリケーションの末端で「本当に壊れていないこと」を保証したい場合には書く意味があると思います。但し，大量に書いて条件分岐を全パターン網羅する必要はないでしょう。

## コンストラクタインジェクションか，メソッドインジェクションか

実は最初， UseCase の実装に関して，以下のうち後者のメソッドインジェクションを使った実装を行ってしまっていました。

コンストラクタインジェクションのコード

Constructor Injection

```php
class UseCase
{
    protected SomeInterface $dependency;

    public function __consturct(SomeInterface $dependency)
    {
        $this->dependency = $dependency;
    }

    public function __invoke(string $arg1, int $arg2)
    {
        return $this->dependency->doSomething($arg1, $arg2);
    }
}

class Controller
{
    public function doSomething(UseCase $useCase)
    {
        return $useCase('foo', 123);
    }
}
```

メソッドインジェクションのコード

Method Injection

```php
class UseCase
{
    protected string $arg1;
    protected int $arg2;

    public function __consturct(string $arg1, int $arg2)
    {
        $this->arg1 = $arg1;
        $this->arg2 = $arg2;
    }

    public function __invoke(SomeInterface $dependency)
    {
        return $dependency->doSomething($this->arg1, $this->arg2);
    }
}

class Controller
{
    public function doSomething(Container $container)
    {
        return $container->call(new UseCase('foo', 123));
    }
}
```

| 種別 | 依存性を受け取る場所 | パラメータを受け取る場所 |
| --- | --- | --- |
| コンストラクタインジェクション | `__construct()` | `__invoke()` |
| メソッドインジェクション | `__invoke()` | `__construct()` |

"UseCase" という言葉の響きがどうも Laravel の Job に近く感じられたので，それに寄せてこのような実装にしてしまっていました。ただコードを見て分かる通り，コンストラクタインジェクションのほうが遥かにたくさんのメリットがあります。

- DI コンテナの存在を意識しなくていい
- IDE が静的解析で返り値を認識することができる
- UseCase クラス名を直接記述しなくていい（インタフェースにしようと思えば出来る）

メソッドインジェクションにも「パラメータを保持したままシリアライズして遅延実行できる」というメリットはありますが， Laravel の Job などと違って UseCase を直接そういう使い方はしないしすべきではないので，今回はメリットとはなりません。

今後は可能な限りコンストラクタインジェクションを使おうかな，と考えております。ここは直近で自分が書いてしまったコードに関する反省点です。

脚注

[GitHubで編集を提案](https://github.com/mpyw/zenn/blob/master/articles/ce7d09eb6d8117.md)

1156

23

この記事に贈られたバッジ

![もっと読みたい](https://static.zenn.studio/images/badges/paid/badge-frame-20.svg) ![Thank you](https://static.zenn.studio/images/badges/paid/badge-frame-10.svg) ![Thank you](https://static.zenn.studio/images/badges/paid/badge-frame-5.svg) ![Thank you](https://static.zenn.studio/images/badges/paid/badge-frame-3.svg) ![Thank you](https://static.zenn.studio/images/badges/paid/badge-frame-3.svg) ![Thank you](https://static.zenn.studio/images/badges/paid/badge-frame-3.svg) ![参考になった](https://static.zenn.studio/images/badges/support-helpful.svg) ![参考になった](https://static.zenn.studio/images/badges/support-helpful.svg) ![参考になった](https://static.zenn.studio/images/badges/support-helpful.svg) ![参考になった](https://static.zenn.studio/images/badges/support-helpful.svg) ![参考になった](https://static.zenn.studio/images/badges/support-helpful.svg) ![参考になった](https://static.zenn.studio/images/badges/support-helpful.svg) ![Thank you](https://static.zenn.studio/images/badges/support-thanks.svg) ![Thank you](https://static.zenn.studio/images/badges/support-thanks.svg) ![参考になった](https://static.zenn.studio/images/badges/support-helpful.svg) ![参考になった](https://static.zenn.studio/images/badges/support-helpful.svg)

[^1]: [mpyw/mockery-pdo: \[Experimental\] BDD-style PDO Mocking Library for Mockery](https://github.com/mpyw/mockery-pdo)

[^2]: [mpyw/laravel-database-mock: \[Experimental\] Database Mocking Library which mocks PDO underlying Laravel Connection classes](https://github.com/mpyw/laravel-database-mock)