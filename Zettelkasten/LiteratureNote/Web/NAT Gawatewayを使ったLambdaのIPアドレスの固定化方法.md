---
title: NAT Gawatewayを使ったLambdaのIPアドレスの固定化方法
source: https://qiita.com/Shinkijigyo_no_Hitsuji/items/06b25f6f7052b852cade
author:
  - "[[Shinkijigyo_no_Hitsuji]]"
published: 2022-10-20
created: 2025-05-22
description: はじめに株式会社マイスター・ギルド新規事業部のヒツジーです。弊社新規事業部では、新規サービスの立ち上げを目指して日々、アイディアの検証やプロトタイプの作成などを行っています！技術の進歩は目…
tags:
  - web
  - AWS
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/6ec16fbf-ad49-b975-cf6a-642ab0476747.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F6ec16fbf-ad49-b975-cf6a-642ab0476747.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=949bb1f704fdbd2ecf4d5047be2d2282)

## はじめに

株式会社マイスター・ギルド新規事業部のヒツジーです。  
弊社新規事業部では、新規サービスの立ち上げを目指して  
日々、アイディアの検証やプロトタイプの作成などを行っています！

技術の進歩は目覚ましいので、置いてかれないように新しい技術のキャッチアップにもいそしんでいます！  
本記事は、以前書いた [FastAPI with Lambda + API Gatewayでサーバレスアプリケーションを作成する方法](https://qiita.com/Shinkijigyo_no_Hitsuji/items/cedd1825e5437663d3ce) の続編として、AWSのサービスであるNAT Gawatewayの使い方についてご紹介します。

アクセス制限がかかっている外部サーバーのDBにAWS Lambdaからアクセスするには、IPアドレスを固定化しないといけませんが、NAT Gatewayを用いることで簡単にIPアドレスを固定化できます。

たまに必要となるような知見ですので、こちらにまとめておきたいと思います。

## なぜIPアドレスを固定化しないといけないのか？

外部サーバーのDBでは特定のIPアドレスからしかアクセスできないように制限をかけていることが多いですが、LambdaはIPアドレスが固定化されていないので、LambdaのIPアドレスを調べて、アクセス許可をしたところで、次のアクセスの際にはIPアドレスが変わってしまっているという事態が発生します。  
なので、IPアドレスを固定化する必要があります。

Lambdaに限らず、AWSは動的にグローバルIPアドレスが変更されます。  
インスタンスを再起動するたびに変更されるものであると覚えておくといいと思います。

## IPアドレスの固定化の概要

LambdaとAWS外のDBとの間にNAT GatewayというAWSのサービスを介在させることで、IPアドレスの固定化が実現します。  
NAT GatewayがElastic IPアドレスという固定化されたグローバルIPアドレスを発行してくれるからです。

## NAT Gatewayの概要

NAT GawatewayはAWSのVPCサービスに付随するサービスです。  
NAT Gatewayでは時間単位の料金とデータ処理の料金を合計した金額が請求されます。  
起動したタイミングから料金が発生しますのでご注意ください。

[アジアパシフィック（東京）リージョンでのNAT Gatewayの料金](https://aws.amazon.com/jp/vpc/pricing/) は

- 時間あたりにかかる料金　0.062 USD/時
- 処理データにかかる料金　0.062 USD/GB

となります。

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/8c09934a-ce12-302c-f1d2-f73a1bbd7bf8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F8c09934a-ce12-302c-f1d2-f73a1bbd7bf8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=82585a249c93bcb8b8a656e7dbd447c8)

## 設定手順

以下の図のようなシステム構成を実現させることを目標とします。  
[![システム構成.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/d47b5ac3-fd8d-0493-0d70-906a421c8cfd.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2Fd47b5ac3-fd8d-0493-0d70-906a421c8cfd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b50d73652fe8da8860fadeff76c41814)

このシステム構成にするためには、以下のような手順を踏む必要があります。

1. Internet Gatewayの作成
2. Public Subnetの作成
3. Subnetに対するInternet Gatewayの割り当て
4. Private Subnetの作成とLambdaとの紐づけ
5. NAT Gatewayの作成
6. Private Subnetに対するNAT Gatewayの割り当て

まずは、Internet gawatewayを使ってpublicなsubnetを用意し、次にprivate subnetを用意、最後にそれらのsubnetを紐づけることをします。以下で、順次解説します。

## 具体的な設定方法

以下ではアジアパシフィック（東京）リージョンで設定を行っていきます。  
AWSのサービス画面ヘッダー右上でリージョンをあらかじめご確認ください。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/e6413a17-1c84-f6b5-4894-6d7140dfa36c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2Fe6413a17-1c84-f6b5-4894-6d7140dfa36c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ed536f85bf2f9c6d8211e5bdf0ff57e4)

以下の設定はすべてVPCサービス内で完結しますので、VPCのサービス画面を開いてください。  
検索ボックスで「VPC」で検索すると出てきます。  
あらかじめVPCを作成しておいてください。  
[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/0c122fb2-4d70-5a0c-eb69-664ca1241acc.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F0c122fb2-4d70-5a0c-eb69-664ca1241acc.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=49972207bc4268e7671fb88c28bb2550)

### Internet Gatewayの作成

- VPCのホーム画面の左サイドバー内もしくは、リージョン別のリソースで「インターネットゲートウェイ」を選択します。  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/7d718022-b1c9-6a03-930b-5bbf9a323a29.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F7d718022-b1c9-6a03-930b-5bbf9a323a29.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a33cf9ac45f40944a7f25e0dd60e2a7b)
- 次に右上の「インターネットゲートウェイを作成」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/13835eb8-88ee-7078-ffe6-7c04280972b6.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F13835eb8-88ee-7078-ffe6-7c04280972b6.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3503eb431fbfdee68946823b131d3e28)
- 名前タグをつけた後に、「インターネットゲートウェイを作成」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/9011a539-44c7-5f52-de46-cdc904f44eda.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F9011a539-44c7-5f52-de46-cdc904f44eda.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b136749666e9a7b134ed82dea7659bb0)

