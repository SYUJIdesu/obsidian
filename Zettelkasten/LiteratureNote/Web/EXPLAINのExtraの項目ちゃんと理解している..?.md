---
title: "EXPLAINのExtraの項目ちゃんと理解している..?"
source: "https://zenn.dev/miya_tech/articles/c1b9ca01e90a7b"
author:
  - "[[Zenn]]"
published: 2023-02-09
created: 2025-05-15
description:
tags:
  - "web"
  - "スロークエリ"
---
17[tech](https://zenn.dev/tech-or-idea)

## Extraとは

Extraはクエリを実行するために、  
オプティマイザがどのような戦略を選択したかを示すフィールドです。

ExtraはEXPLAINした時に一番最後に表示されるし、色々文字で書いててよくわからないしで、  
心なしか軽視してしまいがちな気がします...結局は付帯情報にしか過ぎないので。

しかし、Extraを見ればテーブルに対して何を行っているのかが分かるので、  
EXPLAINの出力結果全体を見通す際に役に立つはずです！

## 代表的なExtraで表示される情報

いい感じにまとめられているサイトがあまりなかったので、こちらでまとめてみます。

## 1\. Using Where

WHERE句に検索条件が指定されており、  
且つインデックスだけではWHERE句の条件を全て適用できないことを示す。

## 2\. Using index

クエリがインデックスだけを用いて解決できることを示す  
secondary Indexes を使用している場合などに表示される。  
Using index が Extra に表示されるようにインデックスを貼るのが一つの目標。

## 3\. Using filesort

orderBy句におけるソート処理にインデックスが適用できず、  
filesort(クイックソート) でソートを行っていることを示す。  
処理が遅くなる可能性有。

※filesortってなんやねんって方は [こちら](http://nippondanji.blogspot.com/2009/03/using-filesort.html)

## 4\. Using index condition

部分的にIndexが適用できているを示す。  
また、ICP(インデックスコンディションプッシュダウン)が使用されている時に表示される。

※ICPってなんやねんって方は [こちら](https://dev.mysql.com/doc/refman/5.6/ja/index-condition-pushdown-optimization.html)

## 5\. Using Temporary

JOINの結果をソートしたり、DISTINCTによる重複の排除を行う場合など、  
クエリの実行にテンポラリテーブルが必要なことを示す。

テンポラリテーブルにデータを展開した後、  
クイックソートを実行してデータのソートを行い結果を返すため、処理が遅くなる可能性有。

## 6\. Using index for group-by

MIN関数やMAX関数がgroupBy句と併用される時に、  
クエリがインデックスだけを用いて解決できることを示す。

## まとめ

Extraに何か表示されている時はしっかり読んでみましょう。  
そうすればソート/テンポラリテーブル/インデックス部分適用に関する情報が得られるはずです。

Extraの中でもUsing filesortと、Using temporaryが出た時は注意。

## 参照

17

### Discussion

![](https://static.zenn.studio/images/drawing/discussion.png)

ログインするとコメントできます