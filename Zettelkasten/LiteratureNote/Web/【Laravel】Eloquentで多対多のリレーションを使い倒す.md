---
title: 【Laravel】Eloquentで多対多のリレーションを使い倒す
source: https://qiita.com/fujita-goq/items/afd4307e90daf95c4f14
author:
  - "[[fujita-goq]]"
published: 2022-11-21
created: 2025-05-22
description: この記事は「GoQSystem Advent Calendar 2022」の2日目の記事です。はじめに多対多のリレーションは「1対1」や「1対多」のリレーション（※リレーションとは）と比べて若干…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

この記事は「 [GoQSystem Advent Calendar 2022](https://qiita.com/advent-calendar/2022/goqsystem) 」の2日目の記事です。

## はじめに

多対多のリレーションは「1対1」や「1対多」のリレーション（ [※リレーションとは](https://wa3.i-3-i.info/word11596.html) ）と比べて若干複雑です。というのも多対多のリレーションには「中間テーブル」という存在があるからです。

[![sample.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1040106/1802dcff-bee1-5212-c058-e993728b96a5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1040106%2F1802dcff-bee1-5212-c058-e993728b96a5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c601be023c078c0b2defc881c5b3505d)

まず「 **多対多** 」とは何か説明します（ ~~前置きが若干長いです~~ ）。  
**いいから本題だ！という方は [こちら](https://qiita.com/fujita-goq/items/#%E5%A4%9A%E5%AF%BE%E5%A4%9A%E3%83%AA%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92laravel%E3%81%A7%E6%89%B1%E3%81%86) からどうぞ！**

## 多対多とは

さて、そもそも多対多ってどういう関係やねんということですが、たとえば上のER図で示した「注文」と「商品」の関係を見てみます。

商品を3つ（りんご、みかん、ぶどう）扱っているものとします。

**商品一覧**  
[![商品ID：1 (2).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1040106/0fc9c77f-3a52-6892-92af-23f4b245a300.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1040106%2F0fc9c77f-3a52-6892-92af-23f4b245a300.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e38b08745dcfa228675847f4d5af0d25)

このような注文が3つ入ってきました。

**注文一覧**  
[![new.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1040106/6d5c676d-2db8-037c-7fc3-6ab9913481a1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1040106%2F6d5c676d-2db8-037c-7fc3-6ab9913481a1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=44f33f24a0e950eba7015c78dc7dbf4b)

これを見ていただいたら分かるように、「注文」は必ず **1個以上の商品を含んでいます** 。  
つまり、注文と商品は「 **1対多** 」の関係にあります。  
[![new.drawio (1).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2884629/4b389097-8eef-de54-4ad0-7220f6777777.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2884629%2F4b389097-8eef-de54-4ad0-7220f6777777.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9c409d5bf1f38fbf8d0e0728a641f2e8)

逆に、「商品」については必ず **0個以上の注文に属しています** 。  
つまり、商品と注文は「 **1対多** 」の関係にあります。

「0個以上」というのはここでの「ぶどう」のように、必ずしも注文に属しているとは限らない商品も存在しているということです。

[![商品ID：1 (6).png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2884629/e88c6745-4302-2ab6-323b-97101916c3c3.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2884629%2Fe88c6745-4302-2ab6-323b-97101916c3c3.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=62e9363ca2ef7b0615105248b860089b)

このように、関連するテーブルどちらからも1対多の関係にある関係性を「 **多対多** 」といいます。

## なぜ中間テーブルが必要なのか

ではなぜ「多対多」の関係では中間テーブルが必要なのでしょうか。  
お互いのテーブルに外部キーを持たせてこんな感じで定義すればよいのでは...？

[![test.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2884629/75ec4475-ee8e-5c3e-e725-b7bccf3f6453.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2884629%2F75ec4475-ee8e-5c3e-e725-b7bccf3f6453.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a09450d15abf00a2359581eadf59b8c1)

しかしこれは基本的には避けるべき以下のアンチパターンになります。

- **主キーが重複してしまう**
- **一つのカラムに複数の値が入ってしまう**

### 主キーが重複してしまう

上記の「注文一覧」の注文1をこのテーブルで表現しようとすると、主キーが被ったレコードを作ることになってしまいます。

**注文1**  
[![new.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1040106/ffb25964-7b4d-3576-5cf5-5c8ac53e29a5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1040106%2Fffb25964-7b4d-3576-5cf5-5c8ac53e29a5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=29afd9fd50e72c40f4a0dfc7472b4408)  
**orders**  
[![new2.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1040106/9c69d12d-fed3-9793-e2d2-4c227c85f3d1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1040106%2F9c69d12d-fed3-9793-e2d2-4c227c85f3d1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4429aec5c533e4cbc49b90fdb623baee)

### 一つのカラムに複数の値が入ってしまう

もしくは一つのカラムに複数の値を入れざるを得ません

**orders**  
[![new2.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1040106/b68dd0a3-f880-e816-4d14-1a419ae15984.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1040106%2Fb68dd0a3-f880-e816-4d14-1a419ae15984.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c2b2a359c46619c23cc3edd1816a17d5)

こうした事態を避けるために中間テーブルを作る必要があるということです。

#### 注文一覧を中間テーブルを用いて表した図

[![new2.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2884629/76e7cea1-62e9-7f04-241b-114f3468acb4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2884629%2F76e7cea1-62e9-7f04-241b-114f3468acb4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e83bf703fdf979c1ed03ab83e416e02a)

## 多対多リレーションをLaravelで扱う

ここからLaravelでリレーションをどう扱うのかについて見ていきます。

### 環境

```bash
$ php --version
> PHP 8.1.11

$ php artisan --version
> Laravel Framework 9.40.1
```

[注文一覧を中間テーブルを用いて表した図](https://qiita.com/fujita-goq/items/#%E6%B3%A8%E6%96%87%E4%B8%80%E8%A6%A7%E3%82%92%E4%B8%AD%E9%96%93%E3%83%86%E3%83%BC%E3%83%96%E3%83%AB%E3%82%92%E7%94%A8%E3%81%84%E3%81%A6%E8%A1%A8%E3%81%97%E3%81%9F%E5%9B%B3) で確認したそれぞれのテーブルのデータはあらかじめ作成されているものとします。

ordersテーブルとproductsテーブルのスキーマは以下の通りです。  
※order\_productsについては以降で解説してます。

orders

```php
public function up()
{
    Schema::create('orders', function (Blueprint $table) {
        $table->id();
        $table->string('orderer_name');
        $table->timestamps();
    });
}
```

products

```php
public function up()
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->timestamps();
    });
}
```

### 基本的な記述

上記で使用した注文と商品を題材に考えます。  
先に説明したように「多対多」は「1対多」と「1対多」の関係なので、Laravelで「1対多」を表す `BelongsToMany` メソッドを使用します。

Order.php

```php
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Order extends Model
{
    public function products(): BelongsToMany
    {
        return $this->belongsToMany(Product::class);
    }
}
```

今回は注文IDと商品ID以外に数量（ `quantity` ）も扱うため、中間テーブルのモデルを作成する必要があります。  
以下のコマンドを使用しマイグレーションファイルとモデル（ `OrderProduct` ）を作成します。

```bash
$ php artisan make:model OrderProduct -m
```

make:modelコマンドの -m オプションでmigrationファイルも同時に作成できます。

`order_products` テーブルのスキーマ定義はこのようにしておきます。

```php
public function up()
{
    Schema::create('order_products', function (Blueprint $table) {
        $table->id();
        $table->foreignIdFor(Order::class)->constrained();
        $table->foreignIdFor(Product::class)->constrained();
        $table->integer('quantity');
    });
}
```

`foreignIdFor()` と `constrained()` については解説を省略しますがこちらの記事がわかりやすいです。  
[Laravel 外部キー制約の設定方法【2つの方法を解説】](https://takuma-it.com/laravel-foreign-key/)

基本的にLaravel（のコマンド）で作られる `migration` ファイルのテーブルはデフォルトでは複数形(`order_products`)です。

しかし `BelongsToMany` で `Laravel` が想定しているテーブルは単数形(`order_product`)なので、ここに一手間加える必要があります。

マイグレーションファイルを編集してテーブルを単数形（ `order_product` ）にすることも可能ですが、今回は `belongsToMany` の第二引数にテーブル名を指定する方法で対処することにします。

Order.php

```php
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Order extends Model
{
    public function products(): BelongsToMany
    {
        return $this->belongsToMany(Product::class, 'order_products');
    }
}
```

これでこのように記述することで関連したモデルを取得することができます。

```php
Order::find(1)->products;
```

ここで取得されるのはモデル（ `App\Models\Product` ）ではなくコレクション（ `Illuminate\Database\Eloquent\Collection` ）です。

逆も同じで `Product` に `orders` メソッドを追加することで `product` 側からも関連したモデルを取得することができます。

Product.php

```php
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Product extends Model
{
    public function orders(): BelongsToMany
    {
        return $this->belongsToMany(Order::class);
    }
}
```

```php
Product::find(1)->orders;
```

### 中間テーブルにアクセスする

中間テーブル(`order_products`)を取得するには `pivot` プロパティにアクセスします。

```php
$order = Order::find(1);

foreach ($order->products as $product) {
    $product->pivot->quantity;
}
```

先ほども触れましたが `$order->products` で取得できるのはコレクションなため、まずモデルにアクセスするためループで回すか `$order->first()` 等でモデルを取得する必要があります。

**ただこのままでは中間テーブルの `quantity` にアクセスできません。**

デフォルトで `pivot` からアクセスできる値はモデルキー（ `order_id` や `product_id` ）のみであるためです。  
モデルキー以外にアクセスしたいカラムがある場合は `withPivot` を定義する必要があります。

Order.php

```php
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Order extends Model
{
    public function products(): BelongsToMany
    {
        return $this->belongsToMany(Product::class, 'order_products')
            ->withPivot('quantity');
    }
}
```

これで中間テーブルのquantityにアクセスできます。

```php
$order = Order::find(1);

foreach ($order->products as $product) {
    dd($product->pivot->quantity); // 1
}
```

#### プロパティの名前を変更する

この `pivot` というプロパティは `as` メソッドで任意の名前に変更できます。

Order.php

```php
class Order extends Model
{
    public function products(): BelongsToMany
    {
        return $this->belongsToMany(Product::class, 'order_products')
            ->as('purchase')
            ->withPivot('quantity');
    }
}
```

```php
$order = Order::find(1);

foreach ($order->products as $product) {
    dd($product->purchase->quantity); // 1
}
```

#### 中間テーブルの値で結果を絞り込む

where〜系のメソッドを用いて、中間テーブルの値で結果を絞り込むことができます。

Order.php

```php
class Order extends Model
{
    public function products(): BelongsToMany
    {
        return $this->belongsToMany(Product::class, 'order_products')
            ->as('purchase')
            ->withPivot('quantity')
            ->wherePivot('quantity', 3);
    }
}
```

```php
$order = Order::find(2);

foreach ($order->products as $product) {
    dd($product->purchase->quantity); // 3
}
```

「 [注文一覧を中間テーブルを用いて表した図](https://qiita.com/fujita-goq/items/#%E6%B3%A8%E6%96%87%E4%B8%80%E8%A6%A7%E3%82%92%E4%B8%AD%E9%96%93%E3%83%86%E3%83%BC%E3%83%96%E3%83%AB%E3%82%92%E7%94%A8%E3%81%84%E3%81%A6%E8%A1%A8%E3%81%97%E3%81%9F%E5%9B%B3) 」によれば現在保持している `order_id` が `2` の中間テーブルでまず取得できるのは `quantity` が `2` の値レコードであるため、 `quantity` が `3` という検索条件が適用されていることが分かります。

#### 中間テーブルの値で結果を並べ替える

`orderByPivot` メソッドを使うことで中間テーブルの値で結果を並べ替えることができます。

Product.php

```php
class Product extends Model
{
    public function orders(): BelongsToMany
    {
        return $this->belongsToMany(Order::class, 'order_products')
            ->as('purchase')
            ->withPivot('quantity')
            ->orderByPivot('quantity', 'desc');
    }
}
```

```php
$product = Product::find(2);

foreach ($product->orders as $order) {
    dd($order->purchase->quantity); // 10
}
```

#### カスタムピボットモデルを使って追加のメソッドを定義できるようにする

現状のままだと中間テーブルのモデルに新規でメソッドを定義しても使用することができません。  
たとえばテーブル名をちょっと変化させたものを取得するメソッドを考えてみます。

OrderProduct.php

```php
class OrderProduct extends Model
{
    public function showTableName()
    {
        return Str::title(Str::replace('_', ' ', $this->table));
    }
}
```

このまま素直にこのメソッドの使用を試みると以下のエラーが発生します。

```text
BadMethodCallException: Call to undefined method Illuminate\Database\Eloquent\Relations\Pivot::showTableName()
```

**中間テーブルに自由にメソッドやプロパティを定義したい場合はカスタムピボットを使用する必要があるためです。**

まず中間テーブルのモデルの継承を `Illuminate\Database\Eloquent\Relations\Pivot` に差し替えます。

OrderProduct.php

```php
use Illuminate\Database\Eloquent\Relations\Pivot;

class OrderProduct extends Pivot // ← ここ
{
    public function showTableName()
    {
        return Str::title(Str::replace('_', ' ', $this->table));
    }
}
```

また、親モデルで `using` メソッドを使用して中間モデルを指定します。

Order.php

```php
class Order extends Model
{
    public function products(): BelongsToMany
    {
        return $this->belongsToMany(Product::class, 'order_products')
            ->using(OrderProduct::class);
    }
}
```

Product.php

```php
class Product extends Model
{

    public function orders(): BelongsToMany
    {
        return $this->belongsToMany(Order::class, 'order_products')
            ->using(OrderProduct::class);
    }
}
```

これではじめに定義した `showTableName` メソッドが使えるようになりました。

```php
$order = Order::find(1);

foreach ($order->products as $product) {
    dd($product->pivot->shoTableName()); // Order Products
}
```

#### 中間テーブルにレコードを挿入する

たとえば注文IDが「3」の注文にぶどう（商品IDが「3」）を3つ追加したい、となった場合 `attach` メソッドを使うことでそれが実現できます。

```php
$order = Order::find(3);

$order->products()->attach(3, ['quantity' => 1]);
```

注文IDが「3」、商品IDが「3」、数量「1」のレコードが挿入されてます。

| id | order\_id | product\_id | quantity |
| --- | --- | --- | --- |
| 6 | 3 | 3 | 1 |

#### syncメソッドでリレーションを同期させる

最後に `sync` メソッドを紹介して終わりにしたいと思います。

「リレーションを同期させる」とはどういうことかというと、 **リレーションの構築と削除を同時に行う** ということです。  
たとえば現状の中間テーブルのデータはこのようになってます。

| id | order\_id | product\_id | quantity |
| --- | --- | --- | --- |
| 1 | 1 | 1 | 1 |
| 2 | 1 | 2 | 1 |
| 3 | 2 | 1 | 2 |
| 4 | 2 | 2 | 3 |
| 5 | 3 | 2 | 10 |
| 6 | 3 | 3 | 1 |

ここで、「 **注文ID1の注文はやっぱりぶどう5個だけにしたい** 」となった場合、このように `sync` メソッドを使うことでその状態を実現することができます。

```php
$order = Order::find(1);

$order->products()->sync([3 => ['quantity' => 5]]);
```

**結果**

| id | order\_id | product\_id | quantity |
| --- | --- | --- | --- |
| 3 | 2 | 1 | 2 |
| 4 | 2 | 2 | 3 |
| 5 | 3 | 2 | 10 |
| 6 | 3 | 3 | 1 |
| 7 | 1 | 3 | 5 |

id `1` 、id `2` のレコードが削除され、新たにid `7` のレコードが挿入されていることが分かります。

つまり、「 `sync` メソッドで指定したid（ここでは `3` ）以外のレコード（ここでは主キーが `1` と `2` のレコード）は削除されて、逆に存在していないidを指定した場合、モデルキー以外のカラムも含めて新しくレコードが追加される」ということですね。

## まとめ

こんな感じでLaravelには多対多のリレーションを自在に扱う術が豊富にあるので便利ですね。

## 最後に

#### GoQSystemでは一緒に働いてくれる仲間を募集中です！

ご興味がある方は以下リンクよりご確認ください。

[0](https://qiita.com/fujita-goq/items/#comments)

[40](https://qiita.com/fujita-goq/items/afd4307e90daf95c4f14/likers)

38