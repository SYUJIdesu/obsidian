---
title: "HapiCode"
source: "https://hapicode.com/php/laravel-sanctum.html"
author:
  - "[[HapiCode Blog]]"
published: 2年前
created: 2025-05-15
description: "Laravel JWT APIトークンの仕組みと Laravel Sanctum SPA認証使って認証周りの仕組みを構築"
tags:
  - "web"
  - "Laravel"
  - "認証"
---
## Laravel Sanctum 使って API トークン JWT 認証と SPA 認証

Laravel は Auth 認証システムとして、passport と sanctum を提供しています。Laravel passport は OAuth2 認証ベースで、Laravel Sanctum は API トークンベースのものになります。  
SPA やモバイルアプリケーションを認証したり、API トークンを発行したりする場合は、Laravel Sanctum を使うので、資料まとめました。

Laravel Sanctum とは

Laravel Sanctum(サンクタム、聖所)は Laravel が提供する API トークンを発行して SPA やモバイルアプリなどに API やり取りするための認証システムです。API トークンを DB に保存し、有効な API トークンを含む必要がある Authorization ヘッダを介して受信 HTTP リクエストを認証する機能を提供します。Laravel Blade で開発するなら Laravel passport がありますが、Laravel を API サーバーとして使う場合の認証システムは Larave sanctum になります。

## API トークン機能と SPA 認証機能

Laravel Sanctum は API トークン機能と SPA 認証機能という２つの機能があります。

==API トークン機能== は JWT 認証でよく使われる API トークンを Authorization ヘッダを介して受信 HTTP リクエストを認証します。  
==SPA 認証機能== はトークンの代わりにクッキーベースのセッション認証サービスを使用します。WEB 認証ガードを利用して、CSRF 保護、セッション認証の利点が提供できるだけでなく、XSS を介した認証資格情報の漏洩を保護します。

Laravel Sanctum は、受信リクエストが自身の SPA フロントエンドから発信された場合にのみクッキーを使用して認証を試みます。Sanctum が受信 HTTP リクエストを調べるとき、最初に認証クッキーをチェックし、存在しない場合は、Sanctum は有効な API トークンの Authorization ヘッダを調べます。

## インストール

パッケージインストール

```sh
composer require laravel/sanctum
```

### Sanctum の設定ファイルが config ディレクトリの下に外だし

```sh
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
```

### データベースのマイグレーションを実行

Sanctum は API トークンを格納するための 1 つのデータベーステーブルを作成

```sh
php artisan migrate
```

Sanctum のデフォルトマイグレーションを使用しない場合は、 `App\Providers\AppServiceProvider` クラスの register メソッドで `Sanctum::ignoreMigrations` メソッドを呼び出す必要があります。

次のコマンドを実行して、デフォルトマイグレーションをエクスポートできます。

```sh
php artisan vendor:publish --tag=sanctum-migrations
```

## ミドルウエアグループ編集

`app/Http/Kernel.php` にある api ミドルウエアグループを編集

```php
'api' => [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    'throttle:api',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```

## API トークン認証

SPA シングルページアプリケーションでの開発なら、API トークン認証より SPA 認証機能がおすすめです。

### トークンの発行

`Laravel\Sanctum\HasApiTokens` トレイトを使用して API トークン発行

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```
```php
use Illuminate\Http\Request;

Route::post('/tokens/create', function (Request $request) {
    $token = $request->user()->createToken($request->token_name);

    return ['token' => $token->plainTextToken];
});
```

### ルートに Middleware 設定

```php
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

### トークン削除

```php
// 全トークンの削除
$user->tokens()->delete();

// 現在のリクエストの認証に使用されたトークンを取り消す
$request->user()->currentAccessToken()->delete();

// 特定のトークンを取り消す
$user->tokens()->where('id', $tokenId)->delete();
```

### トークンの有効期限

デフォルトでは、Sanctum トークンに ==有効期限がなく== 、トークンの取り消しによってのみ無効化されます。

sanctum の設定ファイルで有効期限設定できます。

```php
'expiration' => 60,
```

有効期限過ぎたトークンを一括削除する

コマンドで 24 時間以上有効期限過ぎたものを一括削除できます。

```sh
php artisan sanctum:prene-expired --hours=24
```

スケジュール設定で 24 時間以上有効期限過ぎたものを一括削除できます。

```php
$schedule->command('sanctum:prune-expired --hours=24')->daily();
```

## SPA 認証

**セッションベース** の認証サービスを使用するため、web ガードを使います。つまり、login ルートは api ではなく web ルートにおいた方がいいです。

### CORS 設定

