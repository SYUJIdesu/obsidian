---
title: "DDDの一般的なアーキテクチャをまとめてみた"
source: "https://zenn.dev/ayumukob/articles/ff183004d09ede#%E4%B8%80%E8%88%AC%E7%9A%84%E3%81%AAweb%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AE%E3%82%A2%E3%83%BC%E3%82%AD%E3%83%86%E3%82%AF%E3%83%81%E3%83%A3(3%E5%B1%A4%E3%82%A2%E3%83%BC%E3%82%AD%E3%83%86%E3%82%AF%E3%83%81%E3%83%A3)"
author:
  - "[[Zenn]]"
published: 2022-10-31
created: 2025-05-15
description:
tags:
  - "web"
  - "DDD"
---
182

5[tech](https://zenn.dev/tech-or-idea)

## はじめに

久しぶりの投稿ですが、仕事の中でDDDに関連したアーキテクチャを整理する機会があったので、改めて知識の整理のためにまとめておきたいと思います。

## 一般的なWebアプリケーションのアーキテクチャ(3層アーキテクチャ)

![web3層アーキテクチャ](https://storage.googleapis.com/zenn-user-upload/c7a45ab4f569-20221027.png)  
**矢印の方向に依存** していきます。  
一般にこの依存関係を持ってwebアプリケーションは作られますが、ビジネスロジック層が肥大化する傾向が多いです。  
それぞれの層を言語化すると以下のようになります。

- プレゼンテーション層 = クライアントとの入出力をする層
- ビジネスロジック層 = ビジネスロジックの実現をする層
- データアクセス層 = データベースとの入出力をする層

## 問題点

その中でビジネスロジック層は、そもそもビジネスロジックが複雑になりがちなため、うまく設計ができずに、 **メンテナンス性の悪い、所謂神クラスの氾濫(xxxService、xxxManager)が起こりがち** です。

## DDD本のアーキテクチャ

DDD本では3層アーキテクチャの問題点を緩和するためにレイヤードアーキテクチャを採用しています。  
![Layered Architecture](https://storage.googleapis.com/zenn-user-upload/76399755bb99-20221028.png)  
出展) [https://www.researchgate.net/figure/DDD-Layered-Architecture-2\_fig3\_353246528](https://www.researchgate.net/figure/DDD-Layered-Architecture-2_fig3_353246528)  
それぞれが3層アーキテクチャに照らし合わせると以下のようになる。

- UserInterface = プレゼンテーション層 = クライアントとの入出力をする層
- Application = ビジネスロジック層 = ユースケースの実現をする層
- Domain = ビジネスロジック層 = ルールや制約などのドメイン知識の実現をする層
- Infrastructure = データアクセス層 = データベースとの入出力をする層

こうすることで肥大化していたビジネスロジック層が分離し、 **各層のロジックの凝集度が上がることで、メンテナンス性が向上する** ので神クラスが減ります。

## 問題点

ドメイン層がインフラ層に依存しているということはドメイン層が特定のDBやライブラリに依存しているということになり、DBやライブラリの実装に引きずられてドメイン層の実装が制限されてしまいます。  
**最大の問題点は上記のような密結合状態となってしまう** ことです。

## 依存性逆転の法則を導入

レイヤードアーキテクチャの問題点を解消するためにDDD本が出た後に先人たちが最適化したものが以下の図となります。  
![DIP](https://storage.googleapis.com/zenn-user-upload/5acca2804680-20221028.png)  
出典) [https://codezine.jp/article/detail/9922](https://codezine.jp/article/detail/9922)  
例えばドメイン層にインターフェースを作成し、それをインフラストラクチャ層で実装するとDBやライブラリに関した実装だけがインフラストラクチャ層に実装され、ドメイン層はドメイン層のインターフェースにだけ依存される状態となり、インフラストラクチャ層の処理が隠蔽されます。  
これにより **レイヤードアーキテクチャでの問題点であった依存関係を逆転させることができます。**

## IDDD本のアーキテクチャ(ヘキサゴナルアーキテクチャ)

別名ポートアンドアダプターアーキテクチャ。  
依存性逆転を意識し、2005年に提唱されたアーキテクチャです。  
この後の2つと本質的には同じものになります。  
![ヘキサゴナルアーキテクチャ](https://storage.googleapis.com/zenn-user-upload/54c41a9e0010-20221028.png)  
出典) [https://alistair.cockburn.us/hexagonal-architecture/](https://alistair.cockburn.us/hexagonal-architecture/)  
アプリケーションの外の世界と通信するためには、 **専用のポート** と **アダプター** で通信させるという考え方となります。  
これが意味するものはユーザーインターフェースにビジネスロジックが混入してしまう現象の解決ということで、 **依存性逆転させたレイヤードアーキテクチャと同じ意味を持ちます。**  
つまり、アダプターと呼ばれるものを利用しアプリケーション層のコアロジックが外に漏れることを防いでいます。  
これは、全ての依存が中心の層に向かい、逆に中心から外の層へは依存しないということになります。

## オニオンアーキテクチャ

2008年に提唱されたアーキテクチャ。  
**ヘキサゴナルをもっと直感的にした** ようなイメージです。  
![オニオンアーキテクチャ](https://storage.googleapis.com/zenn-user-upload/41e2574e344c-20221028.png)  
こちらは原典にフラットな図も載っており、より依存性逆転なレイヤードアーキテクチャをイメージできます。  
![オニオンアーキテクチャ(フラット)](https://storage.googleapis.com/zenn-user-upload/1289eef43852-20221028.png)  
出典) [https://jeffreypalermo.com/2008/08/the-onion-architecture-part-3/](https://jeffreypalermo.com/2008/08/the-onion-architecture-part-3/)  
オニオンアーキテクチャの場合、各層は以下のようになります。

- UserInterface = プレゼンテーション層 = クライアントとの入出力をする層
- Infrastructure = インフラストラクチャ層 = データベースとの入出力をする層
- Tests = テストするためのI/O層 = テストのための入出力をする層
- ApplicationService = アプリケーション層 = ユースケースの実現をする層
- ObjectService(DomainServices) = ドメイン層 = ビジネスロジックに関わる振る舞いの処理をするそう
- ObjectModel(DomainModel) = ドメイン層 = ルールや制約などのドメイン知識の実現をする層

フラット表現の場合だといくつか用語が違いますが、Business Logic = アプリケーション層、ObjectModel = ドメイン層(ObjectServices含む)に読み替えれば良いです。  
オニオンアーキテクチャでも全ての依存が中心の層に向かい、逆に中心から外の層へは依存しないということになります。  
DIPを用いたレイヤードアーキテクチャと同様にインフラストラクチャ層の内側の層にInterfaceを定義し、インフラストラクチャ層がそれを実装する形で実現します。

## クリーンアーキテクチャ

2012年に提唱された最新のアーキテクチャ。  
Webアプリケーションに限らず、システム開発するうえでのアーキテクチャになっており、 **オニオンアーキテクチャをさらに細かくした** イメージになります。  
![クリーンアーキテクチャ](https://storage.googleapis.com/zenn-user-upload/15163d215c9e-20221028.png)  
出典) [https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)  
基本的に言ってることは同じで全ての依存が中心の層に向かい、逆に中心から外の層へは依存しないということになります。  
クリーンアーキテクチャの場合、各層は以下のようになります。

- Frameworks & Drivers = フレームワークやDBなど、詳細な技術についてを置くための層
- Interface Adapters = Frameworks & DriversとApplication Business Ruleをやりとりする(相互変換)層
- Application Business Rule = アプリケーション層
- Enterprise Business Rule = ドメイン層

さらにクリーンアーキテクチャは各層がさらに細かく分かれています。

| Frameworks & Drivers | Interface Adapters | Application Business Rule | Enterprise Business Rule |
| --- | --- | --- | --- |
| UI | Presentation | Use Cases | Entities |
| DB | Controller |  |  |
| Web | GateWays |  |  |
| External Interfaces |  |  |  |
| Devices |  |  |  |

## Frameworks & Drivers

各要素はそれぞれの技術ロジックが置かれるものになります。

## Interface Adapters

- Presenter = ユースケースからアウトプットを受け取って、返却に適した形にします。
- Controller = Freamworks & Driversからのデータを抽象化します。
- Gateways = Webなどからデータを受け取り、インプットに変換してユースケースに渡します。

## Application Business Rule

- Use Cases = エンティティを利用しユースケースの実現をします。

## Enterprise Business Rule

- Entities = 大規模プロジェクトレベルのビジネスルールをカプセル化したものになります。わかりづらいですがドメインオブジェクトとみなしてよいです。

最新のアーキテクチャだけあって今までのアーキテクチャをより細かくしており、より具体的なものになっていますが、DDDという文脈でいうとEntitiyや境界付けられたコンテキストの扱いなどがあるため、若干イメージしづらい部分があります。

## 一般的な層の責務

![各層の責務](https://storage.googleapis.com/zenn-user-upload/0da2579f4501-20221031.png)  
参考) [DDDオンライン勉強会 #4「DDDのアーキテクチャ」](https://www.youtube.com/watch?v=y5BX5oxhY1I&t=2433s)  
一般的な各層の責務は図のようになります。  
ドメイン設計時には **ロジックの責務と照らし合わせて、どの層に配置するべきなのか** 共通認識を持つ必要があります。  
また、DDDのアーキテクチャで重要なのは **依存の流れも意識します。**  
**この２点をしっかり守りドメイン設計を行うべきとなります。**

## アーキテクチャの選択

DDDでは決まったルールはないため、これまで挙げたアーキテクチャの中で **チームが一番しっくりくるもの** を選択します。  
ベースのアーキテクチャでわかりずらかったりするのであれば他のアーキテクチャの要素を取り入れてみるなど柔軟に考えてよいです。

## まとめ

改めてまとめると意外にクリーンアーキテクチャが新しいものに関わらず結構わかりづらい印象があるなーと。それ以外のアーキテクチャは個人的にはイメージしやすいと思いました。  
特に依存の向きというのが業務で意識はしてるつもりではあるけれど、徹底出来ていない分があるとは思っているので、しっかりアーキテクチャの目的を意識した上で開発していきたいですね！

ざっくりしたアーキテクチャもよくわからないなーという方がいらっしゃったら参考にしていただければと思います。

182

5