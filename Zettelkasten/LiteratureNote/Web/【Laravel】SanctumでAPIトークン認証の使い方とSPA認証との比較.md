---
title: 【Laravel】SanctumでAPIトークン認証の使い方とSPA認証との比較
source: https://qiita.com/104dev/items/0787a81f7dda892ce86a
author:
  - "[[104dev]]"
published: 2023-06-22
created: 2025-05-22
description: Sanctumを使うとAPIトークン（BearerToken）を使った認証とLaravelのセッション管理を使ったSPA認証のパターンが実装できるが、今回は簡単なAPIトークンでの実装視点でまとめる。…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

Sanctumを使うとAPIトークン（BearerToken）を使った認証とLaravelのセッション管理を使ったSPA認証のパターンが実装できるが、今回は簡単なAPIトークンでの実装視点でまとめる。

## SanctumにおけるAPI認証の仕組み

以下はLaravel Sanctumで実装する場合の仕組みである。

\=====

まず、クライアントがログイン情報をサーバーに送信し、ログインに成功するとサーバーからトークンを返す（これはcreateTokenメソッドを使って手動で実装することになる）。

ログイン状態が継続中はクライアント側でトークンを保持し、リクエスト毎にHTTPヘッダに付与する。

リクエスト時には同ライブラリの組み込み機能である `sanctumガード` がAuthorizationヘッダのトークンをチェックし、そのトークンが正しい場合にリクエストを通す。

このときサーバー側でも認証トークンをデータベースに持っており、sanctumのAPIトークンはリクエスト毎にデータベースとの照合が発生するのでJwtだけでは認証が完結しないステートフルな性質を持っている。

サーバー側のトークンを削除すれば強制的にログアウトすることが可能で、この点は一般的なJwtのデメリットとして言われているサーバー側からのログアウトが困難な状況は克服している（ただしスケーラブルではないというデメリットもある）。

## 【メモ】Laravelにおけるガード（guard）とは

ガード（guard）とは一言で言うと認証方法のパターンを定義したものである。

ガードを構成する要素としてはドライバー（driver）とプロバイダー（provider）が存在し、ドライバーはログイン状態を管理する方法を、プロバイダーは認証自体を行う方法を定義する。

例えば、Laravelではデフォルトでwebという名前のguardが定義されている。

ドライバーの値にはsessionが設定されており、ログイン状態の保持にはCookieを使ったセッション管理機能が用いられている。

プロバイダーにはusersが指定され、さらにusersの具体的な内容がeloquentのUsersモデルとなっており、認証情報の照合はUserモデルを経由したusersテーブルで行っている。

7.x以前はapiガードの定義もデフォルトで用意されていたが、それらのガードを使って自分で実装せずにSanctumを使うことが推奨されているためか、8.x以降のソースからは消えている。

代わりにsanctumガードが導入され、デフォルトではwebガードを使ってログイン状態を判定し、webガードによる認証が通らなければapiトークンを参照する機能を持つ（この設定は変更することができる）。

## SPA認証との違い

LaravelにおいてのAPIトークン認証（以下API認証と呼ぶ）とSPA認証については、大まかに下記の違いがあると認識している。

### 認証方法の観点

大まかに言えばSPA認証はLaravelのセッション管理機能を使った認証、APIトークン認証はAuthorizationヘッダの情報を照合するという違いがある。

API認証はログイン時のJsonレスポンス内で受け取ったBearTokenを、認証が必要なクエストヘッダに付与する。

SPA認証はログイン時にXSRF-TOKENという名前で付与されたトークンを、リクエストヘッダにX-XSRF-TOKENの値として付与する。これがCSRF対策のトークンとしての役割を果たす。ログイン後はXSRF-TOKENとその他の更新されたセッションデータが自動的にCookieにセットされる。

API側ではsanctumガードのミドルウェアを使うことが一般的であり、これはリクエストを受けた際にまず、Laravel組み込みの認証機能であるwebガードでSession情報をチェックし、これにパスしなければAuthorizationヘッダを参照する仕組みとなっている。

### セキュリティの観点

- SPA認証でXSRF-TOKENをCookieで持つ
- API認証で返されたトークンをLocalStorageで持つ

個人的に調べた範囲ではLocalStorageの特性としてJavaScriptに対して無防備な点があるのと、CookieにはHttpOnly属性（これを指定するとJavaScriptから盗み取りができなくなる）があるので、情報認証情報（トークン）の直接的な流出は前者（SPA認証）の方が防げる認識。

また、そもそものSession機能としてsession\_idやその他セッションに関するデータもCookieとして付与され、リクエスト時にはXSRF-TOKENだけでなくこれら全ての整合性がサーバー側で取れる必要があるので、セッションハイジャックのリスクは低いと思われる。