異なるサブドメインからアクセスできるように CORS 設定する必要があります。

`config/cors.php`

```php
'supports_credentials' => true,
```

#### フロント側 axios withCredentials を true に設定

```js
const api = axios.create({
  baseURL: "http://localhost",
  withCredentials: true,
});
```

#### セッションクッキードメイン設定

ここで設定した値は、Set-Cookie の domain 属性に設定されます。  
`config/session.php`

```php
// 'domain' => '.domain.com',
'domain' => env('SESSION_DOMAIN'),
```

### CSRF 保護

SPA を認証ログインする前には、最初に `/sanctum/csrf-cookie` にリクエストして、CSRF 保護を初期化する必要があります。

> このリクエスト中に、Laravel は現在の CSRF トークンを含む XSRF-TOKEN クッキーをセットします。このトークンは、後続のリクエストへ `X-XSRF-TOKEN` ヘッダで渡す必要があります。これは、Axios や Angular HttpClient などの一部の HTTP クライアントライブラリでは自動的に行います。JavaScript HTTP ライブラリで値が設定されていない場合は、このルートで設定された `XSRF-TOKEN` クッキーの値と一致するように `X-XSRF-TOKEN` ヘッダを手作業で設定する必要があります。

### ログイン処理

フロント側

`/sanctum/csrf-cookie` ルートにリクエストを送信しているため、JavaScript HTTP クライアントが XSRF-TOKEN クッキーの値を X-XSRF-TOKEN ヘッダで送信する限り、後続のリクエストは自動的に CSRF 保護を受けます。  
axios が `withCredentials: true` と設定する場合、XSRF-TOKEN をリクエスト時に送信してくれます。

ログイン Controller

`LoginController.php`

## postman で試す sanctum 　 SPA 認証

### Cookies 設定

Cookies > Manage Cookies > Domains Allowlist のメニューから設定できます。

フロントエンドのドメインを追加します。ローカルなので、 `localhost` になります。

### Login の仕組み

- POST `http://localhost/login?email=test@example.com&password=password`

Headers

```json
{
  "X-XSRF-TOKEN": {{xsrf-token}}
}
```

`X-XSRF-TOKEN: NaN` Key と Value を設定します。

次は Postman リクエスト時の ==Pre-request script== を設定します。

### ユーザー情報取得

ログインして CSRF 初期化と session を作成済み状態でユーザー情報取得できます。

- GET `localhost/api/user`

Headers

```json
{
  "Accept": "application/json",
  "Referer": "localhost"
}
```

2022-11-28
- php
- laravel

## 関連記事[正規表現一覧 よく使う検索・置換のパターン](https://hapicode.com/javascript/regular-expression.html)

