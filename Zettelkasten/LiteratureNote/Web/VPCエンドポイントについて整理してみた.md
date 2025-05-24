---
title: VPCエンドポイントについて整理してみた
source: https://qiita.com/charita/items/fe94680133419a03aade
author:
  - "[[charita]]"
published: 2023-02-19
created: 2025-05-22
description: VPCエンドポイントとは？VPC内のサービスとVPC外のサービスとのプライベート接続を提供するコンポーネントのこと。セキュリティ上の問題でインターネットに接続せずにVPC外のサービスと繋がりたい…
tags:
  - web
  - AWS
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

[@charita](https://qiita.com/charita)

投稿日

## VPCエンドポイントとは？

VPC内のサービスとVPC外のサービスとのプライベート接続を提供するコンポーネントのこと。  
セキュリティ上の問題でインターネットに接続せずにVPC外のサービスと繋がりたいときなどに使用する。  
サービス利用側のVPC（S3にアクセスしたいEC2インスタンス、API Gatewayを叩きたいEC2インスタンスなどが設置されているVPC）内に作成する。

上記例の場合、逆にEC2インスタンスがパブリックサブネットにあれば、VPCエンドポイントを使用せずインターネットゲートウェイ経由で接続できる。  
※パブリックサブネットにあっても、VPCエンドポイントが設置されている場合は強制的にVPCエンドポイント経由での接続となるため、同一VPC内で複数サービスが動いている場合、作成時には他のサービスへの影響がないかしっかりと確認する必要がある。

VPCエンドポイントには2種類ある。（正確には3種類で、Gateway Load Balancerエンドポイントというものも存在するが、メインの2種のみ触れる）

以下詳細。  
※とりあえず急ぎ理解する必要があったインターフェース型のみ記載。ゲートウェイ型はまた今度。

## インターフェース型

通常、AWSのAPIを実行する場合は、そのサービスエンドポイントにリクエスト（多くはHTTPSのみ）が到達する必要がある。

> サービスエンドポイント  
> [https://docs.aws.amazon.com/ja\_jp/general/latest/gr/rande.html](https://docs.aws.amazon.com/ja_jp/general/latest/gr/rande.html)  
> [https://docs.aws.amazon.com/ja\_jp/general/latest/gr/aws-service-information.html](https://docs.aws.amazon.com/ja_jp/general/latest/gr/aws-service-information.html)

このサービスエンドポイントはグローバルIPであり、つまりはAWSサービスのエンドポイントに到達するためには、グローバルIPでの通信が必要である。

以下のように、例えばS3のサービスエンドポイントで確認してみると、グローバルIPを持っていることが分かる。

```text
charita:~/environment $ dig +short s3.ap-northeast-1.amazonaws.com
52.219.16.194
52.219.136.98
52.219.68.182
52.219.136.54
52.219.200.0
52.219.152.164
52.219.136.118
52.219.196.92
```

ただし冒頭記載の通り、セキュリティ上の問題でインターネットに接続せずにサービスと繋がりたい場合などに、プライベートサブネット内のEC2インスタンスがVPC外のサービスと通信するための手段の1つがインターフェース型のVPCエンドポイントである。  
※NATゲートウェイの利用も手段の1つではあるが、通信がインターネットを経由するため、特に要件がなければセキュリティ観点的にはVPCエンドポイントを使用することが推奨される。

インターフェース型のVPCエンドポイントは、PrivateLinkと呼ばれるサービスを使って、各サービスを接続させることができる。

### ★PrivateLink

PrivateLinkの実体はVPC内のENI(Elastic Network Interface) のこと。  
PrivateLink自体がIPアドレスを持ってVPCの中にエンドポイントができる。  
より正確には、サブネット内にENIを立ち上げ、そこをサービスのエンドポイントとする。（ENIのIPはサブネットのIPレンジの中から自動的に採番される）

つまり、サービスエンドポイントのグローバルIPではなく、ENIが持つプライベートIPと通信することでVPC外部のサービスとの接続を可能とするもの。  
この時、PrivateLinkを使用した通信はインターネット上に公開されることはなく、VPC エンドポイントを通過する通信は外部から干渉されない。（NATゲートウェイとの違いはここ）

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1131391/400d5e69-7ddb-4a59-8823-0fd43db69d89.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1131391%2F400d5e69-7ddb-4a59-8823-0fd43db69d89.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=33e31fc79b1b37f5fbd9adc85a17ede4)  
上記の図で考えると、EC2インスタンスから見ればENIとプライベートIPアドレスで通信するだけで、インターフェースエンドポイントに到達したリクエストはサービスエンドポイントに到達する。

では、インターフェースエンドポイントに到達したリクエストはどのように名前解決をしてVPC外サービスのエンドポイントに到達するのか。（プライベートIPのENIからグローバルIPのs3.ap-northeast-1.amazonaws.comへの名前解決）

一般的な手段はプライベートDNS名を使用することである。（デフォルトで有効）  
プライベートDNS名を有効にすると、以下の状態になる。

- サービスエンドポイント（=VPC外サービス）のプライベートホストゾーンを該当VPCに関連付ける
- ホストゾーンには以下を可能にするレコードセットが含まれている
	- 「デフォルトのパブリックなサービスエンドポイントDNS名を指定したリクエスト」がサービス（VPC内のリクエスト送信側）へのプライベートネットワーク接続を行うこと
		- 例）s3.ap-northeast-1.amazonaws.comへのリクエストがプライベートネットワークを使用して接続する

### ★プライベートホストゾーン

ホストゾーンとは、ドメインおよびサブドメインのトラフィックのルーティングする方法についての情報を保持するコンテナを指す。  
ホストゾーンには、下記の2種類がある。

1. パブリックホストゾーン
	1. インターネット上でどのようにルーティングするかを指定する DNS レコードが含まれる
2. プライベートホストゾーン
	1. VPC 内でどのようにルーティングするかを指定する DNS レコードが含まれる  
		[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1131391/b8bbe468-e4c1-dbb0-f0bc-40f5cbd5e71e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1131391%2Fb8bbe468-e4c1-dbb0-f0bc-40f5cbd5e71e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6b8ab9e2aad3a50bd165781b4ab7a497)

つまりプライベートホストゾーンとは、インターネット上からはアクセスできないが、同一VPC内部からであればアクセスできるドメインのレコードを管理するコンテナのこと。  
そして、「プライベートホストゾーンをVPCに関連付ける」というのは、異なるアカウント間（サービス提供者としてのAWSを別アカウントのユーザーと捉えている）でのVPC間のプライベートIPでの通信を可能にするということ。

実際にインターフェース型かつプライベートDNSが有効になっているエンドポイントで確認してみると、VPC内部と外部で名前解決をしたときに返るIPは異なるはず。

VPC外

```text
charita-out-of-vpc:~/environment $ dig +short monitoring.ap-northeast-1.amazonaws.com
52.119.219.193
```

VPC内

```text
charita-in-vpc:~/environment $ dig +short monitoring.ap-northeast-1.amazonaws.com
172.16.2.49
```

このVPCエンドポイントにアタッチされているENIのプライベートIPを確認すると「172.16.2.49」であり、VPC内から名前解決した結果と同じものとなっている。  
つまり、CloudWatchのサービスエンドポイント（グローバルIP）へのDNSクエリに対して「インターフェースエンドポイントのプライベート IP 」が返却されている。

このようにして、インターフェースエンドポイントに到達したリクエストは名前解決されてサービスエンドポイントに到達する。  
また、インターフェースエンドポイントには他のENI同様、SecuriryGroupを関連づけることができるが、そのアウトバウンドは考慮する必要がない。  
アウトバウンド通信を許可していなくても、つまりAWSサービスエンドポイント（上記例でいればCloudWatch）への通信を意識しなくともよしなにやってくれる。

[0](https://qiita.com/charita/items/#comments)

コメント一覧へ移動

[21](https://qiita.com/charita/items/fe94680133419a03aade/likers)

いいねしたユーザー一覧へ移動

9

ストックを更新しました