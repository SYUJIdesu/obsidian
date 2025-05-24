---
title: dirtyの由来を調べてみた
source: https://qiita.com/mrskiro/items/56cba87ff8c89f510c95
author:
  - "[[mrskiro]]"
published: 2023-12-11
created: 2025-05-22
description: この記事は ドワンゴ Advent Calendar 2023 12日目の記事です。きっかけドワンゴの教育事業本部でフロントエンドエンジニアをしているむらさきです。みなさんはエンジニアリングを…
tags:
  - web
  - コード
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

この記事は [ドワンゴ Advent Calendar 2023](https://qiita.com/advent-calendar/2023/dwango) 12日目の記事です。

## きっかけ

ドワンゴの教育事業本部でフロントエンドエンジニアをしているむらさきです。  
みなさんはエンジニアリングをしていてdirtyという単語を目にしたことはありませんか？

例えば僕は普段フロントエンドの開発をしていますが、FormikやReact Hook Form  
といったフォームを扱うライブラリでdirtyという単語を目にすることがあります。

Gitのdocumentではdirty stateという言葉やステータスとしてdirtyが使われていたり、

他にはDBのトランザクションにおいてある処理中に他のコミットされていないデータを読み取ってしまうことをdirty readと言ったりするそうです。DBやメモリ周りで使われている方がよく見るかもしれません。

では、dirtyという単語の意味はなんでしょうか。Googleで調べてみた結果は以下です。

> 汚い，汚れた，不潔な, くすんだ

上述したフォームライブラリでのdirtyは、「変更されたどうか？」というフラグとしての機能を持ちます。

「dirtyは言い過ぎじゃない？」

そう思った僕はこの単語が使われる由来を探す旅に出ました。

## whatwgでは

formについての記載を眺めているとdirtyが含まれている文を発見しました。

> input and textarea elements have a dirty value flag. This is used to track the interaction between the value and default value. If it is false, value mirrors the default value. If it is true, the default value is ignored.

「input要素とtextarea要素はdirty valueというフラグを持ち、valueがデフォルト値ならfalse、そうじゃなければtrueである。」といったことが書いてありました。ライブラリの挙動そのままですね。

言及はあったものの、dirtyという言葉が当たり前のように使われていて、起源や由来まではわかりません。

## Stack Exchangeにて

自分と同じ疑問を質問している人がいました。

とても参考になる回答があり、一部要約、翻訳したものを以下にまとめさせていただきます。

- 1973年の特許にて、 [dirtyを述べている資料](https://patents.google.com/patent/US3800294A/en) がある
- 文献や特許の情報によればdirtyとは、dirty copyという言葉として元々紙の世界から来たのだろう
- dirty copyとは、使い古されてはいるもののまだ使用できる (つまり媒体が使い古されている)コピーのこと
- 技術的な意味において、汚い（使い古されている）というのは信頼性の言葉として有用な抽象である

## おわりに

由来を調べた結果、かなり昔から使われ普及してきた言葉であることがわかりました。

当初感じた「言い過ぎでは？」という疑問はまだ少しだけありますが、背景を知った結果から技術的な言葉として使われることには納得できました。

こういった、普段目にする言葉の由来を調べるのは楽しいですね。最近だと、index.(js|ts)でexportするモジュールを管理するbarrel patternという言葉のbarrelという単語の由来も気になっているので、どこかで調べてみようと思います（知っている方がいたらぜひ教えてください）。

[0](https://qiita.com/mrskiro/items/#comments)

コメント一覧へ移動

[4](https://qiita.com/mrskiro/items/56cba87ff8c89f510c95/likers)

いいねしたユーザー一覧へ移動

1

ストックを更新しました