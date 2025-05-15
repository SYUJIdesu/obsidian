---
title: "PHP Laravelフレームワークを用いた期間重複検索の実装ガイド"
source: "https://youta-ms.online/tec/laravel_period_search"
author:
  - "[[Youtaの雑記ブログ]]"
published: 2023-09-28
created: 2025-05-15
description: "今回はLaraveで期間検索時の重複データ取得条件ついての記事です。whereを使用して期間重複データの取得条件や逆に期間外のデータ取得条件に付いて解説しています。初心者の方でも分かりやすいように書いているのでぜひ見ていってください！"
tags:
  - "web"
  - "Laravel"
  - "SQL"
---
- [Home](https://youta-ms.online/)
- PHP Laravelフレームワークを用いた期間重複検索の実装ガイド

- [Web制作](https://youta-ms.online/tec)
![記事のサムネイル](https://youta-ms.online/_next/image?url=%2Fimages%2Flaravel.jpg&w=3840&q=75)

## Laravel とは

![Laravelとはの画像](https://youta-ms.online/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fimage1.111ad85f.png&w=3840&q=75)

Laravelとはの画像

Laravel とは、2011 年にリリースされた比較的新しい VMC モデルのフレームワークです。比較的新しいながら、PHP におけるフレームワークの中では、代表的なフレームワークとなっています。

## 基本的な考え方

基本的には以下の考えで重複期間は取得できます。

`検索開始日付 <= 対象終了日付 AND 検索終了日付 >= 対象開始日付`

![重複チェックの画像](https://youta-ms.online/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fimage2.48aea740.png&w=3840&q=75)

## Laravel での検索期間と重複した期間を持つデータを取得する条件

以下の二つについて解説します。

- 開始日、終了日が決まっている場合
- 終了日が不定の状態が考えられる場合

### 開始日、終了日が決まっている場合

開始日と終了日が定まっている場合には基本に基づき、以下の query で検索期間内のデータを取得できます。

***$start\_date*** が検索条件の開始日時で、 ***$end\_date*** が検索条件の終了日時です。

```php
$duplication_date = $this
->where(function ($q) use ($start_date, $end_date) {
   $q->where('sample_start_date', '<=', $end_date)
   ->where('sample_end_datee', '>=', $start_date)
})
->get();
```

### 終了日が不定の状態が考えられる場合

終了日に不定の場合がある場合は以下の query で取得できます。

$start\_dateが検索条件の開始日時で、$end\_date が検索条件の終了日時です。

```php
if ($end_date) {
  $duplication_count = $this
  ->where(function ($q) use ($start_date, $end_date) {
      $q->where('sample_start_date', '<=', $end_date)
      ->where('sample_end_date', '>=', $start_date)
      ->orWhere(function ($q) use ($end_date) {
          $q->where('sample_end_datee', null)
          ->where('sample_start_date', '<=', $end_date);
      });
  });
} else {
  $duplication_count = $this
  ->where(function ($q) use ($start_date) {
      $q->where('sample_end_datee', null)
          ->orWhere('sample_end_date', '>=', $start_date);
  });
}
```

両方とも日付の条件のみなので、実際に使用する場合は必要な条件を付けくわえてください。

## おまけ 検索期間外のデータを取得する条件

検索期間外のデータを取得する条件は以下になります。

`検索開始日付 > 対象終了日付 OR 検索終了日付 < 対象開始日付`

![対象期間外の画像](https://youta-ms.online/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fimage3.762a4b31.png&w=3840&q=75)

```php
$duplication_date = $this
->where(function ($q) use ($start_date, $end_date) {
   $q->where('sample_start_date', '>', $end_date)
   ->where('sample_end_datee', '<', $start_date)
})
->get();
```

## まとめ

いかがでしたでしょうか！

自分が実装を行う際に条件がどうなるのか迷った箇所だったので、備忘録としての役割も兼ねてまとめてみました！

## 実装やエラーが解決できない場合

プログラミングの実装やエラーでどうしてもわからない場合はメンターに相談するのが一番です。

考えている、見えている範囲が狭くなり、解決から遠くに行って何時間も、何日も経っていることなんてよくある話です。

そういう時は聞ける先輩や、メンターに相談することが大事です。

僕にも相談可能なので気軽に相談してください。

[

ご相談はこちら  
（Twitterのプロフィールへ飛びます）

](https://twitter.com/youta_ms)
- [Web制作](https://youta-ms.online/tec)

Share

![筆者のイメージ画像](https://youta-ms.online/_next/image?url=%2Fimages%2Fme.jpg&w=3840&q=75)

Author

Youta

山口県出身の現役のWebエンジニアです。仕事を通して学んだことや、生活していく中で学んだことを発信していきます。

## OTHER ARTICLES

[![railsでjsではなくrubyでtooltipの単位を設定するときの話のサムネイル画像](https://youta-ms.online/_next/image?url=%2Fimages%2Frails.jpg&w=3840&q=75)](https://youta-ms.online/tec/rails_chartjs_tooltip)

- [Web制作](https://youta-ms.online/tec/rails_chartjs_tooltip)
[- Rails](https://youta-ms.online/tec/rails_chartjs_tooltip)

railsでjsではなくrubyでtooltipの単位を設定するときの話

2024/08/20

[View original](https://youta-ms.online/tec/rails_chartjs_tooltip)

[![Railsにおけるクラスメソッドとインスタンスメソッドの違いについてのサムネイル画像](https://youta-ms.online/_next/image?url=%2Fimages%2Frails.jpg&w=3840&q=75)](https://youta-ms.online/tec/rails_instance_class_method)

- [Web制作](https://youta-ms.online/tec/rails_instance_class_method)
[- Rails](https://youta-ms.online/tec/rails_instance_class_method)

Railsにおけるクラスメソッドとインスタンスメソッドの違いについて

2024/05/30

[View original](https://youta-ms.online/tec/rails_instance_class_method)

[![DockerでRails7 + PostgreSQL + esbuildの環境を構築する方法のサムネイル画像](https://youta-ms.online/_next/image?url=%2Fimages%2Fdocker.jpg&w=3840&q=75)](https://youta-ms.online/tec/docker_rails_v7)

- [Web制作](https://youta-ms.online/tec/docker_rails_v7)
[- Docker
- Rails](https://youta-ms.online/tec/docker_rails_v7)

DockerでRails7 + PostgreSQL + esbuildの環境を構築する方法

2024/05/16

[View original](https://youta-ms.online/tec/docker_rails_v7)

[![Dockerで建てたrails7環境で「undefined method `devise' for 〜」が発生した話のサムネイル画像](https://youta-ms.online/_next/image?url=%2Fimages%2Frails.jpg&w=3840&q=75)](https://youta-ms.online/tec/rails_devise_error_1)

- [Web制作](https://youta-ms.online/tec/rails_devise_error_1)
[- Rails
- Docker](https://youta-ms.online/tec/rails_devise_error_1)

Dockerで建てたrails7環境で「undefined method \`devise' for 〜」が発生した話

2024/05/16

[View original](https://youta-ms.online/tec/rails_devise_error_1)

[![Ruby on RailsのルーティングとRESTfulルートのサムネイル画像](https://youta-ms.online/_next/image?url=%2Fimages%2Frails.jpg&w=3840&q=75)](https://youta-ms.online/tec/rails_route_foundation)

- [Web制作](https://youta-ms.online/tec/rails_route_foundation)
[- Rails](https://youta-ms.online/tec/rails_route_foundation)

Ruby on RailsのルーティングとRESTfulルート

2024/05/09

[View original](https://youta-ms.online/tec/rails_route_foundation)

[![Ruby on Railsにおけるresourceとresourcesの違いのサムネイル画像](https://youta-ms.online/_next/image?url=%2Fimages%2Frails.jpg&w=3840&q=75)](https://youta-ms.online/tec/rails_difference_between_resource_resources)

- [Web制作](https://youta-ms.online/tec/rails_difference_between_resource_resources)
[- Rails](https://youta-ms.online/tec/rails_difference_between_resource_resources)

Ruby on Railsにおけるresourceとresourcesの違い

2024/05/01

[View original](https://youta-ms.online/tec/rails_difference_between_resource_resources)