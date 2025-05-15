---
title: "AWS Elemental MediaConvertを使った動画自動変換サービスを構築した備忘録"
source: "https://blog.cloudsmith.co.jp/2022/06/182/"
author:
  - "[[エンジニアBLOG]]"
published: 2022-06-01
created: 2025-05-15
description: "スマートフォンで撮影された動画を自動変換してストリーミング再生させるサービスの構築方法と必要な知識について説明します。"
tags:
  - "web"
  - "AWS"
  - "MediaConvert"
---
こんにちは！

今回はスマートフォンで撮影された動画を自動変換してストリーミング再生させるサービスの構築方法と  
必要な知識について、備忘録も兼ねて箇条書きにしていきたいと思っています。  
（具体的にここをこう設定しよう！という内容ではありません）

## 今回の要件

![](https://wp.cloudsmith.co.jp/wp-content/uploads/2022/06/Untitled-Diagram.drawio-1-640wri.png)

iPhoneやAndroidで撮影された動画を自動で変換し、  
PC、iPhone、Androidでストリーミング再生ができるようにします。

## 必要なこと

### 動画ファイルの形式について知る

iPhoneで撮影した動画の拡張子は.MOV（video/quicktime）

- Androidで撮影した動画の拡張子は.mp4（video/mp4）
- ストリーミングする動画はHLS形式（HTTP Live Streaming）  
	  
	→ MOVとMP4をどちらもHLS形式にする必要がある！  
	  
	HLS形式については、以下のページが詳しいです。  
	参考： [HTTPライブストリーミングとは？| HLSストリーミング – Cloudflare](https://www.cloudflare.com/ja-jp/learning/video/what-is-http-live-streaming/)

### 動画を変換する方法の調査

- [Elemental MediaConvert](https://aws.amazon.com/jp/mediaconvert/) を使用して変換を行う
- S3バケットではファイルがアップロードされたとき、イベントを送信できる
- Lambdaでイベントをキャッチし、MediaConvertを起動
- MediaConvertではあらかじめ「何をどの様に変換するか」のテンプレートを作成しておき、Lambdaでそれを指定する
- 変換された動画は別のS3バケットに保存する  
	同じバケットでも構わないが、設定によっては変換後のファイルが再帰的に変換され、利用料金が高額になる場合があるため注意が必要

### 各デバイスでストリーミング動画を再生するには？

- PC、iPhone、AndroidのブラウザではiPhoneのみHLS形式の再生がネイティブで可能
- 再生できないものでは [HLS.js](https://github.com/video-dev/hls.js/) というクライアントライブラリを用いて再生する

### セキュリティ関連

①変換前の動画はS3で直接見られないようにする必要がある（アップロードだけできるようにする）

②変換後の動画もS3で直接見られないようにする（ただし、CloudFrontからはアクセス可能）

①②はS3のバケットポリシー等で設定（CloudFrontにはCORSの設定をする必要がある）

[参考：CloudFront から「リクエストされたリソースに「Access-Control-Allow-Origin」ヘッダーが存在しません」というエラーを解決するにはどうすればよいですか? – AWSサポート](https://aws.amazon.com/jp/premiumsupport/knowledge-center/no-access-control-allow-origin-error/)

CloudFrontを認証必須にするには、以下の２通りの方法がある

- 署名付きURL
- 署名付きCookie

ただし、変換後のHLS形式の動画はマニフェストファイルと分割された複数動画ファイルの形になっており、  
分割された動画ファイルをマニフェストファイルをもとに取得する必要があるため、 **署名付きCookie** を使用する必要がある。

署名付きCookieを使用する場合、CORSの問題があるためCloudFrontも同ドメインである必要がある

[参考：AWS SDK for Javaを使ってカスタムポリシーを使用した署名付きCookie（CloudFront用）を設定する – Qiita](https://qiita.com/hmatsu47/items/8edd5cfde92bfd0d5a6c)

### 料金の計算

①アップロードされる動画のサイズ×本数、どのくらいの期間保存するのか

- S3（ストレージ代）
- Lambda（実行回数）
- MediaConvert（変換時間、回数など）

②変換後の動画のサイズ×本数

- S3（ストレージ代）
- CloudFront（リクエスト数、通信量など。HLSでのファイル分割数にもよる）

③スマートフォンでも再生する場合、スマートフォンでのパケット代など…

以上、調査が必要な項目数は多いですが、何を調べれば良いのか分かっていればある程度は楽になると思います。  
これからも、やったことは箇条書きでも良いので残しておきノウハウを貯めていきたいですね。