[

よく使う正規表現、文字検索・置換をまとめてみました。...

](https://hapicode.com/javascript/regular-expression.html)[

Apache 初期設定メモ

LAMP環境のApache初期設定メモ...

](https://hapicode.com/php/apache.html)[

codeigniter email ライブラリでメール送信 日本語対応

codeigniterメール送信で日本語文字化け対策、改行コードとSMTP項目設定...

](https://hapicode.com/php/codeigniter-email.html)[

Carbon で php date 日付の日数・月数差を計算

Laravel Carbonは、DateTimeクラスよりもシンプルで読みやすいコードを書くことがで...

](https://hapicode.com/php/carbon.html)[

nuxtjs と codeigniter で jwt システム構築

nuxtjsとcodeigniterでシステム構築する際のメモ...

](https://hapicode.com/php/codeigniter-nuxtjs.html)[

Codeigniter APPPATH BASEPATH FCPATH 各種パスと URL 取得

codeigniter host url取得方法と APPPATH BASEPATHの意味を紹介...

](https://hapicode.com/php/codeigniter-path.html)[

爆速軽量フレームワーク codeigniter PHP 開発

ルーター設定もなく、初期設定も少ない！軽量と爆速で動いてくれるフレームワーク codeigniter...

](https://hapicode.com/php/codeigniter.html)[

Codeigniter 画像アップロードとリサイズ

Codeigniter アップロード画像リサイズと回転するなど画像操作するライブラリの使い方...

](https://hapicode.com/php/codeigniter-resize-upload.html)[

PHP mbconvertkana 全角半角英数カナ変換

php カナコンバートなら mb\_convert\_kana を使う！全角半角、英数字や記号など簡単に...

](https://hapicode.com/php/convert-kana.html)[

Composer コマンドとオプション

composer install updateの違い post-install-cmdとpost-u...

](https://hapicode.com/php/composer.html)[

php CSV データ取得は fgetcsv 使う

PHP csvアップロードしてデータインポート作業がありましたが、fgetcsv使ってcsv処理につ...

](https://hapicode.com/php/csv.html)[

PHP empty isset is\_null の違い

PHP empty, isset, is\_nullの違い よく使うPHP isset と if の違...

](https://hapicode.com/php/empty-isset.html)[

php Exception エラーキャッチでメール送信

エラーメッセージ、エラーコード、エラー詳細取得、php例外処理exceptionのいろいろ...

](https://hapicode.com/php/exception.html)[

FlattenException deprecated

laravelバージョンアップ作業する際にFlattenException deprecatedにつ...

](https://hapicode.com/php/flatten-exception.html)[

php curl 使って クリックなしで POST 送信

PHP curl\_setopt使ってシンプルにデータ転送してpost投げます。PHPは本当に短いソー...

](https://hapicode.com/php/curl-post.html)[

Class 'Imagick' not found Error

phpで画像操作したらimagickエラーで動かなくなったため、imagickモジュールをインストを...

](https://hapicode.com/php/imagick-error.html)[

allowurlinclude の設定で ftp\_connect()エラー

FTPサーバーとやりとりする時にftp\_connect()エラーが発生、PHP７にアップしたばかりで...

](https://hapicode.com/php/ftp-connect-error.html)[

開発におけるコーディングルール・規約

コーディングルールと規約を大事にする、複数人で開発しても、一人で開発してもメンテナンスやコードのわか...

](https://hapicode.com/php/coding-rule.html)[

Laravel blade foreach loop と current url

laravel blade foreach loop変数とurlのプロパティ対照表...

](https://hapicode.com/php/laravel-blade.html)[

Laravel Email バリデーションについて

Laravel Validation Email チェックする際の注意点、バリデーションルールオプシ...

](https://hapicode.com/php/laravel-email-validation.html)[

Laravel でカテゴリー作成 テーブル構築と再帰クエリ

カテゴリ関係を管理するためのテーブルを設計...

](https://hapicode.com/php/laravel-categories.html)[

Laravel eloquent model の規約

最新Laravel開発eloquent model設定資料まとめ...

](https://hapicode.com/php/laravel-eloquent.html)[

Laravel Error についてのメモ

laravelを使う時に出てくるエラー対応...

](https://hapicode.com/php/laravel-error.html)[

Laravel Lumen Faker 日本語設定

Laravel Faker 日本語化設定 fakerphpの使い方とよく使うオプション 名前や住所の...

](https://hapicode.com/php/laravel-faker.html)[

Laravel を API サーバーとしての初期設定

Laravel インストールからローカル環境のSail docker開発、初期設定までプロジェクト構...

](https://hapicode.com/php/laravel-init.html)[

Laravel Log の基本設定＆使い方

Laravel Logの基本設定 エラー発生時にslackに連絡通知する設定方法 tailコマンド使...

](https://hapicode.com/php/laravel-log.html)[

Laravel lang バリデーションメッセージを多言語対応

Laravelバリデーションのエラーメッセージを多言語で対応する手順を紹介...

](https://hapicode.com/php/laravel-lang.html)[

Laravel メンテナンスモード

Laravelのメンテナンスモードの使い方 Artisan down コマンド使ってメンテナンスモー...

](https://hapicode.com/php/laravel-maintenance.html)[

Laravel logger でエラーログを chatwork に自動送信

Laravel 使ってエラー発生した場合、chatworkにエラー内容をリアルタイムで連絡するcha...

](https://hapicode.com/php/laravel-logger.html)[

laravel method の基本 get post put options

Laravel Routeでget post メソッド同時に設定したい！Laravel ルート制約の...

](https://hapicode.com/php/laravel-method.html)[

Laravel model で hidden に設定したカラムを一時解除

Laravel モデルにhiddenで非表示になったカラムを一時的に表示するように変更しました。...

](https://hapicode.com/php/laravel-model-hidden.html)[

Laravel notification メール通知カスタマイズ

Laravelの通知（Notification）機能を使ってメール送信する作業があったので、まとめま...

](https://hapicode.com/php/laravel-notification.html)[

Laravel リクエストログ出力

Laravelでリクエストログを出力する手順をまとめました。...

](https://hapicode.com/php/laravel-request-log.html)[

Laravel Queue で非同期処理

Queue処理のメリットとLaravel Queue使って一括処理の例をまとめ...

](https://hapicode.com/php/laravel-queue.html)[

Laravel Sail で Docker 環境構築

Laravel Sail で Docker 開発環境PHP Mysql phpMyAdmin構築 s...

](https://hapicode.com/php/laravel-sail.html)[

Lumen と Laravel 違い比較

軽量で速いマイクロフレームワークlumenとlaravelの違い、使い方比較と本当にlumen必要か...

](https://hapicode.com/php/laravel-lumen.html)[

Laravel Test についてのメモ

Laravel自動テスト行う際のコマンドと設定 faker 使ってデータ生成...

](https://hapicode.com/php/laravel-test.html)[

laravel session を制する

laravel sesssion 使い方を紹介、読込保存削除を使えるようになったらsession制覇...

](https://hapicode.com/php/laravel-session.html)[

Laravel schedule 設定

Laravel スケジュール管理の基本とtimezone設定などlaravel 8最新メソッド説明...

](https://hapicode.com/php/laravel-schedule.html)[

Laravel 429 Too Many Requests

Laravel 19使ってAPIサーバー構築とフロントはReact nextjsで構築しています。複...

](https://hapicode.com/php/laravel-too-many-requests.html)[

Laravel toSql パラメータ付きで出力

Laravel Query Builder toSqlで出力したsql文にパラメータを自動でセット...

](https://hapicode.com/php/laravel-tosql.html)[

Laravel tinker 使って DB データベース接続とコマンド

Laravel tinker REPLツール使ってDBアクセス、環境確認など対話式で楽しく遊べる...

](https://hapicode.com/php/laravel-tinker.html)[

Laravel 5.1 から 8.1 にバージョンアップ

laravel5 から8にアップグレードする際に遭遇したエラーとカスタマイズauth認証の設定を修正...

](https://hapicode.com/php/laravel-upgrade.html)[

laravel に vuejs 使うための初期設定

PHPフレームワークlaravelにvuejs入れる時の初期設定まとめ、備忘録的...

](https://hapicode.com/php/laravel-vuejs.html)[

Lumen8 で JWT ユーザー認証

Lumen8でJWT認証のAPIサーバー構築してみた、便利になった時代だなあ...

](https://hapicode.com/php/lumen8-jwt.html)[

HTML から PDF に変換 PHP ライブラリ mPDF の設定

HTMLレンダリングしてPDFに変換出力するライブリmpdfを使いこなせるための設定いろいろ...

](https://hapicode.com/php/mpdf.html)[

Lumen8 で API 開発

codeigniter 愛用者がLumenでAPIサーバーを立ててみた...

](https://hapicode.com/php/lumen.html)[

Laravel timestamp() auto update 有効化無効化

Laravel updated\_at timestamp自動にしたい！MysqlはすぐにできるけどS...

](https://hapicode.com/php/laravel-timestamp.html)[

PHP 8 リリース新機能と変更

php8がリリースされたので、新機能と特徴などの資料まとめました。25周年迎えたPHPは更に速いプロ...

](https://hapicode.com/php/php8.html)[

解決！phpMyAdmin テーブル構造の内容が表示されない問題

phpmyadmin5.0.1のxampp入れたら テーブル構造メニューが真っ白になって表示されない...

](https://hapicode.com/php/phpmyadmin-error.html)[

PHP 7.4 にアップグレードして使えなくなる機能

php 5 から php7.4 にアップする作業があったので、アップグレード時のエラーと7.4では動...

](https://hapicode.com/php/php74.html)[

PHP 文字列長さ・文字列の幅を取得方法

PHP 文字列操作関数 マルチバイト文字列日本語韓国語中国語の長さ取得など...

](https://hapicode.com/php/string-len.html)[

php.ini 初期設定のいろいろ

php ini ローカル環境や新規リモート環境の初期設定...

](https://hapicode.com/php/phpini.html)[

Smarty HTTP URL 取得できるサーバー関数

スマーティで取得するサーバー関数URLとenvなど...

](https://hapicode.com/php/smarty.html)[

twig 3 人気 PHP テンプレートエンジンがバージョンアップ

以前からtwigテンプレートエンジン使っていて、非常にパワーフルで使いやすさに魅了しました。15日に...

](https://hapicode.com/php/twig.html)[

Exception: Class 'ZipArchive' not found

エクセル操作でZipArchive classが見つからないエラーがあったので、対応しました。...

](https://hapicode.com/php/zip-archive-error.html)[

開発時によく使うゼロ埋めパディング作業まとめ

mysql twig phpとjavascriptでよく使うゼロ埋めのいろいろ...

](https://hapicode.com/php/zero-padding.html)[

AWS SES メール開封確認　 DB に集計

メール送信の開封確認して DB更新するために使う AWS サービス SES SNS LAMBDA と...

](https://hapicode.com/server/aws-mail-open.html)