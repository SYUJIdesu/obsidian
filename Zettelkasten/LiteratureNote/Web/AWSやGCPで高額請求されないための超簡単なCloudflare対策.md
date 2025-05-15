---
title: "AWSやGCPで高額請求されないための超簡単なCloudflare対策"
source: "https://zenn.dev/ikuosaito1989/articles/2b37438a708a5c"
author:
  - "[[Zenn]]"
published: 2025-05-08
created: 2025-05-15
description:
tags:
  - "web"
  - "Cloudflare"
---
10

7[tech](https://zenn.dev/tech-or-idea)

## はじめに

自分はいくつかのサービスをCloud Runで運用しており、月3000円程度で済んでいた料金が突然1日にで5000円近い請求があり、そのまま放置していたら月に10万以上の請求になってました。AWSやGCPなどの従量課金では利用に応じて料金がかかってしまうため、外部からの攻撃によって莫大な費用がかかるケースがあります。Cloudflareの例を載せますが簡単なWAFがほとんどなので同じような設定はできると思います

![image.png](https://res.cloudinary.com/zenn/image/fetch/s--YKDrWvHX--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/496802/d6584283-b91d-4149-b285-05b7579f282c.png)

## Bot対策

これらは以下の画面で有効にするだけで設定可能です

![image.png](https://res.cloudinary.com/zenn/image/fetch/s--H-YhTkKa--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/496802/fc3c8a51-7b6f-4287-a453-d2484c9ffc12.png)

## Bot Fight Mode（ボットファイトモード）

botのトラフィックを検出し、悪質なボットによるアクセスを検知・排除するためのものです。これだけでも相当数のbot対策になります

## Block AI Bots

AIクローラー（人工知能が学習や解析に使うボット）からのアクセスをブロックするためのオプションです。OpenAIなどの生成AIがクロールするbotなどを排除してくれます

## WAF（Web Application Firewall）

## IPブロック

特定のIPをブロックする方法です。有害なbotを調べる簡単な方法として\[Analytics\]->\[トップ統計\]を見るとアクセスが多いIPを調べることが可能です。やばいときは中華botから40M（4000万）アクセスが24時間以内に来てました

式は以下のように記載します

```
ip.src eq XXX.XXX.XXX.XXX
```

## 国ブロック

悪質なbotやサイバー攻撃が多い国などをブロックします。言わずもがなですが、以下の国は特に悪質なものが多いとされているため、必要がなければ問答無用でブロックしても良いと思います。

```
ip.geoip.country in {"CN" "RU" "KP" "IR"}
```

日本以外をブロック、JSチャレンジにしても良いですが、GooglebotなどのSEOに必要なbotも除外してしまうため注意が必要です

## JSチャレンジ

サイトにアクセスした際に以下のように人間か確認するチャレンジ画面が表示されます。アクセスが多いけど悪質なIPではない場合（特定のプロバイダなど）に設定すると良いと思います。  
![image.png](https://res.cloudinary.com/zenn/image/fetch/s--Pjoy191m--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/496802/a8c22c78-3d89-42d6-932d-f1d0fa9cd9ed.png)

## ブラウザ キャッシュ

高額請求を防ぐためにキャッシュも有効です。これはアプリケーションに応じて設定すれば良いと思います。私のアプリは4時間で設定しています

## Under Attack Mode

サイトにアクセスする訪問者に JavaScript のチャレンジを表示します。全部これを表示します。SEOとかも気にしないし特定のリテラシーがあるユーザーのみ利用するアプリケーションであればこれを設定しても良いかもしれません

![image.png](https://res.cloudinary.com/zenn/image/fetch/s--Pjoy191m--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/496802/a8c22c78-3d89-42d6-932d-f1d0fa9cd9ed.png)

## 最後に

ここまで基本的な設定をご紹介しましたが「自分のサービスは小規模だから攻撃なんかされないよな〜」とか思って油断して設定していなかったりすると自分のように痛い目に遭うので、サービスを公開する方は簡単なので設定してもらえると安全な開発ライフを過ごせるかと思います

10

7

### Discussion

![](https://static.zenn.studio/images/drawing/discussion.png)

ログインするとコメントできます