---
title: Laravelのwithとjoinの違いについて
source: https://qiita.com/ryocha12/items/8bf538b89739b903e437
author:
  - "[[ryocha12]]"
published: 2023-04-26
created: 2025-05-22
description: N+1問題についてwithやjoinを使う背景にはこのN+1問題があるので、まずはN+1問題について簡単に説明します。理解しているよという人は読み飛ばしてください！！Laravelでこのような…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

## N+1問題について

`with` や `join` を使う背景にはこのN+1問題があるので、まずはN+1問題について簡単に説明します。  
理解しているよという人は読み飛ばしてください！！

Laravelでこのようなリレーションが設定されていたとします。

Customer.php

```php
/**
 * purchasesテーブルへのリレーション（1対多）
 *
 * @return HasMany
 */
public function purchases(): HasMany
{
    return $this->hasMany(Purchase::class);
}
```

以下のようにしてデータを取得することができます。

```php
// customersテーブルから全件取得
$customers = Customer::query()->get();

foreach ($customers as $customer) {
    // purchasesテーブルのデータへアクセスできる
    $customer->purchases;
}
```

上記コードでN+1問題が発生しています。  
発行されたクエリを確認してみます。

[![スクリーンショット 2023-04-13 23.31.31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/934895/913e051b-9725-2d8a-805e-a2810724faf5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F934895%2F913e051b-9725-2d8a-805e-a2810724faf5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6309bb8a85a48ce03b1d50342ed7ffb1)

ご覧の通り一番上のクエリでcustomersテーブルから全件取得しているクエリが1件あります。  
そしてその下に続いているクエリはcustomersテーブルから取れた件数分クエリが発行されています。foreachで回されるたびにクエリを発行しています。  
つまり最初のクエリで取れた件数がN件だった場合、最初の全件取得のクエリの1回 + N回のクエリが発行されるということです。  
N+1より1+Nの方が分かりやすいかもしれません。

仮に数万人のcustomerがいたとしたら数万件のクエリが発行されることになりDBへの負荷がとても高い状態になりますし、スピードもかなり遅くなってしまいます。

これを解消するためにLaravelには `with` と `join` があります。

## withを使う場合

まずは `with` を使って解消してみます。

```php
// customersテーブルから全件取得
$customers = Customer::query()
    ->with('purchases')
    ->get();

foreach ($customers as $customer) {
    // purchasesテーブルのデータへアクセスできる
    $customer->purchases;
}
```

この状態でクエリを確認してみましょう！  
[![スクリーンショット 2023-04-13 23.48.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/934895/5e9f9217-a6ae-eeeb-cd80-3bf5ee325a4c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F934895%2F5e9f9217-a6ae-eeeb-cd80-3bf5ee325a4c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=605f1271cc5a8f90970dd7cea47b0433)

なんとクエリの件数が2件になりました。  
見て分かるようにwhere inを使って複数指定して一括で取得しています。  
このようなクエリにすれば、いくらcustomersテーブルの件数が増えてもクエリの件数は変わることはありません。

## joinを使う場合

`join` を使う場合も見てみましょう。

```php
// customersテーブルから全件取得
$customers = Customer::query()
    ->join('purchases', 'customer.id', '=', 'purchases.customer_id')
    ->get();
```

この時のクエリがこちらです。  
[![スクリーンショット 2023-04-14 0.16.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/934895/d08c8505-4615-abea-e35c-3d83fa020fa4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F934895%2Fd08c8505-4615-abea-e35c-3d83fa020fa4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=739fd6bffc0b56002698f643043d963e)

こちらはクエリ1件で取得ができています。

どちらを使ってもN+1問題は解消することができます。  
ではどちらを使う方がいいのでしょうか？

2つの詳しい違いについて説明します！

## withとjoinの違い

まずそれぞれのやり方で取得したデータを見ていきます。

### withの場合

```php
$customers = Customer::query()
    ->with('purchases')
    ->get();

$customers->count();
$customers[0];
```

こちらの取得結果が以下です。  
[![スクリーンショット 2023-04-15 16.03.09.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/934895/6b614976-12f6-b175-780d-512df0805e14.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F934895%2F6b614976-12f6-b175-780d-512df0805e14.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=62ba73a20d7f2d7649f25faf6f802966)  
customersの件数が1001件です。  
そして、 `$customers[0]` の型はCustomerです。なのでattributesを見るとcustomersテーブルのカラムの情報が取得できています。 `with` で取得した `purchases` はrelationsのところで取得できているのが分かると思います。

これが `with` で取得した場合のデータの情報です。

### joinの場合

```php
$customers = Customer::query()
    ->join('purchases', 'customer.id', '=', 'purchases.customer_id')
    ->get();

$customers->count();
$customers[0];
```

取得結果はこちらです。  
[![スクリーンショット 2023-04-15 16.35.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/934895/4167f593-b19c-d4fb-be85-639dad414076.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F934895%2F4167f593-b19c-d4fb-be85-639dad414076.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=62346e9178416d7df49c9b4ddd3e38d3)

joinの場合は、件数が30000件になっています。これはcustomersの件数ではなく紐付くpurchasesのデータの件数になっています。  
さらにattributesの中の最後の2行を見るとcustomersテーブルには存在しないカラムが追加されています。これはpurchasesテーブルのカラムです。  
型はCustomerなのにpurchasesのデータも取れてしまっています。これはアクティブレコードの利点を失ってしまいます。

型はCustomerですが実体はCustomerでもPurchaseでもないものになっています。  
ORMはテーブルとモデルクラスが1対1の関係で紐づいているのが前提ですがその関係が崩れてしまって不可解な挙動になり、バグが起こる原因になってしまいます。

なので結論、 **withを使うようにしましょう！**

もしjoinを使う場合は

```php
// Eloquentビルダをクエリビルダに変換
$row = Customer::query()
    ->toBase()
    ->join('purchases', 'customer.id', '=', 'purchases.customer_id')
    ->get();

// もしくは

// モデルではなくテーブルをクエリする
$row = DB::table('purchases')
    ->join('purchases', 'customer.id', '=', 'purchases.customer_id')
    ->get();
```

注意点としては結果セットを変形するクエリをEloquentから使うのを避けたり、モデルなのかレコードセットなのか変数名などで明確に区別してあげる必要があります。

また、Eloquentとクエリビルダを混ぜて使うのはバグが紛れ込む原因になるので基本的にはどちらかに統一した方がいいと思います。  
個人的にはLaravelを使ってるならEloquentを使った方がいいと思ってます。  
Eloquentを使ってこそLaravelを使う意味があるんじゃないのかなと思っているからです。

Laravelの開発者のTaylor Otwellさんも開発時に一番大変だったのがEloquentと言っていたそうなのでちゃんと使いこなしてあげたいです。。

以上です！  
最後まで読んでいただきありがとうございました！！

### 参考記事

[0](https://qiita.com/ryocha12/items/#comments)

コメント一覧へ移動

[56](https://qiita.com/ryocha12/items/8bf538b89739b903e437/likers)

いいねしたユーザー一覧へ移動

27

ストックを更新しました