これでインターネットゲートウェイの作成は完了です。

### Public Subnetの作成

- VPCのホーム画面の左サイドバー内もしくは、リージョン別のリソースで「サブネット」を選択します。  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/1db41553-53cf-cf52-9173-f8067dffe275.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F1db41553-53cf-cf52-9173-f8067dffe275.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d35f356bd14aa6b539f175a237ad6f8f)
- 右上の「サブネットの作成」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/5d7e085a-ec65-1b58-4f88-ac22bb454923.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F5d7e085a-ec65-1b58-4f88-ac22bb454923.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c5e3ede07c6700c891a30c8cc04f0bd5)
- VPC IDで作成済みのVPCを選択します  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/c826dd40-997f-49e7-1216-5f445c9699af.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2Fc826dd40-997f-49e7-1216-5f445c9699af.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b0761ac238dcdb6b693af911d7d32e11)
- サブネット名はpublicであることがわかりやすいように「public-xxxxx」のような名前を付けましょう
- IpV4 CIDRブロックは、紐づいているVPCの範囲内に収まるように設定しましょう
- 「サブネットを作成」ボタンをクリック

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/2e9468ed-d658-0e5f-fe33-7e97c628cd25.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F2e9468ed-d658-0e5f-fe33-7e97c628cd25.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5ed6da18734eb4863809242ba3c054e9)

### Subnetに対するInternet Gatewayの割り当て

↑で作成したサブネットは現状ではpublicにはなっていませんので、PublicにするためにInternet gatewayを割り当てる必要があります。

- 先ほど作成したサブネットの詳細ページを開き、「ルートテーブル」を確認します。
- 送信先「0.0.0.0/0」がInternet gatewayになりますが、それを追加する設定を次に行います  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/e4451d68-8dac-bf2e-11c9-23620d48201f.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2Fe4451d68-8dac-bf2e-11c9-23620d48201f.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=802853107f5dcb95ce4885033bcd1907)
- VPCのホーム画面の左サイドバー内で「ルートテーブル」を選択します
- 「ルートテーブルを作成」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/b5016682-35b7-e578-a203-e3aa1e827697.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2Fb5016682-35b7-e578-a203-e3aa1e827697.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c0b07b5ac31bbd4c58eaa014f2784210)
- publicであることがわかるような名前「public-routing-xxxx」をつけるといいでしょう
- VPCはすでに作成してあるものを選択します
- 「ルートテーブルを作成」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/678988f6-42c0-48f9-173b-0d28046de635.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F678988f6-42c0-48f9-173b-0d28046de635.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=62fdde0def0b26ec4bcf64337dbb1db5)
- 作成したルートテーブルの詳細画面にて、「ルートを編集」をクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/94a4d29c-1b78-3196-f711-f7df5aff00d9.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F94a4d29c-1b78-3196-f711-f7df5aff00d9.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4f9eb18572b0db70e8dc8d2fba9a6f55)
- 送信先に「0.0.0.0/0」を入力後、ターゲットで「インターネットゲートウェイ」を選択し、「変更を保存」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/2df69f43-eceb-bc18-ec8f-c6fa92e4524d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F2df69f43-eceb-bc18-ec8f-c6fa92e4524d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8fb8f15c7f2ab809a63b51474b3fe601)

