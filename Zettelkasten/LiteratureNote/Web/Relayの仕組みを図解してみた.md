---
title: Relayの仕組みを図解してみた
source: https://qiita.com/cheez921/items/cecd00136f79111e81ef
author:
  - "[[cheez921]]"
published: 2021-12-16
created: 2025-05-22
description: Relayを触り始めた当初、useQueryとuseFragmentって何してるの？何が違うの?ってレベルでよくわからなかったので、初めてRelayを触る人のために詳しく図解してみました。そもそ…
tags:
  - web
  - GraphQL
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から3年以上が経過しています。

Relayを触り始めた当初、 `useQuery` と `useFragment` って何してるの？何が違うの?ってレベルでよくわからなかったので、  
初めてRelayを触る人のために詳しく図解してみました。

## そもそもRelayとは

Relayは、facebook社(今はMetaか...)が開発しているReactのためのGraphQLクライアントです。  
Relayには下記のようなメリットがあります。

- 取得したいデータの構造をコンポーネントに閉じ込めることができる
	- つまり、コンポーネント内で取得するデータが変更されても他のコンポーネントに影響がでないため、依存関係を気にしなくて良い。
	- `useQuery` と `useFragment` の仕組みによるもの → のちほど詳しく解説！
- フェッチ・レンダリングの自動的に最適化される
	- クエリの重複フィールドの除去
	- データに変更があった時に、必要なコンポーネントのみ更新する

▼ 公式サイト

## めっちゃ簡単に仕組みを解説！

ファミレスに家族で行ったとします。  
子供が「食べたい！！」って言ったものを、お母さんが店員さんに注文するような光景をよくみますね。

子供は〇〇が欲しいと伝えるだけ  
直接注文するのはお母さん

そのような時のお母さんがQueryで、子供たちがFragmentです。

[![フラグメント例.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/142322/a8da8e60-deab-9d4e-5ed9-5b2df4b852b2.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F142322%2Fa8da8e60-deab-9d4e-5ed9-5b2df4b852b2.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5243d775282f7897d11727f782c2d4f6)

つまり、 **Fragmentは直接データベースとはやりとりをしません。**  
Queryが各コンポーネントのFragmentを集約してデータベースにお問い合わせします。

## もう少し詳しく

[![フラグメント.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/142322/944fd20c-db8c-7690-c226-0cc8b07c6668.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F142322%2F944fd20c-db8c-7690-c226-0cc8b07c6668.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4db0ea44a00c0e23b7081e2d3bb572bb)

### useFragment

子コンポーネントでは、欲しいデータをGraphQLのフラグメントとして定義します。

useFragmentは、Queryで生成されるフラグメントに対する参照と、そのフラグメント自体を受け取ってデータを返します。

### useQuery

親コンポーネントで、子コンポーネントのフラグメントをまとめたクエリを作り、データベースでお問い合わせします。  
もし、コンポーネントAでも〇〇のデータが欲しい！って書いてあってBでも同じく〇〇のデータが欲しい！って書いてあったとしても、クエリをまとめる時にマージされるため問題ないです。

useQueryの返り値にはフラグメントの参照を持っています。  
これらを子コンポーネントに渡してあげる必要があります。

### なぜこの仕組みが実現できてるのか

Relayでは、schemaファイルからデータの構造を見て各クエリ、フラグメントをコンパイルし設計書のようなものを生成します。  
その設計書がデータの依存関係を解決するため、このような仕組みが実現できます。

---

React ConfでもRelayの話が取り上げられていました！！

[0](https://qiita.com/cheez921/items/#comments)

コメント一覧へ移動

[39](https://qiita.com/cheez921/items/cecd00136f79111e81ef/likers)

いいねしたユーザー一覧へ移動

7