ただし、XSSが起きえる状況（ユーザーがWebサイトを操作している最中に攻撃側が悪意のあるスクリプトの実行に成功した状況）においては、クッキーに直接アクセスできなくても画面上の情報（DOM操作で読み取れる情報）は盗まれるうるのでXSSのリスクを完全に拭いきることは不可能。

### 設定難易度の観点

SPA認証も難しくはないが、初学ならAPI認証の方が楽。

API認証はヘッダに付与されたトークンを照合するだけで良いのに対して、SPA認証は参照元ドメインの定義やCORSを考慮する必要があり純粋に設定項目が多い。また、セッション周りの知識ゼロの状態からでは学ぶ必要のある周辺知識も多い。

## 設定する必要があるもの/しなくて良いもの早見表

認証に関するパッケージが何もない状態から、最低限の認証まで行おうとしたら下記のようになるという認識。

| ファイル名 | API認証 | SPA認証 |
| --- | --- | --- |
| /config/auth.php | 必要 | 必要 |
| /config/cors.php | 不要 | 必要 |
| /config/sanctum.php | 不要 | 必要 |
| /.env | 不要 | 必要 |
| /app/Models/(認証モデル).php | 必要 | 必要 |
| /app/Http/Kernel.php | 不要 | 必要 |
| /app/AppServiceProvider | 不要 | 必要 |
| /database/migrations/create\_personal\_access\_tokens\_table.php | 必要 | 不要 |
| HasApiTokensトレイトの読み込み | 必要 | 不要 |

※HasApiTokensトレイトを読み込ませる必要はないという意味で、認証モデルに関する設定はもちろん必要。

## 最低限の実装手順

ここからは最低限APIトークン認証を実装する手順を記載する。

### 認証に使うテーブル関係

#### 認証テーブルのスキーマを作成

デフォルトでusersを認証テーブルとして使う想定でマイグレーション・モデルファイルが用意されているが、今回はaccountsを認証テーブルとして使用する。

database/migrations/create\_accounts\_table.php

```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('accounts', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('accounts');
    }
};
```

スキーマはusersテーブルのマイグレーションファイルを内容をそのまま貼り付けたもの。

#### モデルファイルの定義

```text
php artisan make:model Account
```

app\\Models\\Account

```php
<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class accounts extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
    ];
}
```

トークンを関連づけたい認証モデルにHasApiTokensトレイトを読み込ませる  
（今回は認証テーブルのモデルAccountに持たせた）。

ここで必ずしもAuthenticatableクラスを継承しているモデル（デフォルトではUserモデル）でHasApiTokensトレイトを読み込ませないといけないわけではない。

例えば、「組織（organizations）」と「メンバー（members）」についてそれぞれテーブルが存在するとする。ここでログイン認証自体はorganizationsテーブルで行い、認証トークンはmemberテーブルのレコードに関連づけて発行するということも可能。

試してはないが、personal\_access\_tokensテーブルの内容を見る限り、複数のテーブルにHasApiTokensトレイトを用いることができるのでないかと思われる。

#### トークンを保持するテーブルを作成

Laravelのプロジェクト作成時に生成される `personal_access_tokens` テーブルのマイグレーションファイルをそのまま使えば良い。

database/migrations/personal\_access\_tokens.php

```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('personal_access_tokens', function (Blueprint $table) {
            $table->id();
            $table->morphs('tokenable');
            $table->string('name');
            $table->string('token', 64)->unique();
            $table->text('abilities')->nullable();
            $table->timestamp('last_used_at')->nullable();
            $table->timestamp('expires_at')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('personal_access_tokens');
    }
};
```

認証テーブルとトークンを保持するテーブルのマイグレーションファイルができたら、migrationを実行。

```text
php artisan migrate
```

#### 初期データの投入

```bash
php artisan make:seeder AccountSeeder
```

/database/seeders/AccountSeeder.php

```php
class AccountSeeder extends Seeder
{
    public function run(): void
    {
        Account::create([
            'name' => 'testuser',
            'email' => 'test@examle.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

### 設定ファイルの修正

`config/auth.php` を編集する。

ここでは認証テーブルをusersからaccountsに変更したことに伴う変更を記載する。

プロバイダ名の書き換えは任意であるが、今回は分かりやすいように認証に使うテーブルaccountsに揃えた。

auth.php

```php
<?php

