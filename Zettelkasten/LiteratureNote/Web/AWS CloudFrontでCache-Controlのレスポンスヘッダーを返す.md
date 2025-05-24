---
title: "[AWS] CloudFrontでCache-Controlのレスポンスヘッダーを返す"
source: https://qiita.com/t-son-wk/items/ede6d6299db5f3f2369e
author:
  - "[[t-son-wk]]"
published: 2023-04-04
created: 2025-05-22
description: 概要AWS CloudFrontでレスポンスヘッダーにCache-Controlを返す方法を記載します。経緯画像をS3に置いて、CloudFrontを通してサイトに配信しています。その際対象…
tags:
  - web
  - AWS
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

## 概要

AWS CloudFrontでレスポンスヘッダーにCache-Controlを返す方法を記載します。

## 経緯

画像をS3に置いて、CloudFrontを通してサイトに配信しています。

その際対象のサイトを `PageSpeed Insights` で計測をすると、CloudFront経由で配信しているS3の画像が「キャッシュの設定をしてくだいさい」的な感じで警告になる場合がありました。

原因はCloudFrontのキャッシュにはヒットしているのですが、レスポンスにCache-Controlが無いため、PageSpeed Insightsではブラウザキャッシュの設定をしてくださいと出ているようだと思いました。

本方法と別方法で、S3の画像にカスタムヘッダーでCache-Controleをつければ解決する方法もあるようですが、画像アップロードのたびにカスタムヘッダー情報を付けたりする方法が少し面倒でCloudFrontの設定で付けれないか試してみたのがきっかけになります。

> 参考  
> [https://www.aws-room.com/entry/cloudfront-s3](https://www.aws-room.com/entry/cloudfront-s3)

## 設定

### CloudFrontのレスポンスヘッダーを新規作成

1. CloudFrontのレスポンスヘッダーの新規作成画面をひらきます  
	[![](https://storage.googleapis.com/zenn-user-upload/b4678be6791c-20230303.jpg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fstorage.googleapis.com%2Fzenn-user-upload%2Fb4678be6791c-20230303.jpg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b6c997090a881ddf5fdeee030d6f3099)
2. 名前・説明を入れ、カスタムヘッダーを追加します。それ以外はデフォルトのままです。  
	[![](https://storage.googleapis.com/zenn-user-upload/5d7817fad38b-20230303.jpg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fstorage.googleapis.com%2Fzenn-user-upload%2F5d7817fad38b-20230303.jpg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d11354d60200133ef764fc90409066f5)

### ビヘイビアの設定に追加する

1. CloudFrontのビヘイビア編集画面に入ります
2. レスポンスヘッダーポリシーで先程作成したポリシーを選択し、変更を保存します  
	[![](https://storage.googleapis.com/zenn-user-upload/b9deca4614de-20230303.jpg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fstorage.googleapis.com%2Fzenn-user-upload%2Fb9deca4614de-20230303.jpg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5af89ac2b3508ff413ed4452e8e78835)

## 確認

1. PageSpeed Insightsで確認する
2. ブラウザで開発ツールなどでResponse Headersを確認する  
	（下記画像はChrome > デベロッパーツール > ネットワークタブ > 任意の画像を選択）  
	[![](https://storage.googleapis.com/zenn-user-upload/dab009b97859-20230404.jpg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fstorage.googleapis.com%2Fzenn-user-upload%2Fdab009b97859-20230404.jpg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2d3eaac53da1c5b062623bd91561f73b)

以上になります。

[0](https://qiita.com/t-son-wk/items/#comments)

コメント一覧へ移動

[0](https://qiita.com/t-son-wk/items/ede6d6299db5f3f2369e/likers)

いいねしたユーザー一覧へ移動

3