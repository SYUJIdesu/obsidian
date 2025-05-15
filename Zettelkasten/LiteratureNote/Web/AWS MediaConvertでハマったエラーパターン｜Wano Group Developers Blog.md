---
title: "AWS MediaConvertでハマったエラーパターン｜Wano Group Developers Blog"
source: "https://developers.wano.co.jp/1495/"
author:
  - "[[Wano Group Developers Blog]]"
published:
created: 2025-05-15
description: "AWS MediaConvertを使う上でハマったエラーや制限等に対する備忘録。"
tags:
  - "web"
  - "AWS"
  - "MediaConvert"
---
## AWS MediaConvertでハマったエラーパターン

naoto

2018.10.25

![](https://developers.wano.co.jp/wp-content/uploads/2018/10/download.png)  
AWSによるElemental買収後に追加された Media Service群。  
その中でも [MediaConvert](https://aws.amazon.com/jp/mediaconvert/) は、動画/音楽メディアのエンコードを担当するサービス。  
ProresやHEVCの出力等にも対応しており、実質Elastic Transcoderの上位互換サービスという扱いになっている。

動画や音楽をエンコードするだけならコンテナとffmpegで自前で実装するのも柔軟かつ安いので良いが、「権利関係の問題に安心が置ける/エンコード自体が安定してて楽…」という理由から弊社ではそれなりに使用頻度が高い。

このMediaConvertを使っている中で悩まされたパターンを備忘的にメモ。

## 各種サイズ指定は奇数に弱い

```javascript
BadRequestException: /outputGroups/0/outputs/0/videoDescription/position/x: Should be a multiple of 2 status code: 400, request id: xxxxx9b-d678-11e8-9d7e-fb8xcccccc
```

そのままの意味で、Crop, Position, (Output)Width, (Output)Heightなどサイズに関わる設定項目は、基本的に出力コーデック問わず奇数ピクセルでの設定ができない。  
APIを叩いた時点でエラーが起きる。  
\-1 するなりして偶数にしましょう。

## クロッピングや出力ポジショニングは「縁ぴったりの値」に弱い

MediaConvertは動画ソースの一部だけを切り取って、出力する動画の任意の地点に任意の大きさで表示する機能がある。  
この機能を使って、動画の黒帯(レターボックス/ピラーボックス)などを消すクロップ処理を書くケースがあった。  
クロップすべき領域があろうがなかろうが共通でCrop指定をjob実行命令に付与していたが、「ソースのサイズと同じサイズのクロッピング」はそもそもできなかった。

```javascript
Position rectangle [720,480,0,0] lies outside output resolution [640x426] for video_description [1].
```

これはjobのput自体は成功するが、実行後にエラーとなる。  
対策としては単純に、「ソースのサイズと同じサイズのクロッピング」では該当箇所をnullにして送ること。

## userMetadataは 各キー255B x 10 まで

AWS Batch や MediaConvertの複数回使用も絡めた複雑なワークフローを StepFunctionsで組んでいた際、 [Activity](https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/concepts-activities.html) のToken をMediaConvertのjobのuserMetadataに付与し、エンコードが終わったらCloudwatch Event + Lambda でStepFunctionsのTaskのdoneを叩く….という機構を書いていた。

しかし ActivityのTokenは1kbくらいあり、この長すぎるTokenは1つのuserMetadataのvalueとしては乗らず、結構困ってしまった。

対策としては、  
(1)大人しくS3やDynamo等にTokenの実体を移し、userMetadataでは参照だけ保持する  
(2)複数のuserMetadataにtokenを分割して載せる、みたいな闇の実装を書く

あたりを検討した。　　  
  
…が、その後　StepFunctions　自体の使用を一時取りやめることになったので特には問題ならなくなった。  
同じような機構を書くときには(1)を採用するかも。  
(StepFunctionsとBatchやMediaConvertの連携の構成がもっと綺麗に定義できるといいなあ)

## 動画の結合とクロッピングを同時に行うとエラー

3動画の結合とソースクロッピングを1回のエンコードで済まそうとしたときに、Elementalのjobがずっと実行状態になり、中断もできないというエラーがあった。  
(これは問い合わせもしたので、現在は対応されている可能性がある)

---

以上、使っていく上で色々あったが、なにかあるたびにたまに追記する。

[« O2Oの効果測定をGoogle Analyticsで行う](https://developers.wano.co.jp/1492/) [database/sqlのコネクション周りの実装をちゃんと調べた »](https://developers.wano.co.jp/1420/)

## About

[

![Wano Group Media](https://developers.wano.co.jp/wp-content/themes/wgd/common/images/wgm_banner.png)

](https://group.wano.co.jp/)

Wanoグループのエンジニアやデザイナーが使っている技術、興味のある技術、(色んな意味で)はまっている技術などをお伝えしていきます。