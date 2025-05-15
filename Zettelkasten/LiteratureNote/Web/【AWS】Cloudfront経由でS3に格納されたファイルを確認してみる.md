---
title: "【AWS】Cloudfront経由でS3に格納されたファイルを確認してみる"
source: "https://zenn.dev/ashisan/articles/571f2d1c0f2c36#%E5%8E%9F%E5%9B%A0"
author:
  - "[[Zenn]]"
published: 2021-04-19
created: 2025-05-15
description:
tags:
  - "web"
  - "CloudFront"
  - "S3"
---
10[tech](https://zenn.dev/tech-or-idea)

こんにちは。  
CloudFront使っていますか。  
個人的にCloudFrontは、AWS-Solution Architectの資格試験勉強中によく見たサービスで、名前と役割については知っていましたが、実際使ってみたことはありませんでした。  
構築自体は簡単でしたが、ハマったところもあったので、備忘も兼ねて記事にしました。

## 対象読者

- AWS初心者の方
- AWS認定試験(SAAなど)を勉強中の方

## 前提条件

- AWSアカウントを所持していて、AWSコンソールにログインできること

## 本書の目的

- AWS Cloudfront経由でS3に格納された画像ファイルを確認する
- 下記画像のように、S3への直接アクセスは制限し、CloudFront経由でのみアクセス許可する  
	※矢印はhttps接続を表す

![](https://storage.googleapis.com/zenn-user-upload/buj82nl7sl4den4oz6hvmev12yqv)

## 構築手順

## 1\. S3バケットを作成する

- S3管理コンソールから「バケットを作成」をクリックする  
	![](https://storage.googleapis.com/zenn-user-upload/vmqjhsbnc1ej532bo4zhhoawhapq)
- 任意のバケット名、リージョンを選択する(今回はap-northeast-1リージョンを選択)  
	![](https://storage.googleapis.com/zenn-user-upload/d3jheg2cnu1y77op2m0zigb8g2tc)

**※下記項目のチェックは入れておく(デフォルトでチェックされている、バケットのパブリックアクセスをブロック)**  
![](https://storage.googleapis.com/zenn-user-upload/m2s8o9mj3th32n30ippx0uml2etq)

- 一番下の「バケットの作成」をクリックし、S3バケットが作成できたことを確認する  
	![](https://storage.googleapis.com/zenn-user-upload/gr5s2e4y3lbabjvuad8gr7cwhj0u)

## 2\. S3にファイルをアップロードする

- S3管理コンソールから、前の手順で作成したS3バケットを選択し、「アップロード」をクリックする  
	![](https://storage.googleapis.com/zenn-user-upload/0k27fywgglaqc6xvxyduj5eilp7z)
- 「ファイルを追加」から任意の画像ファイルを選択し、画面下の「アップロード」をクリックする。  
	![](https://storage.googleapis.com/zenn-user-upload/ocx3uld4cdwom9oecxuypibmmv5c)
- 画像がアップロードできたことを確認する  
	![](https://storage.googleapis.com/zenn-user-upload/lp69mufxnbje018wdx6g3kmrxdjj)

※ブラウザからS3上のファイルにアクセス(オブジェクトURLにアクセス)すると、下記のようにアクセスがブロックされ、バケットのパブリックアクセスがブロックされていることが確認できる  
![](https://storage.googleapis.com/zenn-user-upload/kq5fgjfh58c780haph3f2bqkrtey)

## 3\. CloudFrontディストリビューションを構築する

- CloudFront管理コンソールの「Destributins」から、「Create Destribution」をクリックする  
	![](https://storage.googleapis.com/zenn-user-upload/0qkuc354tnvaenosie8cvt92uix4)
- 「Get Started」をクリックする  
	![](https://storage.googleapis.com/zenn-user-upload/1t0byi4409pwhfrzq993kzs5bduc)
- 下記画像を参考に、各情報を入力し(赤枠以外はデフォルトでOK)、一番下の「Create Destribution」をクリック  
	![](https://storage.googleapis.com/zenn-user-upload/eqzafqogq5nh7tegynnwjbi25gci)
- CloudFront管理コンソールの「Destributins」にアクセスし、Statusが「Deployed」、Stateが「Enable」となっていることを確認する  
	![](https://storage.googleapis.com/zenn-user-upload/vb79l7f7pljcsfgnnu2ugeaq0fbt)

## 4\. ブラウザからアクセス

- ブラウザから下記のURLにアクセスする

```
"CloudfrontのDomainName"/"S3のファイル名"
```

- アクセスできることを確認  
	![](https://storage.googleapis.com/zenn-user-upload/81rtsv1mcnn8bedcgz4jcb9x13h1)

## ハマった点

- 上記の手順通りやったのにアクセスできない！！！！

## 原因

> Amazon S3 オリジンで Amazon CloudFront ディストリビューションを使用する場合、CloudFront はリクエストをデフォルトの S3 エンドポイント ( s3.amazonaws.com) に転送します。デフォルトの S3 エンドポイントは、us-east-1 リージョンにあります。バケットを作成後 24 時間以内に Amazon S3 にアクセスする必要がある場合は、このディストリビューションのオリジンドメイン名を変更します。ドメイン名には、バケットのリージョンのエンドポイントを含める必要があります。例えば、バケットが us-west-2 にある場合は、オリジンドメイン名を awsexamplebucketname.s3.amazonaws.com から awsexamplebucket.s3.us-west-2.amazonaws.com に変更することができます。

上記を要約すると、

- デフォルトのS3エンドポイントは「us-east-1」にあり、CloudFrontディストリビューションもデフォルトでは「us-east-1」のS3を参照するため、他のリージョン [^1] でS3バケットを作成した場合、リージョンの変更の伝播に時間がかかる。
- 変更がus-east-1に反映されるまでしばらくアクセスできない減少が起きる。
- しばらく待てばアクセスできるようになりますが、すぐにアクセスしたい場合、CloudFrontのOrigin Domain Nameを選択するときに下記のように書き換えるとアクセスができる。

```bash
"バケット名".s3-"リージョン名".amazonaws.com

入力例)
ashitest-s3-bucket.s3-ap-northeast-1.amazonaws.com
```

## まとめ

- S3+CloudFront連携は結構簡単に実装できる
- リージョンサービスならではの仕様について理解しておかないとハマる

## 参考文献

脚注

10

### Discussion

![](https://static.zenn.studio/images/drawing/discussion.png)

ログインするとコメントできます

[^1]: 今回はap-northeast-1で作成