return [

    'defaults' => [
        'guard' => 'web',
        'passwords' => 'accounts',
    ],

    'guards' => [
        'web' => [
            'driver' => 'session',
            //'provider' => 'users',
            'provider' => 'accounts',
        ],
        //'api' => [
        //    'driver' => 'token',
            'provider' => 'accounts'
        //]
    ],

    'providers' => [
        /* 書き換え前 */
        /*
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\User::class,
        ],
        */

        /* 書き換え後 */
        'accounts' => [
            'driver' => 'eloquent',
            'model' => App\Models\Account::class,
        ],
    ],

    'passwords' => [
//        'users' => [
        'accounts' => [
            //'provider' => 'users',
            'provider' => 'accounts',
            'table' => 'password_reset_tokens',
            'expire' => 60,
            'throttle' => 60,
        ],
    ],

    'password_timeout' => 10800,

];
```

### ルーティング

APIトークン認証で実装する場合はSPA認証と異なってCookieやCSRFトークンは関係ないため、ログイン・ログアウトのエンドポイントも、認証中に使うエンドポイントも `/config/api.php` に定義したので良い。

今回はログインメソッドと、ログインしているユーザーの情報を返すメソッドを定義。

### コントローラー

ログインに成功した場合はHasApiTokenトレイトからcreateTokenメソッドを呼び出してトークンを返す。  
失敗した場合は401エラーを返す。

## APIアクセスしてトークンを取得してみる

localhostにAPIサーバーを立てて、Insomniaを使ってアクセスできるか試してみたい。

▼emailとパスワードを送信してログインを試みる。  
[![スクリーンショット 2023-06-21 9.36.21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2982811/8ff302ce-fbbf-3f13-d650-fbc593845bb0.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2982811%2F8ff302ce-fbbf-3f13-d650-fbc593845bb0.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=551a105cd57d2eafa9335fb4d1371b73)

▼ログインに成功するとトークン入りのレスポンスが返される。  
[![スクリーンショット 2023-06-21 9.36.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2982811/711b02b7-3067-d894-6014-5e1b94667951.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2982811%2F711b02b7-3067-d894-6014-5e1b94667951.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bab386a3e42e3ae0e325df9f2ee72432)

▼phpMyAdminでpersonal\_access\_tokensテーブルを見ると、レスポンスで返したトークンのペアがサーバー上にも生成されている。  
tokenable\_typeカラムにはそのトークンが関連づけられたモデルクラスが書かれており、tokenable\_idカラムと合わせてaccountsテーブルのID=1のレコードに関連づけられていることが読み取れる。  
[![スクリーンショット 2023-06-21 23.41.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2982811/89683b23-2c79-fbc3-d1ff-ebd6f3bfc643.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2982811%2F89683b23-2c79-fbc3-d1ff-ebd6f3bfc643.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=107c854c3d67a1977eefdb2203e673f3)

▼受け取ったトークンをAuthorizationタブでBearerTokenとして設定すると、認証が必要なリクエストに成功する。  
（サンプルプロジェクトなのでセキュリティ上は誰も困らないはずであるが、一応意識の問題としてトークン全文を晒さないようにXXXの文字列に置き換えさせて頂いている）  
[![スクリーンショット 2023-06-21 9.39.37.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2982811/dca981a4-d0b0-55fd-5fe8-7645b9efc458.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2982811%2Fdca981a4-d0b0-55fd-5fe8-7645b9efc458.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a86c74348c9196666f41cbbe486de063)

▼トークンがない状態でログインするとリクエストは通らない。  
※ここでは何も設定していないので/loginにリダイレクトしてしまっているが、本当は例外処理を施してステータスコード401とJsonでのエラーメッセージを返す必要がある。  
[![スクリーンショット 2023-06-22 0.03.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2982811/88ebe1d2-3da3-9c93-de0c-70fa4d117598.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2982811%2F88ebe1d2-3da3-9c93-de0c-70fa4d117598.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c9d64ed63b997a7f56bdaa0b222250a1)

▼ログアウトを実行  
[![スクリーンショット 2023-06-21 23.53.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2982811/4f1f0b57-243f-ff82-65d9-e98dcc4a6dd0.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2982811%2F4f1f0b57-243f-ff82-65d9-e98dcc4a6dd0.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=33b0ee5a8590519eb99590dffc3e206d)  
▼データベースを確認するとログアウトのリクエスト送信時に付与したトークンがデータベース上から物理削除され、クライアントで保持しているトークンではアクセスできず、再度ログインが必要となる。  
[![スクリーンショット 2023-06-21 23.54.20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2982811/58f94f01-70ae-323f-c293-a5497291bca3.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2982811%2F58f94f01-70ae-323f-c293-a5497291bca3.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=eee7073eb5f6c9bf9519818ad250ba50)

以上。

[1](https://qiita.com/104dev/items/#comments)

[51](https://qiita.com/104dev/items/0787a81f7dda892ce86a/likers)

27

ストックを更新しました