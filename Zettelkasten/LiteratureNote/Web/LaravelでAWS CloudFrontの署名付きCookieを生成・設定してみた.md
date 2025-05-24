---
title: LaravelでAWS CloudFrontの署名付きCookieを生成・設定してみた
source: https://zenn.dev/carenet/articles/laravel-cloudfront-s3#%E3%81%AF%E3%81%98%E3%82%81%E3%81%AB
author:
  - "[[Zenn]]"
published: 2024-06-28
created: 2025-05-15
description: 
tags:
  - web
  - Laravel
---
[CareNet Engineers](https://zenn.dev/p/carenet) [Publicationへの投稿](https://zenn.dev/faq#what-is-publication)

9[tech](https://zenn.dev/tech-or-idea)

## はじめに

こんにちは、Kouです。

AWS CloudFrontを使用すると、コンテンツの配信を効率化できますが、  
例えば、企業が限定イベントをオンラインでストリーミング配信し、チケット購入者のみが視聴できるようにしたいケースでは、  
チケット購入者が他人に視聴リンクを共有しても不正なアクセスを防ぐ必要があります。  
本記事では、Laravelを使用してCloudFrontの署名付きCookieを生成し、設定する方法について詳しく説明します。

前回は「Laravel Auditingで始めるデータ監査：不正やミスを簡単に追跡できるようにする方法」を投稿しました。

## 前提条件

以下はセットアップ済みとします。

1. AWSアカウント
2. CloudFront ディストリビューション
3. Laravelプロジェクト

## 手順

1. [CloudFrontに代替ドメイン名（CNAME）を追加する手順](https://zenn.dev/carenet/articles/#cloudfront%E3%81%AB%E4%BB%A3%E6%9B%BF%E3%83%89%E3%83%A1%E3%82%A4%E3%83%B3%E5%90%8D%EF%BC%88cname%EF%BC%89%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B%E6%89%8B%E9%A0%86)
2. [CloudFrontのアクセス制限を設定する手順](https://zenn.dev/carenet/articles/#cloudfront%E3%81%AE%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E5%88%B6%E9%99%90%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B%E6%89%8B%E9%A0%86)
3. [Laravelで署名付きCookieを生成するミドルウェアを作成する手順](https://zenn.dev/carenet/articles/#laravel%E3%81%A7%E7%BD%B2%E5%90%8D%E4%BB%98%E3%81%8Dcookie%E3%82%92%E7%94%9F%E6%88%90%E3%81%99%E3%82%8B%E3%83%9F%E3%83%89%E3%83%AB%E3%82%A6%E3%82%A7%E3%82%A2%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B%E6%89%8B%E9%A0%86)

## CloudFrontに代替ドメイン名（CNAME）を追加する手順

CloudFrontの署名付きCookieを利用するためには、代替ドメインを設定し、Cookieを設定するドメインと一致させることが必要です。  
以下はその手順になります。

1. AWS マネジメントコンソールにログインします。
2. CloudFront に移動:  
	コンソールのサービス一覧から「CloudFront」を選択します。
3. ディストリビューションの選択:  
	代替ドメイン名を追加したい CloudFront ディストリビューションを選択します。
4. ディストリビューションの編集:  
	選択したディストリビューションの「設定の編集」ボタンをクリックします。
5. 代替ドメイン名 (CNAME) の追加:  
	「代替ドメイン名 (CNAME)」のフィールドに、追加したいドメイン名（例: www.example.com）を入力し保存します。  
	※複数のドメイン名を追加する場合は、カンマで区切って入力します。  
	※代替ドメイン名は有効な SSL/TLS 証明書の設定がある必要があります。（手順6~8）
6. ACM に移動:  
	コンソールのサービス一覧から「Certificate Manager」を選択します。

![](https://res.cloudinary.com/zenn/image/fetch/s--nIAhK_kz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/678601662cc82e57c4decc1f.png%3Fsha%3D966db5185fcbe81a14b6ed9c3bc08899adc9be70)

1. 証明書の設定
	- リクエスト（新規の証明書）
		- 「リクエスト」ボタンをクリックし、必要な情報を入力して証明書をリクエストします。
			- 証明書の検証: ドメイン所有権を確認するための手順に従い、証明書を検証します。
	- インポート（サードパーティーの認証を利用する場合）
		- 証明書本文, プライベートキー, 証明書チェーンをそれぞれ貼り付ける。
2. CloudFront に証明書を追加:  
	CloudFront ディストリビューションの設定で、設定した証明書を選択します。
3. DNS 設定の更新:  
	最後に、ドメイン名の DNS 設定で CloudFront ディストリビューションのドメイン名（例: d123456abcdef8.cloudfront.net）を指すように CNAME レコードを設定します。

![](https://res.cloudinary.com/zenn/image/fetch/s--BJONb4jt--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/266e51f0e88def3b5d04013a.png%3Fsha%3Dba2bc9f2e4fd6400eadb1e7eea3f1da60ef4445b)

## CloudFrontのアクセス制限を設定する手順

CloudFrontディストリビューションのビューワーアクセスを制限する方法について、以下の手順で詳しく説明します。  
これにより、特定の条件下でのみコンテンツにアクセスできるようになります。

1. ディストリビューションの選択:  
	アクセスを制限したい CloudFront ディストリビューションを選択します。
2. ビヘイビアの編集:  
	「ビヘイビア」タブに移動し、アクセスを制限したいビューワーリクエストに対応するビヘイビアを選択します（例えば、デフォルトのビヘイビアや特定のパスパターンを持つビヘイビアなど）。
3. 設定の編集:  
	選択したビヘイビアの「設定の編集」ボタンをクリックします。
4. アクセス制限の設定:  
	「ビューワーのアクセスを制限する」の「Yes」を選択します。
5. 信頼された認可タイプの設定:  
	「信頼された認可タイプ」で「Trusted key groups (recommended)」を選択します。
6. 信頼されたキーグループの設定:  
	「信頼されたキーグループ」セクションで、「キーグループ」を入力します。（作成方法はリンク先を参照）  
	[https://docs.aws.amazon.com/ja\_jp/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html#choosing-key-groups-or-AWS-accounts](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html#choosing-key-groups-or-AWS-accounts)

![](https://res.cloudinary.com/zenn/image/fetch/s--AhA3hKbF--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/ff9d6bb5afe53810c5ffb48a.png%3Fsha%3D8c71ce9d0b0df3a9ebb49086b474ed4002412579)

1. キャッシュキーとオリジンリクエスト:

こちらは長くなるので大幅に省略しますが、キャッシュポリシーとオリジンリクエストポリシーは適宜必要なものを設定します。

レスポンスヘッダーポリシーの設定:  
レスポンスヘッダーポリシーを設定することで、JavaScriptからのクロスオリジンリクエストに対して適切なレスポンスヘッダーを提供し、  
ブラウザがリクエストを許可します。これにより、JavaScriptからのFetchリクエストでCookieを含むリクエストが正常に送信されるようになります。

「Create response headers policy」からポリシーの作成をします。

「CORSを設定」を選択します。  
「Access-Control-Allow-Credentials」にチェックを入れ、各項目に必要な情報を設定します。

![](https://res.cloudinary.com/zenn/image/fetch/s--_d5natIe--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/f838f9805ddbcad0ea7e1861.png%3Fsha%3D28dcaba8c562b2679894b4287844a60f1c9723e2)

## Laravelで署名付きCookieを生成するミドルウェアを作成する手順

### 環境設定ファイルの設定

.env ファイルにCloudFrontの設定を追加します。

.env

```php
AWS_CLOUDFRONT_KEY_PAIR_ID=your_key_pair_id
   AWS_CLOUDFRONT_PRIVATE_KEY_PATH=/path/to/your/private/key.pem
   AWS_CLOUDFRONT_RESOURCE_URL=https://www.example.com/your/resource/path/*
```

次に、config/services.php ファイルにCloudFrontの設定を追加します。

config/services.php

```php
+ 'aws' => [
+     'key_pair_id' => env('AWS_CLOUDFRONT_KEY_PAIR_ID'),
+     'private_key_path' => env('AWS_CLOUDFRONT_PRIVATE_KEY_PATH'),
+     'resource_url' => env('AWS_CLOUDFRONT_RESOURCE_URL'),
+ ],
```

### Laravelで署名付きCookieを生成するミドルウェアを作成

`php artisan make:middleware SignedCookiesMiddleware`

sail環境

`sail artisan make:middleware CloudFrontSignedCookie`

app/Http/Middleware/CloudFrontSignedCookie.php

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Aws\CloudFront\CloudFrontClient;

class CloudFrontSignedCookie
{
    public function handle(Request $request, Closure $next)
    {
        // CloudFront署名付きCookieの生成
        $cookies = $this->createSignedCookies();

        // Cookieをレスポンスに追加
        $response = $next($request);
        foreach ($cookies as $name => $value) {
            $response->headers->setCookie(cookie($name, $value, 60));
        }

        return $response;
    }

    private function createSignedCookies()
    {
        // CloudFrontクライアントの作成
        $client = new CloudFrontClient([
            'region'  => 'us-east-1',
            'version' => 'latest',
            'credentials' => false, // 署名付きCookie生成のためのクレデンシャルは不要
        ]);

        // 保護されたリソースのURL
        $resourceUrl = config('services.aws.resource_url'); // 必要なURLに変更

        // 有効期限の設定（例: 1時間後）
        $expires = time() + 60 * 60;

        // ポリシーの生成
        $policy = json_encode([
            'Statement' => [
                [
                    'Resource' => $resourceUrl,
                    'Condition' => [
                        'DateLessThan' => [
                            'AWS:EpochTime' => $expires
                        ]
                    ]
                ]
            ]
        ]);

        // 署名の生成
        $privateKey = file_get_contents(storage_path(config('services.aws.private_key_path')));
        $signedPolicy = $client->getSignedCookie([
            'policy' => $policy,
            'key_pair_id' => config('services.aws.key_pair_id'),
            'private_key' => $privateKey,
        ]);

        // 署名付きCookieの生成
        return $signedPolicy;
    }
}
```

### Cookieの暗号化を無効にする

EncryptCookiesの$except配列に署名付きCookieの名前を追加します。

### ミドルウェアの登録

次に、このミドルウェアをアプリケーションに登録します。app/Http/Kernel.php ファイルを編集し、$routeMiddleware 配列にミドルウェアを追加します。

### ルートの設定

最後に、署名付きCookieを必要とするルートにミドルウェアを適用します。

## 注意点

レスポンスヘッダーポリシーの設定でも触れた通り、 JavaScriptからのリクエスト（Fetch APIやXMLHttpRequest）ではデフォルトでCookieが送信されません。  
これはセキュリティ対策の一環で、意図しないクロスサイトリクエストにCookieを送信しないようにするためです。

そのため、CloudFrontからコンテンツを取得する際は対応が必要です。

### 解決方法

JavaScriptからのリクエストでCookieを送信するために、Credentialsオプションを使用する必要があります。  
使用しているJavaScriptライブラリによってはCredentialsオプションがあるかもしれません。

#### Fetch APIの設定例

Fetch APIを使用してリクエストを行う際、Credentialsオプションを適切に設定します。

ただし、以下の対策は必要になります。

- CSRFトークンをリクエストに含めて、サーバー側で検証する。
- クッキーにSameSite属性を設定して、クロスサイトリクエストでのクッキー送信を制限する。
- CORS設定を強化して、特定のオリジンのみからのリクエストを許可する。

```php
fetch('your-url', {
    method: 'GET', // または 'POST' など
    credentials: 'include' // ここが重要
    
    // カスタムヘッダーとしてCSRFトークンを送信
    // CSRF-Token: csrfToken
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));
```

## 参考

[GitHubで編集を提案](https://github.com/koya-cn/zenn/blob/main/articles/laravel-cloudfront-s3.md)

9