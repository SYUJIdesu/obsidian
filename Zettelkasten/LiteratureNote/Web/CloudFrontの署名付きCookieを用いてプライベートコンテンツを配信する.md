---
title: CloudFrontの署名付きCookieを用いてプライベートコンテンツを配信する
source: https://qiita.com/DaichiAndoh/items/738cf75b015a7d2b50c8
author:
  - "[[DaichiAndoh]]"
published: 2024-02-29
created: 2025-05-22
description: はじめにこの記事は、CloudFrontの署名付きCookieを用いたプライベートコンテンツ配信についてまとめています。背景現在開発しているWebアプリケーションに、以下のような追加実装を行う…
tags:
  - web
  - AWS
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

## はじめに

この記事は、CloudFrontの署名付きCookieを用いたプライベートコンテンツ配信についてまとめています。

## 背景

現在開発しているWebアプリケーションに、以下のような追加実装を行うことになりました。

- 認証されたユーザーのみがアクセス可能なコンテンツを作成する
- コンテンツは静的なWebページ
- 認証機能は既存のものを使用する

この仕様を踏まえて、実装方法を検討した結果、CloudFrontの署名付きCookieを採用することにしました。  
Webアプリケーションの特徴的に、ユーザー数（コンテンツへのアクセス数）はそこまで多くならない想定であり、既にあるWebアプリケーションのインフラは全てAWSで構築されていますので、運用コスト・実装コスト面でも最適な選択肢と考えました。

## 前提

## CloudFrontの署名付きCookieとは

CloudFrontはAWSのCDNサービスであり、Webアプリケーションのフロント画面や、サイト内で使用される画像等の静的コンテンツをパブリックに配信するという使い方をよく耳にします。

ただ、開発物によっては、パブリックではなく、ユーザーによるアクセス制御を行った上で、コンテンツ配信を高速に行いたいといった要件が出てくることも容易に想像できます。

そのような要件を満たすために使用できるのが、署名付きCookieという機能です。  
簡単な全体像としては以下のような流れになります。

**登場するモジュール**

- CloudFront
- 署名付きCookieを作成して、ブラウザ側にセットするアプリケーション

**流れ**

1. 署名のための秘密鍵と公開鍵のキーペアを作成する
2. CloudFront側に公開鍵を設定する
3. アプリケーション側で秘密鍵を用いて署名したCookieを作成し、ブラウザにセットする
4. ブラウザは署名付きCookieがセットされた状態でCloudFrontにアクセス
5. CloudFront側で署名付きCookieを検証し、問題がなければコンテンツへのアクセスを許可する（署名付きCookieがない、不適切な場合はアクセスを拒否する）

CloudFrontでユーザーによるアクセス制御を行う方法としては、署名付きURLという機能もあり、AWS公式ドキュメントでも署名付きCookieと署名付きURLは比較して説明されていたりします。

[署名付きURLと署名付きCookieの選択](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/private-content-choosing-signed-urls-cookies.html)

署名付きURLの説明はここでは割愛させていただきますが、それぞれに特徴があり、要件に応じて選択することになります。  
今回の場合、ユーザーはブラウザからアクセスしてくる（Cookieをサポートしているクライアント）、複数コンテンツをアクセス制御の対象にしたいといったことから署名付きCookieを選択しました。  
また、署名付きURLでは、認証していないユーザーであってもURLが分かればアクセス可能になるので、署名付きCookieの方がプライベートな配信を高いレベルで実現できるかと思います。

## 本題

それでは、署名付きCookieを用いたプライベートコンテンツ配信の仕組みを実際に作成していきます。  
全体の流れは以下のようになります。

1. CloudFrontのOACを用いて、CloudFrontとコンテンツを管理するS3バケットを接続する
2. キーペアを作成
3. パブリックキーを作成
4. キーグループを作成
5. CloudFrontのビヘイビアを編集
6. 署名付きCookieを作成するアプリケーションを作成
7. 動作確認

## 1\. CloudFrontのOACを用いて、CloudFrontとコンテンツを管理するS3バケットを接続する

CloudFrontに署名付きCookieを設定しても、オリジン（今回はS3）にパブリックアクセスできるようでは意味がありません。  
CloudFrontの署名付きCookieはCloudFrontへのアクセスは制御できますが、S3へのアクセスは制御できません。そのため、S3へのアクセスはCloudFrontのみに限定する必要があります。

