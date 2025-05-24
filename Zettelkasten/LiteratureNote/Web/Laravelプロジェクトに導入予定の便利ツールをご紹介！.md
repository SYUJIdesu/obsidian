---
title: Laravelプロジェクトに導入予定の便利ツールをご紹介！
source: https://qiita.com/hiroki_kawaguchi/items/1fc2ed13daffc06dff41
author:
  - "[[hiroki_kawaguchi]]"
published: 2024-12-18
created: 2025-05-22
description: 背景laravelを用いたプロジェクトに参画しており、お客様が開発生産性に対して関心が高いこともあり開発生産性を上げるべく新しいツールや方法をよく提案しております。その中で現在、導入しようと画策し…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

## 背景

laravelを用いたプロジェクトに参画しており、お客様が開発生産性に対して関心が高いこともあり開発生産性を上げるべく新しいツールや方法をよく提案しております。その中で現在、導入しようと画策しているツールとその課題点・改善作を紹介したいと思います。

## 各ツールの紹介

## PHPStan

[![phpstan.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3935913/957d7bd6-6567-20c7-416c-7102a41ec5d9.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3935913%2F957d7bd6-6567-20c7-416c-7102a41ec5d9.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6d95d9eaa25678000e13cb3f17360978)  
（最初はゾウが虫眼鏡を持ってるんじゃなくて、目の大きいバケモノだと勘違いしました...）

### PHPStanとは？

PHPStan は、Ondřej Mirtesさんが開発しているPHPへの静的解析ツールで、VSCODEの拡張などを使用すれば開発中にコードの問題点などを指摘してくれる便利ツールです。

エラーとして出力してくれるものはレベルによって設定可能で以下のようなものになります。

| レベル | 文章 |
| --- | --- |
| 0 | 未知のクラス   未知の関数   `$this` の未知のメソッド   メソッド・関数の引数の数の不一致   常に未定義の変数 |
| 1 | 潜在的に未定義の変数   `__call` や `__get` のマジックメソッドと未知のプロパティ |
| 2 | 全ての式での未知のメソッドチェック   PHPDocの検証 |
| 3 | メソッドの戻り値の型   プロパティへの型代入 |
| 4 | 常に偽の `instanceof` チェック   到達不可能なコード   不要な `else` 分岐 |
| 5 | メソッドと関数に渡される引数の型 |
| 6 | 不足している型ヒント |
| 7 | ユニオン型の部分的な不正確さ   一部の型にのみ存在するメソッド呼び出し |
| 8 | Nullableな型のメソッドアクセス   `mixed` 型の厳格な制限 |
| 9 | 暗黙の `mixed` 型の検出   型の明示的な指定を強制 |
| 10 | 最も厳格な型チェック   考えられるあらゆる型の不整合を検出 |

### 導入背景

導入対象のプロジェクトは長期間運用されているコードでありリファクタリングや新規開発を行う際に効率的になるツールを導入しようと考えていました。既存のコードに潜在する型の不整合や未定義の変数、不適切なメソッド呼び出しなどの問題を静的に検出し、可視化することが主な目的で、これによってコードの品質と開発速度が向上することが狙いです。

PHPStanの使い勝手はよく、自分の気づいてないコードの欠点を指摘をしてくれます。

### 導入する際の課題

問題は最初からPHPStanが導入されていたプロジェクトではないため、既存コードに対して非常に多いPHPStanのエラーが発生しました。一度に全てのエラーを修正することは難しく、新規に書いたコードも過去のエラーに埋もれてしまうことがあります。

### 改善策

このようなケースはPHPStanは [ベースライン機能](https://phpstan.org/user-guide/baseline) があります。ベースライン機能は既存のコードから発生するエラーを記録し、該当のエラーを無視することができます。これによって新規のコードで発生するエラーのみに注力することが可能です。

ベースラインの設定ファイルの作成方法は以下のコマンドを実行するのみで可能です。

```bash
./vendor/bin/phpstan analyze --generate-baseline
```

## Pint

[![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3935913/42f48260-1ff9-549b-7554-ebbf176e3e97.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F3935913%2F42f48260-1ff9-549b-7554-ebbf176e3e97.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f7062c3d4e111fcee1be678840ee7c82)

### Pintとは

Laravel のコーディングスタイルの問題を検出するために PHP CS Fixer の上に構築されたコード品質ツールです。

### 導入背景

各個人の手癖で書かれたコードはインデントやスコープといったものが既存コードでは様々であり、統一感がなく可読性が悪い状態です。可読性が悪い状態は開発効率を落とし、バグをも生み出す可能性があります。あと、保存時に整形されない状態は非常に使い勝手が悪いですね

### 導入する際の課題

既存コードを全て変更しリリースするとなると全ての機能のテストを行う必要があります。

時間とタイミング的な要因で導入する時期を伺っているのですが、機能に影響のない範囲を考えて導入することは可能です。

### 改善策

Pintの設定ファイルであるpint.json内で設定を行い、部分的に適応したものを随時リリースすることで低い労力で修正します。

具体的には、presetのemptyを使用し、コードの整形を掛けたいルールのみを設定していきます。ルールの参照ですが、 [こちら](https://cs.symfony.com/doc/rules/index.html) のサイトのよく使用しています。

```json
{
    "preset": "empty",
    "cache-file": ".pint.cache",
    "exclude": [
    ],
    "rules": {
        "ordered_imports": {
            "imports_order": [
                "class",
                "function",
                "const"
            ],
            "sort_algorithm": "alpha"
        },
        "no_unused_imports": true
    }
}
```

## husky

### huskyとは

husky は git のフックを利用してスクリプトを実行するツールです。npmを用いて導入することができます。

こちらのhuskyを使用して、gitのcommit時などでPHPStanとPintを実行します。PHPStanなどでエラーが発生している場合は開発者に対して、エラー情報を提示し、警告することが可能です。

```bash
# pintを実行
    [ -z "$STAGED_FILES" ] || composer pint $STAGED_FILES

    # phpstanを実行
    [ -z "$STAGED_FILES" ] || composer stan $STAGED_FILES
```

---

以上が紹介した便利ツールになります。  
最初からLinterなりが入っているプロジェクトを体験しており、入れる苦労を知らなかったので今回の経験は非常に良いものとなりました。  
最初にこれらの指針を決めて導入くれていた方々には頭が下がる思いです...🙇♂️

[0](https://qiita.com/hiroki_kawaguchi/items/#comments)

コメント一覧へ移動

[109](https://qiita.com/hiroki_kawaguchi/items/1fc2ed13daffc06dff41/likers)

いいねしたユーザー一覧へ移動

13