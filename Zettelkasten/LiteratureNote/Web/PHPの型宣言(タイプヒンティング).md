---
title: PHPの型宣言(タイプヒンティング)
source: https://qiita.com/buntafujikawa/items/675dec8f3f357b23da5e
author:
  - "[[buntafujikawa]]"
published: 2016-12-17
created: 2025-05-22
description: 型宣言(タイプヒンティング)とは関数に渡すパラメータ（引数）が、特定の型であることを関数の宣言時に要求できるようになります。型宣言をするには、引数名の前に型名を追加するだけです。$userLi…
tags:
  - web
  - PHP
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

## 型宣言(タイプヒンティング)とは

関数に渡すパラメータ（引数）が、特定の型であることを関数の宣言時に要求できるようになります。  
型宣言をするには、引数名の前に型名を追加するだけです。

```php
$userList = ['user1', 'user2'];
function test(array $list) // タイプヒンティング
{
    echo $list;
}

test($userList); // OK
test('hoge');    // Catchable fatal error: Argument 1 passed to test() must be of the type array, string given
```

この場合はわかり易いですが、$listは配列でなければエラーが発生します。  
arrayだと思ってたのにstringじゃんって怒られます。

## すべての型を指定できるのか

バージョン毎にアップデートされているので、最新の情報はこちらを参考にしてください。  
[https://www.php.net/manual/ja/language.types.declarations.php](https://www.php.net/manual/ja/language.types.declarations.php)

古いバージョンだと、  
PHP5.4まではself,array,callable,クラス名・インターフェイス名のみで  
PHP7からbool,float,int,stringの指定ができます。

## クラス名・インターフェイス名の指定

```php
class Test
{
    private $hoge;

    public function __construct(Hoge $hoge = null) // タイプヒンティング
    {
        $this->hoge = $hoge ?: new Hoge();
    }
}
```

この場合$hogeはHogeクラスのインスタンスなければエラーが発生します。

## デフォルト値をnullにしている場合

```php
function test(array $array = null)
{
    var_dump($array);
}

test(); // NULL
test('hoge'); // PHP Fatal error:
```

引数のデフォルト値にnullを指定している場合には、タイプヒンティングと相違が生まれるが問題ありません。  
この場合には引数の値がnullもしくはarrayであればエラーは発生しません。

## タイプヒンティングのメリット

1.phpdocよりも厳密で引数の型がわかり易い  
メソッドの上にコメントで型が書いてあることも多いですが、引数の隣に明示的に型が書いてあるためわかりやすいです。(個人的には)  
コメントはあくまで補足のため、実際の型と相違があってもエラーにはなりません。

```php
<?php

function hoge(string $string)
{
    var_dump($string);
}

// string型を宣言している引数にarrayを指定するとfatal errorになる
hoge(['hoge']);
// PHP Fatal error:  Uncaught TypeError: Argument 1 passed to hoge() must be of the type string, array given

/**
 * @param $string
 */
function fuga($string)
{
    var_dump($string);
}

// 型宣言はせずにphpdocでstring型を指定したメソッドの引数にarrayを渡す
fuga(['fuga']);
/*
array(1) {
  [0]=>
  string(4) "fuga"
}
 */
```

2.エラー検知  
違う型を引数に渡した場合、関数宣言時にCatchable fatal errorが発生するため、原因の特定が楽なのとcatch可能です。

3.無駄な型判定のコードなどを書かなくて済む  
タイプヒンティングをしているとその関数の宣言時に判定が行われます。

```php
function hoge($fuga)
{
    // array $fuga と書けばこの判定のコードは不要になる
    if (!is_array($fuga)) { 
        return false;
    }
    
    // 処理
}
```

## 連想配列は使いすぎないようにしたい

タイプヒンティングによって型が分かったとしても、連想配列をたくさん使った場合はどうなるでしょうか？  
配列を引数もしくは返り値にした場合、結局全て型が `array` になってしまいます。

```php
function save(array $data): array {
    // ここでもらったデータをもとに何かしらのデータを加工や保存する
    // $data['name']
    // $data['phone']
    // $data['something...']

    return doSomething(); // 何か処理して配列を返す
}
```

PHP を書いていると、このように、この配列には一体何が入ってくるんだろうかと思ったことないでしょうか。

理解するために呼び出し元(呼び出し先)のコードを全部読む必要が出てきたり、実は他のところから使われていて...みたいなことがありますよね。

### 連想配列ではなくクラスを渡すようにする

key が 5 個ある連想配列であれば、それを Class として定義し、他のメソッドに渡すというやり方もあります。

Laravel 向けですが、よくまとまっている記事があったので参考まで。

デメリットとしては作成するファイル数が増えるのですが、ここら辺は GitHub Copilot や ChatGPT を使うことで、かなりの部分が自動化できるようになったように感じます。

プログラムを書く回数 <<<< プログラムを読む回数なので、クラスを1つ作っておくだけで解消できる問題であれば、クラスにしておくのが良いのではないでしょうか。

## 注意点

## 強い型付けと弱い型付け

[マニュアル](http://php.net/manual/ja/functions.arguments.php#functions.arguments.type-declaration) にこのような記載があったので、違う型を渡しても変換されてしまうことがあるそうです。

> デフォルトでは、間違った型を渡された場合でも、可能な限りは来されている型に変換します。 たとえば、string を想定している関数のパラメータに integer が渡された場合は、その値を string 型として受け取ります。

実際に書いてみるとたしかに `1 → "1"` となりました。(弱い型付け)  
厳密な型チェック(強い型付け)をするには、 `declare(strict_types=1);` の記載が必要だそうです。

```php
<?php

//declare(strict_types=1);

function test(string $string)
{
    return $string;
}

var_dump(test("hoge")); // string(4) "hoge"
var_dump(test(1)); // string(1) "1"
```

## 型宣言や declare(strict\_types=1) は使うべきなのか

個人的には全てのプロジェクトで使うようにしています。IDE の補完を使えば楽に欠けますし、PHPStan で declare(strict\_types=1) を必須にすれば良いだけです。

そこまで型をちゃんとしたいなら、PHP 使うなよという意見も見かけることがあると思います。(何回かどこかで見かけたことがある)  
実際の業務で使うことを考えると、長く PHP を使っている会社には PHP 開発者が集まるため、言語を変える意思決定は小さくないはずです。

開発者のレベル感もばらつく可能性があるので、PHP を使った上でどこまできっちり作れるかを考えると、書ける型はちゃんと書いておくということが、業務上でコストパフォーマンスのよい方法に感じます。

[0](https://qiita.com/buntafujikawa/items/#comments)

[153](https://qiita.com/buntafujikawa/items/675dec8f3f357b23da5e/likers)

132