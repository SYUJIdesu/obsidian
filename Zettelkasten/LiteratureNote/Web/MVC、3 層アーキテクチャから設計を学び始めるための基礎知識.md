---
title: MVC、3 層アーキテクチャから設計を学び始めるための基礎知識
source: https://qiita.com/os1ma/items/7a229585ebdd8b7d86c2
author:
  - "[[os1ma]]"
published: 2019-06-21
created: 2025-05-22
description: はじめにアプケーション・アーキテクチャについて学ぶと「MVC」や「3 層アーキテクチャ」といった言葉にたどり着きます。さらに勉強を進めると「MVVM」、「ドメインモデル」、「クリーンアーキテクチ…
tags:
  - web
  - アーキテクチャ
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から3年以上が経過しています。

## はじめに

アプケーション・アーキテクチャについて学ぶと「MVC」や「3 層アーキテクチャ」といった言葉にたどり着きます。  
さらに勉強を進めると「MVVM」、「ドメインモデル」、「クリーンアーキテクチャ」など、よく分からない言葉がどんどん増えていきます。  
また、「オブジェクト指向」を勉強しても、実際のアプリケーションでの使いどころが分からなかったりします。

この記事では、これらの用語の非常に分かりにくい関係を整理しました。

## 3 層アーキテクチャ

## 2 種類の 3 層

伝統的な Web アプリケーションは、以下のように 3 種類のサーバから成り立ちます。