これで、作成したサブネットに対してインターネットゲートウェイが割り当たったので、publicになりました。  
このサブネット内にNAT Gatewayを作成することで、NAT Gatewayはinternet gatewayを通して、外部のDBへアクセスすることができるようになります。  
NAT Gatewayの設定はのちほど。

### Private Subnetの作成とLambdaとの紐づけ

- public subnetを作成したときと同様の方法で作成します。サブネット名はprivateであることがわかりやすいように「private-xxxxx」のような名前を付けましょう
- IpV4 CIDRブロックは、紐づいているVPCの範囲内に収まるように設定しましょう

このprivate subnet内にLambdaを配置します。  
Lambdaはprivate subnet内にあるので、直接外部とは接続されていない状態となります。

- Lambdaのサービスページを開き、作成済みのFunctionを開きます。
- 「設定」の中の「VPC」を開き、右側の「編集」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/0c690947-8eeb-2576-9c9b-437b6428c094.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F0c690947-8eeb-2576-9c9b-437b6428c094.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=66ec883c9691408d10b241d03141ce97)
- VPC、サブネット、セキュリティグループを選択し、「保存」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/b34d961a-4f57-e29a-f2b8-6374d1657e0b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2Fb34d961a-4f57-e29a-f2b8-6374d1657e0b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e4f4ba7b25c0f886b1267c41e7b6c842)

サブネットは間違ってpublicを選択しないようにしましょう。  
これで、作成したprivate subnet内にLambdaを配置できました。

### NAT Gatewayの作成

ここでやっと登場しました。NAT Gatewayです。

- VPCのホーム画面の左サイドバー内もしくは、リージョン別のリソースで「NATゲートウェイ」を選択します。  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/0c819eae-c3c8-7033-d37e-de6efabd4402.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F0c819eae-c3c8-7033-d37e-de6efabd4402.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3c714c88832e900ea2ae25bad2ccdc62)
- 「NATゲートウェイを作成」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/5cb7a21f-bf0b-5795-6a40-aafe68d4e204.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F5cb7a21f-bf0b-5795-6a40-aafe68d4e204.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=545c1e0ab3acb2d3c293be3095c7e207)
- 名前を入力
- ↑で作成したpublicなサブネットを選択
- 接続タイプは「パブリック」を選択
- 「Elastic IPを割り当て」をクリックした後に「NATゲートウェイを作成」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/e2ec5a34-d51b-53bd-13b1-71722899be4f.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2Fe2ec5a34-d51b-53bd-13b1-71722899be4f.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a9dd20f67ecdc094fd30939f2cd54089)

これで、publicサブネット内にNAT Gatewayを作成することができました。  
Elastic IPアドレスの割り当てもできました。

### Private Subnetに対するNAT Gatewayの割り当て

ここまでの手続きを踏むと、NAT GatewayとLambdaの設定はできましたが、それぞれが結びついていません。  
最後にPrivate Subnet（Lambda）にNAT Gatewayを割り当てます。  
↑で（public）サブネットに対してInternet Gatewayを割り当てたように、今回はPrivateサブネットに対してNAT Gatewayを割り当てます。

- VPCのホーム画面の左サイドバー内で「ルートテーブル」を選択します
- 「ルートテーブルを作成」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/d2a68c29-3cf7-de34-d9ba-ffc282f61c89.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2Fd2a68c29-3cf7-de34-d9ba-ffc282f61c89.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4eb63ca26d00e52d5691d9796667f11d)
- privateであることがわかるような名前「private-routing-xxxx」をつけるといいでしょう
- VPCはすでに作成してあるものを選択します
- 「ルートテーブルを作成」ボタンをクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/972e0da2-452c-e315-9e6c-faec734ecb34.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F972e0da2-452c-e315-9e6c-faec734ecb34.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5705736631266a871460661e3d5a9475)
- 作成したルートテーブルの詳細画面にて、「ルートを編集」をクリック  
	[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/7b77d3bc-d8f3-8ffc-d0ca-b18f94918f8d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F7b77d3bc-d8f3-8ffc-d0ca-b18f94918f8d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bc69ad99232410258ece488fe3f744e1)
- 送信先に「0.0.0.0/0」を入力後、ターゲットで「NATゲートウェイ」を選択し、「変更を保存」ボタンをクリック

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/556658/65afff93-fb17-cd79-40bc-b1cf5c54afb1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F556658%2F65afff93-fb17-cd79-40bc-b1cf5c54afb1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=583092076436b4569e41313d72e84747)

以上で完了です！

[0](https://qiita.com/Shinkijigyo_no_Hitsuji/items/#comments)

コメント一覧へ移動

[22](https://qiita.com/Shinkijigyo_no_Hitsuji/items/06b25f6f7052b852cade/likers)

いいねしたユーザー一覧へ移動

8

ストックを更新しました