CloudFrontとS3を接続するためにOACを用います。  
これについては別途記事を書きましたので、 [こちら](https://qiita.com/DaichiAndoh/items/d48daae9df552b2c30a6) をご覧ください。

この記事では、CloudFrontとS3をOACを用いて接続し、S3には `index.html` と `error.html` という2つのHTMLファイルをアップロードしています。

## 2\. キーペアを作成

キーペアには要件がありますが、基本的には [公式ドキュメント](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html#private-content-creating-cloudfront-key-pairs) の通りに作成すれば問題ありません。

```text
$ openssl genrsa -out private_key.pem 2048
$ openssl rsa -pubout -in private_key.pem -out public_key.pem
```

## 3\. パブリックキーを作成

CloudFrontコンソールでパブリックキーを作成します。

[![スクリーンショット 2024-02-28 10.19.18.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/dcbc00fb-94bd-1522-2a35-f2c6b78db77e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2Fdcbc00fb-94bd-1522-2a35-f2c6b78db77e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=37495fdde3c149c83b2fc844c53ed003)

「キー」の入力フォームには、先ほど作成した公開鍵の中身をコピー&ペーストします。

[![スクリーンショット 2024-02-28 10.39.58.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/27650e04-61dd-7886-a27e-b79a2d657d33.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2F27650e04-61dd-7886-a27e-b79a2d657d33.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=39485ab8efa1cef92831cc761c3b9703)

[![スクリーンショット 2024-02-28 10.20.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/48d51c6b-9871-9a9f-d711-8938b89ad279.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2F48d51c6b-9871-9a9f-d711-8938b89ad279.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=dbbf910f487beecea406607eaefd04c4)

## 4\. キーグループを作成

CloudFrontコンソールでキーグループを作成します。

[![スクリーンショット 2024-02-28 10.29.02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/0c3db1e9-d4d0-2317-3651-e8277e8030ac.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2F0c3db1e9-d4d0-2317-3651-e8277e8030ac.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=15ba0f78dd61c03f347344ff0f50b0da)

パブリックキーは先ほど作成したものを指定します。

[![スクリーンショット 2024-02-28 10.45.33.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/3641d2ea-8ee9-5780-e3fd-28823ac9e0b9.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2F3641d2ea-8ee9-5780-e3fd-28823ac9e0b9.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9f9643acaf5350321b7b2db268805d8e)

[![スクリーンショット 2024-02-28 10.46.00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/83f06660-dc9e-3977-9647-dc60cc7f95de.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2F83f06660-dc9e-3977-9647-dc60cc7f95de.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=db38a84292a15fa7cf7e22ee268f950e)

## 5\. CloudFrontのビヘイビアを編集

現在、手順1で作成したディストリビューションには、全てのコンテンツに対してのビヘイビアが設定されている状態です。  
そしてこのビヘイビアには、署名付きCookieなどのアクセス制御の設定は行われていませんので、全てのコンテンツにアクセスできる状態となっています。

今回は、全てのコンテンツをアクセス制御の対象にしたいため、この既存のビヘイビアを編集していきます。  
つまり、このビヘイビアに署名付きCookieの設定を追加します。

ビヘイビアの編集画面を開きます。

[![スクリーンショット 2024-02-29 10.42.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/bbce474d-66bd-353f-ff7a-6edfefd4816e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2Fbbce474d-66bd-353f-ff7a-6edfefd4816e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=86a31bd2a42d5da29a31e71cce3769c3)

「ビューワーのアクセスを制限する」の項目で「Yes」を選択します。  
そして、「信頼された認可タイプ」に先ほど作成したキーグループを指定します。  
設定できれば編集は完了です。

[![スクリーンショット 2024-02-29 10.43.32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/484fea76-3213-5ff3-6cee-d8cfd87fa6a9.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2F484fea76-3213-5ff3-6cee-d8cfd87fa6a9.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=939b523abcdb2ec4c7a9f4c5b51f6cef)

これで、全てのコンテンツに署名付きCookieを用いたアクセス制御が設定されたので、確認してみます。  
S3にある2つのHTMLファイルへのアクセスが拒否されます。

[![スクリーンショット 2024-02-29 10.50.34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/3916703e-be43-b37e-5758-b1490b65c8e1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2F3916703e-be43-b37e-5758-b1490b65c8e1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f854d01943c5afda59b939b997775366)

[![スクリーンショット 2024-02-29 10.49.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/8fb2f414-17b0-dda1-6c0e-1f2461695c9d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2F8fb2f414-17b0-dda1-6c0e-1f2461695c9d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3eeab7dc20d9d0836a52754f4abd5b78)

## 6\. 署名付きCookieを作成するアプリケーションを作成

このブロックのタイトルにアプリケーションと記載していますが、ここでは簡単なプラグラムをNode.jsで実行して、標準出力に署名付きCookieを出力します。

Node.jsで署名付きCookieを作成するソースコードは、AWSのSDKに含まれています。

- GitHub
	- [https://github.com/aws/aws-sdk-js-v3/tree/main/packages/cloudfront-signer](https://github.com/aws/aws-sdk-js-v3/tree/main/packages/cloudfront-signer)
- npm
	- [https://www.npmjs.com/package/@aws-sdk/cloudfront-signer](https://www.npmjs.com/package/@aws-sdk/cloudfront-signer)

`cloudfrontDistributionDomain` と `keyPairId` はそれぞれ上の手順で作成したCloudFrontディストリビューションのドメイン名とパブリックキーIDを指定してください。

ここで署名付きCookieのポリシーというものについて簡単に説明しておきます。  
署名付きCookieには規定ポリシーとカスタムポリシーという2つのポリシーがあり、それぞれの特徴を踏まえて開発物の要件に適した方を選択する必要があります。

この2つの違いについては [公式ドキュメント](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-cookies.html) に詳しく記載されていますが、簡単な比較表を以下に抜粋させていただきます。

[![スクリーンショット 2024-02-28 18.16.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/194979bc-1575-2a36-e75b-fecc02951e0c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2F194979bc-1575-2a36-e75b-fecc02951e0c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7906652be78cc4508f1047d0133b3898)

今回は、複数のコンテンツを対象としたいため、カスタムポリシーを選択しています。  
上のJavaScriptのソースコードもカスタムポリシー用の署名付きCookieを作成するようになっています（規定ポジシーとカスタムポリシーでは必要となる署名付きCookieも異なります）。

では、JavaScriptのソースコードを実行します。  
実行すると、標準出力に3つの署名付きCookieが出力されます。

```bash
$ node index.js
{
  'CloudFront-Key-Pair-Id': 'K33GQBHZF4GDTS',
  'CloudFront-Signature': 'V3nyamwvLuX0xtX1dUfH6kCRGo3P7JQ5pQllcWhZeJrFKqNUcpSRXHR-kaFSmNURdk2siQWEo~zl1LZPUVh7hvR5hbiVNOR-d0ebnuWB1x~GcCYyzPqt5usOdwYWTEbsXK-e8Qk5Q7CbVGTDx1WAWu5dLLMwnbb9il5husoehMN2~puLWPkGCkrQD6b77XFxXErDpIipvCrZeslSpPaR-mcNLZ4DVL1SHt6KWyqeo4GJbVn-NyvuiwKZtxhKJOO-b2S~8I2N1YS6aJbl6H2u~tunTx5foEr6NASbZcA7fom8fUDb7yeIneAFi2OxBWtYX1DCPr6R-hsbvTTe19ep5g__',
  'CloudFront-Policy': 'eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9kM2FqZmxvenI5MDV0cS5jbG91ZGZyb250Lm5ldC8qIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzA5MjE4ODAwfX19XX0_'
}
```

## 7\. 動作確認

動作確認を行います。  
まずは、署名付きCookieを設定しない状態でアクセスしてみます。

CloudFrontのビヘイビア編集後にブラウザで確認した時と同様、エラーが返ってきます。  
では、署名付きCookieをヘッダーにセットした状態でアクセスしてみます。

どちらのコンテンツにもアクセスすることができました。  
先ほど規定ポリシーとカスタムポリシーについては軽く触れたときに、今回は複数のコンテンツをアクセス制御の対象としたいためにカスタムポリシーを選択したと記載しました。

今回、2つのコンテンツ（オリジンのS3にある全てのコンテンツ）に対して、同じ署名付きCookieを用いてアクセスすることができました。  
これは、カスタムポリシーの `Resource` の値にワイルドカードを使用することで、オリジンのS3にある全てのコンテンツを指定しているからです。

規定ポリシーでは `Resource` にワイルドカードを使用できないため、作成した署名付きCookieでアクセスできるコンテンツは1つに限られます。  
もし今回のケースで規定ポリシーを使用する場合、以下のような流れになると思います。

1. 署名付きCookieを作成するアプリケーションで `Resource` に `index.html` を指定した署名付きCookieを作成
2. CloudFrontにアクセス（ `index.html` にアクセスできる）
3. 次に `error.html` に署名付きCookieを用いてアクセスしたい場合、再びアプリケーションで `Resource` に `error.html` を指定した署名付きCookieを作成
4. CloudFrontにアクセス（ `error.html` にアクセスできる）

つまり、毎回アクセスするコンテンツに対応した署名付きCookieを設定する必要があるということです。  
これは、現実的ではありません。

そのため、複数コンテンツを対象にして署名付きCookieを用いる場合は、カスタムポリシーを使用して、ポリシーステートメントがコンテンツ間で再利用されるようにします。

## 補足

## 署名付きCookieを作成するアプリケーションについて

上の説明では、署名付きCookieを作成するアプリケーションは、標準出力に署名付きCookieを出力する簡単なプログラムとしています。ただし、実際には、Web APIなどの形態で、レスポンスを通してブラウザ側に署名付きCookieをセットするアプリケーションにする必要があります。  
Set-Cookieヘッダーを使用して、作成した署名付きCookieをブラウザにセットする具合です。

秘密鍵はSecrets Managerなどで管理して、そこから読み込むようにすると良いと思います。

署名付きCookie作成の部分は今回のコードをほぼそのまま使用できると思います。

Cookieをセットする際にアプリケーションとCloudFrontのドメインに注意する必要があります。ドメインが異なれば、Cookieをセットできません。  
これについては [こちらの記事](https://qiita.com/HAYASHI-Masayuki/items/209039717c15834603d8) が参考になると思います。

## 実装後の簡単な全体像

今回CloudFrontの署名付きCookieを用いることで、本記事冒頭部分の「背景」で記載した要件を満たす実装ができました。実装後の簡単な全体像は以下のようになります。

[![スクリーンショット 2024-02-29 14.52.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3687813/d4de7788-1753-bde8-069f-181e648a8325.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3687813%2Fd4de7788-1753-bde8-069f-181e648a8325.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2bf15ebc27f9a2dfdda65bbcddfd6e28)

要件を改めて確認します。

- 認証されたユーザーのみがアクセス可能なコンテンツを作成する
- コンテンツは静的なWebページ
- 認証機能は既存のものを使用する

既存の認証機能を使用して、認証に成功したユーザーに対して署名付きCookieを作成する処理を追加しました。

## 最後に

最後まで読んでいただきありがとうございます。  
読んでいただいた方にとって何らかのプラスになっていれば嬉しいです。

## 参考ページ

本記事作成にあたり参考にさせていただきました。  
ありがとうございました。

- [https://docs.aws.amazon.com/ja\_jp/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html)
- [https://github.com/aws/aws-sdk-js-v3/tree/main/packages/cloudfront-signer](https://github.com/aws/aws-sdk-js-v3/tree/main/packages/cloudfront-signer)
- [https://www.npmjs.com/package/@aws-sdk/cloudfront-signer](https://www.npmjs.com/package/@aws-sdk/cloudfront-signer)
- [https://dev.classmethod.jp/articles/aws-sdk-for-javascript-v3-aws-sdk-cloudfront-signer-get-signed-cookie/](https://dev.classmethod.jp/articles/aws-sdk-for-javascript-v3-aws-sdk-cloudfront-signer-get-signed-cookie/)
- [https://qiita.com/HAYASHI-Masayuki/items/209039717c15834603d8](https://qiita.com/HAYASHI-Masayuki/items/209039717c15834603d8)

[0](https://qiita.com/DaichiAndoh/items/#comments)

コメント一覧へ移動

[6](https://qiita.com/DaichiAndoh/items/738cf75b015a7d2b50c8/likers)

いいねしたユーザー一覧へ移動

2

ストックを更新しました