[![1 (1).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/216010/9b718e12-8520-580e-a6b0-53e5a5475adb.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F216010%2F9b718e12-8520-580e-a6b0-53e5a5475adb.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6072ff84b6923e5257094293b87516c1)

このサーバ構成を 3 層アーキテクチャと言うことがあります。

一方、アプリケーションサーバで動いているプログラムの内部構造も、以下のように 3 層に分離することがあります。

[![1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/216010/082f0516-dbb4-888c-a286-d64d4bbc3909.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F216010%2F082f0516-dbb4-888c-a286-d64d4bbc3909.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=47ea547522c653b27bfb59a4025a1fb3)

これも 3 層アーキテクチャと言うことがあります。

この記事では、サーバの構成ではなく、 **アプリケーションサーバ内で動くプログラムの構造** に関する用語を説明していきます。

以後、この記事で「3 層アーキテクチャ」と言うときは、後者の (アプリケーション内の) 3 層アーキテクチャを指します。

## アプリケーション・アーキテクチャの基本は 3 層

Web アプリケーションなどのプログラムの構成を考えるとき、一番基本になるのは以下の 3 層に分割することです。

- プレゼンテーション層 (またはユーザインターフェース層)
- ビジネスロジック層 (またはアプリケーション層)
- データアクセス層

[![1 (2).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/216010/3d0e9eaf-5f7d-9994-0c73-8b6dd6eb62bc.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F216010%2F3d0e9eaf-5f7d-9994-0c73-8b6dd6eb62bc.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=dd10dfa7cfd420bf72a43aac40cd24bc)

例えば、じゃんけんアプリケーションを作るとします。

**ユーザとやりとりするのがプレゼンテーション層** です。

「グーがチョキに勝ち、チョキがパーに勝ち、パーにグーが勝つ」などの **ルールを持つのがビジネスロジック層** です。

ジャンケンの **結果を保存するのであれば、それはデータアクセス層の仕事** です。

**CLI アプリだろうが Web アプリだろうが、それはプレゼンテーション層が異なるだけ** です。  
また、 **データの保存先がファイルだろうが DB だろうが、それはデータアクセス層が異なるだけ** です。

「表示上の関心と、コアなルールと、データの永続化を分離する」という 3 層が何より基本です。

## プレゼンテーション層

[![1 (3).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/216010/a97e3c11-bcd4-ea7e-3d52-d2f5d2cb1ed4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F216010%2Fa97e3c11-bcd4-ea7e-3d52-d2f5d2cb1ed4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c9af7ab84a532cc5166299b11b3cbe2d)

## MVC と 3 層はどっちをやればいいの??

さて、3 層が基本と言いましたが、「MVC (Model View Controller)」という構成もよく耳にすると思います。  
MVC と 3 層アーキテクチャでは、どちらを選択すればいいのでしょうか?

MVC と 3 層アーキテクチャの関係を図にすると、以下のようになります。

[![1 (6).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/216010/390519b5-6e5c-0a91-c370-26c3bd5bcc38.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F216010%2F390519b5-6e5c-0a91-c370-26c3bd5bcc38.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=55ca4f9a035e99a1d7bff414d9081d5f)

つまり、3 層アーキテクチャと比較して、 **プレゼンテーション層** 周辺に着目しているのが MVC です。

ウィキペディアの [Model\_View\_Controller](https://ja.wikipedia.org/wiki/Model_View_Controller) のページにも、

> MVC（Model View Controller モデル・ビュー・コントローラ）は、ユーザーインタフェースをもつアプリケーションソフトウェアを実装するためのデザインパターンである。

と書かれています。  
実は、MVC、MVP、MVVM といったものは、全て **プレゼンテーション層のアーキテクチャ** なのです。

なので、アプリケーションの構成を検討するときに、「MVC にするぜ」というだけでなく、「 **全体としては 3 層で、プレゼンテーション層は MVC にする** 」という話になるわけです。

ちなみに、上図から明らかだと思いますが、MVC で言う Model [^1] は、ただのデータの入れ物ではありません。 **Model はビジネスロジックを書くところ** です。

## ビジネスロジック層

[![1 (4).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/216010/6d0da194-2c1c-5fe6-1a71-bcd27e329ec6.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F216010%2F6d0da194-2c1c-5fe6-1a71-bcd27e329ec6.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=279305fa5c20e9dad814cd03e35287ea)

## ビジネスロジックは「サービス」に書くとは限らない

さて、次は 3 層の真ん中の、ビジネスロジック層に注目します。

ビジネスロジック層には、大きく次の 2 種類の実装方法があります。

- トランザクションスクリプト
- ドメインモデル

**トランザクションスクリプトは、いわゆる手続き型プログラミングを使って実装する方式** です。  
**データと getter、setter だけを持つような DTO といった入れ物と「サービス」クラスを作成し、サービスに処理を書く** のが定番です。

一方で、 **ドメインモデルは、データを持つクラスに処理も書くという、オブジェクト指向プログラミングで実装する方式** です。

## ここで言う「オブジェクト指向」とは?

オブジェクト指向プログラミングと手続き型プログラミングでは、「データ」と「処理」が一緒にいるかどうかが大きく異なります。

ドメインモデル方式では、オブジェクト指向で実装するため、「データ」を内部に隠蔽したオブジェクトに対し「処理」を命令するようなコードになります。

Java などのオブジェクト指向言語を使っていて、DTO を new していても、それはオブジェクト指向ではありません。  
DTO にデータだけを持たせて処理はサービスに記述するのではなく、 **データと処理を同じクラスに書くのがオブジェクト指向プログラミングであり、ドメインモデルという実装方式です** 。

ドメインモデル方式はトランザクションスクリプトと比較して、学習コストや初期実装コストが高い代わりに、長期的に変更に強いアプリケーションになるとされています。

## ドメインモデルの場合は 4 層になることも

ドメインモデルの実装例は、DDD という手法の中でもまとめられています。

多くの例では、 **ビジネスロジック層を「アプリケーション層」と「ドメイン層」の 2 つに分離** しています。

[![1 (7).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/216010/7dbd36a4-8083-c73c-d976-35a6105aaede.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F216010%2F7dbd36a4-8083-c73c-d976-35a6105aaede.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=10921cbd70e9bbb2ecec465281c5f747)

**ドメイン層は、「グーがチョキに勝つ、チョキがパーに勝つ、パーがグーに勝つ」といったコアなロジックを持つ層であり、ピュアなオブジェクト指向モデルの世界として実装されます** 。  
これがいわゆる「オブジェクト指向により現実世界をプログラムに反映する」というやつです。 [^2]

これに対して、アプリケーション層は以下のような役割を担います。

- ドメイン層のオブジェクトたち (ドメインモデル) に対する操作で **ユースケースを実現** する
- ピュアなオブジェクト指向モデルの世界に入れたくない、 **トランザクション管理のような、アプリケーション都合のロジックを実現する**

## データアクセス層

[![1 (5).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/216010/dfc4d798-ae45-a3e4-0ab7-df3b918b3df4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F216010%2Fdfc4d798-ae45-a3e4-0ab7-df3b918b3df4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=cada90887202a374706449afbeaf737a)

## データアクセス層は、コードを書かない場合もある

データアクセス層は DB などにデータを保存する責務を持つ層ですが、O/R マッパーに任せて、一切コードを書かない場合があります。

例えば Rails の ActiveRecord を利用すると、データアクセス層として記述する内容はほとんどなくなる場合があります。

[![1 (9).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/216010/f19feef3-b83b-ce4b-a4c7-ced4cae8d9f8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F216010%2Ff19feef3-b83b-ce4b-a4c7-ced4cae8d9f8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b831b77d9b5e8c183664073057be3f46)

Rails などの文脈で 3 層アーキテクチャよりも MVC が強調されるのは、上図のように、データアクセス層をほとんど書かないことにより、プレゼンテーション層のアーキテクチャだけ検討すればよくなるからです。

## (余談) ActiveRecord の Model はビジネスロジック層か

※ 初心者向けではない話になります

多くの場合、ActiveRecord の Model はビジネスロジック層の要素として扱うと思います。  
しかし、 [Rails ガイド](https://railsguides.jp/active_record_basics.html) にある以下のコードのように、ActiveRecord の Model にはデータアクセスの感心事が漏洩しています。 [^3]

```ruby
class Product < ApplicationRecord
  self.table_name = "PRODUCT"
end
```

ビジネスロジックとデータアクセスを分離するという思想からすれば、ビジネスロジック層には ActiveRecord を継承することのないピュアなモデルを作成し、ActiveRecord を継承したモデルはデータアクセス層にだけ隠蔽すべきです。

[![1 (10).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/216010/fdf72a0f-b200-4aa3-348d-4e7bf44267dd.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F216010%2Ffdf72a0f-b200-4aa3-348d-4e7bf44267dd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=96b5ec7a356def2ad7d17239149e4d7f)

しかし、その場合はコードの記述量が一気に増えるというトレードオフが発生します。

ここまでビジネスロジック層とデータアクセス層は分離すべきという話で進めてきましたが、どこまで厳密に分離するかは状況次第です。

小規模なアプリケーションを短期で作るのであれば、Rails のようなフレームワークを採用し、ビジネスロジックとデータアクセスの分離まで深く気に留めず、ビジネスロジックはトランザクションスクリプトにした方が高速に開発できます。  
一方、大規模なアプリケーションを長期にわたって改修し続けるのであれば、レイヤーをしっかり分離し、ビジネスロジックはドメインモデルで、適切な言語・フレームワークを選択した方がいいかもしれません。

## レイヤー構成

## ヘキサゴナルアーキテクチャとかクリーンアーキテクチャってやつは??

さて、ここまで 3 層アーキテクチャや、その各層の構成について書いてきましたが、実はレイヤー構成にも選択肢があります。

それが以下のようなアーキテクチャです。

- レイヤードアーキテクチャ
- ヘキサゴナルアーキテクチャ
- オニオンアーキテクチャ
- クリーンアーキテクチャ

この記事ではこれらの違いには触れないので、『 [先行開発！Javaでクリーンアーキテクチャ / Clean architecture with java](https://speakerdeck.com/nrslib/clean-architecture-with-java) 』などを参照ください。

## 要するに

ここまでの話をまとめると、アプリケーション・アーキテクチャとして、以下のような選択があることが分かります。

レイヤー構成... 3 層、ヘキサゴナル、クリーンなどから選ぶ  
プレゼンテーション層... MVC、MVP、MVVM などから選ぶ  
ビジネスロジック層... トランザクションスクリプト、ドメインモデルから選ぶ

なので、「3 層 + MVC + トランザクションスクリプト」といった構成や、「クリーンアーキテクチャ + MVVM + ドメインモデル」といった構成になるわけです。

どの組み合わせを採用すべきかは、開発規模や扱えるフレームワークとの相性などにより異なります。

## おわりに

「MVC」、「3 層アーキテクチャ」、「ドメインモデル」、「クリーンアーキテクチャ」などなど、アプリケーション・アーキテクチャに関する用語はたくさんあります。  
まずは、どれがどの部分の話をしているのかを理解することが重要です。  
どの部分の話をしているのか分かれば、内容の理解も捗ります。

次はこの内容をコードにして公開したいと思います。

## 【2020/12/12 追記】

アドベントカレンダーでこの記事と関連する内容をコードとともに解説しています。  
興味を持ってくださった方は [じゃんけんアドベントカレンダー](https://qiita.com/advent-calendar/2020/janken) を参照ください。

## 【2021/05/23 追記】

この記事の内容と関連するサンプルコードを以下の記事で解説しました。

- [Raspberry Pi で動かすコードをクリーンアーキテクチャ的な考え方で整理する](https://www.kanzennirikaisita.com/posts/raspberrypi-clean-architecture)

## 補足

## SPA などのフロントエンドやモバイルは??

SPA やモバイルの文脈では、MVVM などのプレゼンテーション層のアーキテクチャが話題になることが多いですが、クリーンアーキテクチャなども採用可能です。  
iOS アプリの開発などでは、クリーンアーキテクチャの採用が以前から話題になっているようです。  
もちろん、フロントエンドであってもドメインモデルでビジネスロジックを実装することもできます。

## さらに大きくなると

さらに規模が大きくなると、1 つのアプリケーションでは開発効率などが悪くなるため、いわゆるマイクロサービス化するのが最近のトレンドです。  
その場合であっても、各アプリケーションはここまでで説明したようなアーキテクチャの組み合わせで実装することになります。

また、フロントエンド + BFF + API + DB といった構成は、ドメインモデルを採用した場合のプレゼンテーション層・アプリケーション層・ドメイン層・データアクセス層という構成と類似する部分があると思われます。

## 参考

書籍

- [エリック・エヴァンスのドメイン駆動設計](https://www.amazon.co.jp/dp/4798121967)
- [実践ドメイン駆動設計](https://www.amazon.co.jp/dp/479813161X)
- [．ＮＥＴのエンタープライズアプリケーションアーキテクチャ　第２版](https://www.amazon.co.jp/dp/4822298485)
- [「実践ドメイン駆動設計」から学ぶDDDの実装入門](https://www.amazon.co.jp/dp/4798161500)
- [Clean Architecture](https://www.amazon.co.jp/dp/4048930656)

Web

- [Model\_View\_Controller](https://ja.wikipedia.org/wiki/Model_View_Controller)
- [やはりお前らのMVCは間違っている](https://www.slideshare.net/MugeSo/mvc-14469802)
- [Rails ガイド](https://railsguides.jp/active_record_basics.html)
- [先行開発！Javaでクリーンアーキテクチャ / Clean architecture with java](https://speakerdeck.com/nrslib/clean-architecture-with-java)

[3](https://qiita.com/os1ma/items/#comments)

[703](https://qiita.com/os1ma/items/7a229585ebdd8b7d86c2/likers)

590

[^1]: Model という言葉は、場面によって全然意味が違います。注意してください  
「グーがチョキに勝ち、チョキがパーに勝ち、パーがグーに勝つ」と言った **コアなルール (ビジネスロジック) を View や Controller には書いてはいけません** 。  
このことは『 [やはりお前らのMVCは間違っている](https://www.slideshare.net/MugeSo/mvc-14469802) 』などでも言われています。

[^2]: もちろん、完全に現実を反映することは不可能です

[^3]: さらに、データ中心に設計されたモデルをビジネスロジック層で使う場合、オブジェクト指向のモデルを使うドメインモデル方式とも相性が悪いです