---
title: データベースフレームワークMedooの書き方　備忘録
source: https://qiita.com/iiizoo/items/4a5dfce4c6f0e47d7c22
author:
  - "[[iiizoo]]"
published: 2020-08-29
created: 2025-05-22
description: たまたまMedooを使う機会があったので、備忘録として書き方を残しておきます。MedooとはPHPのデータベースフレームワークです。（公式サイトはこちら）構造私が使った内容がこちらです。配…
tags:
  - web
  - PHP
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から3年以上が経過しています。

たまたまMedooを使う機会があったので、備忘録として書き方を残しておきます。

## Medooとは

PHPのデータベースフレームワークです。 （公式サイトは [こちら](https://medoo.in/) ）

## 構造

私が使った内容がこちらです。配置する順番は以下になります。

sample.php

```php
select(
  "Account",
   [
     "[><]Shop" => ["Shop" => "id"],
     "[><]Product" => ["Product" => "id"]
   ],
   [
     "Account.Id",
     "Account.Name",
     "Account.Tel",
     "Shop.Id(Shop_id)"
     "Shop.Name(Shop_Name)",
     "Product.Name(Product_Name)"
   ],
   [
      "AND" =>
        [
          "Account.Id" => 1,
          "OR" => [
             "Account.Name[~]" => '田中',
             "Account.Name[~]" => '鈴木'
          ]
        ],
     "GROUP" =>
       [
         "Account.Id"
       ],
     "ORDER" =>
      [
        "Account.CreatedDate" => "DESC"
      ]
   ]
);
```

## 書き方

### １） テーブルをJOINする

JOINしたいテーブルの文頭にJOIN方法に合わせて以下の内容を付けます。

sample.php

```php
// [>] == LEFT JOIN
// [<] == RIGH JOIN
// [<>] == FULL JOIN
// [><] == INNER JOIN

select(
  "Account",    
  "[><]Shop" => ["Shop" => "id"]

\`\`\`

<h3>２） 別名をつける</h3>
SQLで言うところの「AS句」です。
以下の方法でShopテーブルのIdカラムに対して「Shop_id」と言う別名をつけることができます。

\`\`\`sample.php

[
  "Account.Id",
  "Shop.Id(Shop_id)"
]
\`\`\`

<h3>３） あいまい検索</h3>

検索対象のカラム名の後ろに[~]を付けます。

\`\`\`sample.php

 "AND" =>
   [
     "Account.Name[~]" => '田中',
   ],
\`\`\`

<h3>４） 重複を除く</h3>

SQLで言うところの「distinct」に相当する書き方は現時点ではありません。
色々とやり方はあるかもしれませんが、私はGROUPを使う方法で代替しました。（SQLのGROUP BYと同じ）

\`\`\`sample.php

"GROUP" =>
  [
    "Account.Id"
  ],
\`\`\`

<h2>まとめ</h2>
調べてみてもあまり多くの情報が出てきません。
<a href="https://medoo.in/doc">公式ドキュメント</a>を確認するのが一番良さそうです。

そこまで複雑でないSQLであれば、Medooを使ってみても良いかと思います。
```

[0](https://qiita.com/iiizoo/items/#comments)

コメント一覧へ移動

[2](https://qiita.com/iiizoo/items/4a5dfce4c6f0e47d7c22/likers)

いいねしたユーザー一覧へ移動

2