---
title: TableDataGateway & RowDataGateway
source: https://qiita.com/tanakahisateru/items/dd6797f21c02797006da
author:
  - "[[tanakahisateru]]"
published: 2022-12-04
created: 2025-05-22
description: https://bliki-ja.github.io/pofeaa/TableDataGateway/https://bliki-ja.github.io/pofeaa/RowDataGatew…
tags:
  - web
  - アーキテクチャ
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

[![04.TableDataGateway & RowDataGateway.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/11525/eeac3eb6-fa11-473d-e608-fcbb987d18a0.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F11525%2Feeac3eb6-fa11-473d-e608-fcbb987d18a0.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=81afe9c1e5b05eb56b52a60228aaab66)

ここからは、ビジネスロジックをどのように書くかとは別の視点、データソースのアーキテクチャに関するパターンを扱います。

ビジネスロジックの記述、とくに Domain Model においては、データ(あるいはオブジェクト)の扱いと取得/同期方法の関心を、できるだけ分離するのが健全です。ほとんどのデータベース操作は、単純で冗長な SQL の組み立てになってきます。データが出入りするインターフェースを、ビジネスロジックから見たゲートウェイとするのです。

Table Data Gateway は、特定のテーブルとの単純な入出力を担います。責務範囲がひとつのテーブルに閉じるため、Table Module との依存関係がシンプルになります。データの操作は `find(id)` や `insert(data)` といったメソッドで担います。

![](https://qiita.com/tanakahisateru/items/)

データの形式に、固有のデータ型を設けるか、汎用的なレコードセット型のままにするかについては、どちらでもかまいません。データベースの具体的操作を隠蔽することにのみ着目してください。

Table Data Gateway だけで十分な場合、このオブジェクトは `update(id, data)` と `delete(id)` を持つこともあります。が、そうした、対応する行が明らかな更新や削除の操作は、Row Data Gateway に担わせることもできます。

Row Data Gateway は対応する行の主キーを知っており、ビジネスロジックで扱うデータを参照します (データ側に主キー値を持つこともありますが、わかりやすさのため、図ではゲートウェイに id 属性を持たせています)。改めてメソッド引数を取ることなく、この既知の知識を使って操作用の SQL を構築します。対応する行以外への操作を行わないものとすることで、責務範囲が明確になり、扱いやすくなります。

![](https://qiita.com/tanakahisateru/items/)

Table Data Gateway のインスタンス数は、テーブルの数と一致します。Row Data Gateway のインスタンス数は、取得された行の数と一致します。Row Data Gateway の生成の知識を隠蔽するために、Table Data Gateway の find メソッドを Row Data Gateway のファクトリとすることも可能です。

[0](https://qiita.com/tanakahisateru/items/#comments)

コメント一覧へ移動

[17](https://qiita.com/tanakahisateru/items/dd6797f21c02797006da/likers)

いいねしたユーザー一覧へ移動

7

ストックを更新しました