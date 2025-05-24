---
title: Larastan(PHPStan)のレベルを5から6(以上max未満)に上げたときの話
source: https://qiita.com/shimabox/items/df03dde8bd6db4733231
author:
  - "[[shimabox]]"
published: 2022-03-07
created: 2025-05-22
description: はじめにはいさい。今回はLarastan(PHPStan)のレベルを、5から6(以上max未満)へ上げたときに自分がぶち当たった壁みたいなものについて書いてみたいと思います。自身のOSSで対応…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から3年以上が経過しています。

## はじめに

はいさい。  
今回はLarastan(PHPStan)のレベルを、5から6(以上max未満)へ上げたときに自分がぶち当たった壁みたいなものについて書いてみたいと思います。  
自身の [OSS](https://github.com/shimabox/laqu "shimabox/laqu: Laqu is Laravel Db Query Helper.") で対応したときにつまづいた際の話であり、スコープが何より狭いですし望むものが得られるか分かりませんが、何か参考になるものがあれば幸いです。

## Larastan(PHPStan)とは

### PHPStanとは

Larastanの前に、まずはPHPStanとはなんぞやという話ですが、PHPStanはPHPの静的解析ツールです。  
[Getting Started | PHPStan](https://phpstan.org/user-guide/getting-started "Getting Started | PHPStan")

コードを実行せずに

- 構文エラーはないか？
- 関数に渡すパラメータの数が適切か？
- 未定義のものにアクセスしようとしていないか？
- 関数に渡す値が関数のパラメータの型宣言と一致するか？
- PHPDocの内容と関数の戻り値は同じか？

などのエラーを見つけてくれます。

### Larastanとは

[nunomaduro/larastan: ⚗️ Adds code analysis to Laravel improving developer productivity and code quality.](https://github.com/nunomaduro/larastan "nunomaduro/larastan: ⚗️ Adds code analysis to Laravel improving developer productivity and code quality.")

**LarastanはPHPStanをLaravel用に最適化したパッケージ** になります。  
Laravel特有の魔法(マジックメソッドの多用とかFacadeとか)を `PHPStan` に認識させるための設定が含まれています。  
Laravelで書いたアプリにPHPStanを適用させたい場合は、Larastanを使ったほうがいいかなぁ〜というか、使うべきです。はい。  
ラッパーを被せることによってブラックボックス化する割合が増すと思うので、PHPStanでバリバリ頑張るのももちろんアリです。  
今回の話において、 [Psalm](https://psalm.dev/ "Psalm - a static analysis tool for PHP") は知らない子とします。

Larastan(PHPStan)は、端的に言うと **型チェックをしてくれる優れもの** だと認識してくれればいいのかなと思います。  
(あと、PHPDocの内容と関数に矛盾が無いか見てくれるのでドキュメントの更新漏れも防げるのがいいね!とも思っています)

インストール方法などは、ググればたくさん出てくると思うので割愛します。

## レベル5はたぶんいける

それではいざ、我がプロジェクトにもLarastanを入れるぞ!と意気込んで導入すると、(PHPDocも型もある程度書いていれば、)個人的な感覚ですけどレベル5くらいまではすんなりいける気がします。  
実案件でも何回か導入していますが、レベル5までは対応させました。  
本当はmaxまで対応したかったんですけど、受託だったこともあり継続的なメンテを考えるとここらへんが妥当かなと。

もちろん、導入するプロジェクトのボリュームにもよりますが、昨今のIDEであれば自動でPHPDocも出力できる※と思いますし、せっかくPHP7.1以上を使っているはずでしょうから型を書いていないのはゲフンゲフン。  
※ Modelであれば [laravel-ide-helper](https://github.com/barryvdh/laravel-ide-helper "barryvdh/laravel-ide-helper: Laravel IDE Helper") を使うのもアリ？なのかな

というのは置いておいて、 [The Baseline | PHPStan](https://phpstan.org/user-guide/baseline "The Baseline | PHPStan") を使ったりとか、 `paths` で対象を絞りながらとかでコツコツやっていけばどうにかなる気はします。  
※ 自分はそうやってきた！  
※ どうしてものところは、 `excludes_analyse` や `ignoreErrors` もある！

#### レベルとは

Larastan(PHPStan)は `level` を指定することで適用するルールを選択できます。  
levelは `0` から `9` までの数字で指定し、 `0` が最も緩いルール、 `9` が最も厳格なルールとなります。levelでは `max` も指定できますが `9` と同義です。  
`9|max` はけっこうキツイです。  
※ levelの範囲はPHPStanのバージョンによって変わります

[Rule Levels | PHPStan](https://phpstan.org/user-guide/rule-levels "Rule Levels | PHPStan")

| レベル | 内容 |
| --- | --- |
| 0 | 基本的なチェック、未知のクラス、未知の関数、$this上で呼び出された未知のメソッド、それらのメソッドや関数に渡された引数の数が間違っている、常に未定義の変数をチェック |
| 1 | 未定義の変数、\_\_call と \_\_get を持つクラスの未知のマジックメソッドとプロパティがある可能性がある |
| 2 | ($this だけでなく)すべての式で未知のメソッドをチェックし、PHPDocs を検証する |
| 3 | 戻り値の型、プロパティに割り当てられた型の確認 |
| 4 | 基本的なデッドコードチェック - instanceofやその他の型チェックが常に `false` 、到達しないelse文、return後の到達不能コードなど |
| 5 | メソッドや関数に渡される引数の型チェック |
| 6 | タイプヒントの欠落を報告する |
| 7 | 部分的に間違っている論理和型の報告 - 論理和型の一部の型にしか存在しないメソッドを呼び出した場合、レベル7はそのことを報告し始めます(その他の不正確な状況も) |
| 8 | null可能な型に対するメソッド呼び出しとプロパティへのアクセスを報告する |
| 9\|max | 混合型に厳密であること - この型で唯一許される操作は、この型を別の混合型に渡すことである |

※ by Google翻訳

## レベル6にして出てくるエラー

ではでは、レベル5まで対応が済んでレベル6に設定を変えていざ `phpstan analyse` をッターンしてみましょう。  
たぶん、けっこうエラーが出てくるのでは？その中でも多分、

- `Method xxx() has no return type specified.`
- `Method xxx() return type has no value type specified in iterable type array.`

が多く出たのではないかなと思います。  
前者は、おそらくですが戻り値が `void` の場合に、 `@return void` なり、 `xxx(): void {}` を書いていないケースが多く当てはまる気がします。もしかしたらそこまで出ないかもしれませんが自分はtestコードで多く出ました。

そして問題は、 `Method xxx() return type has no value type specified in iterable type array.`です。  
※ 直訳すると、 `メソッド xxx() の戻り値の型が反復可能型配列に指定されていない。` です。なんのこっちゃ。

### 配列の中身よろしく

このエラーが出る原因はかんたんに言うと、 **配列の中身がどうなっているか分からんから定義してな!!** ってことです。  
連想配列を使っているところはほば100%怒られる印象です。

例えば、

```php
<?php declare(strict_types = 1);

class User
{
    /** @var string */
    private $name;

    /** @var int */
    private $age;

    /** @var bool */
    private $isActive;

    /**
     * @param string $name
     * @param int    $age
     * @param bool   $isActive
     */
    public function __construct(
        string $name,
        int $age,
        bool $isActive
    ) {
        $this->name     = $name;
        $this->age      = $age;
        $this->isActive = $isActive;
    }

    /**
     * @return array
     */
    public function state(): array
    {
        return [
            'name'     => $this->name,
            'age'      => $this->age,
            'isActive' => $this->isActive
        ];
    }
}
```

こういうのがあったとして、 `phpstan analyse` をしてみるとlevel5だとエラーは出ないのに、level6だとエラーが発生します。

- level5
	- [https://phpstan.org/r/d9f3c65e-f134-4b40-983a-4fd2647d654f](https://phpstan.org/r/d9f3c65e-f134-4b40-983a-4fd2647d654f)
- level6
	- [https://phpstan.org/r/ee2b8c46-3448-4637-beb9-5a2d1276b87d](https://phpstan.org/r/ee2b8c46-3448-4637-beb9-5a2d1276b87d)

[Playground | PHPStan](https://phpstan.org/ "Playground | PHPStan") 便利だ。

### 対応策

ではどうするのという話ですが、

```php
/**
 * @return array<string, string|int|bool>
 */
public function state(): array
{
    // ....
}
```

[こうする](https://phpstan.org/r/1fd467b8-b39d-4dc7-bd37-2359c1a7f81b "Playground | PHPStan") か、

```php
/**
 * @return array{name: string, age: int, isActive: bool}
 */
public function state(): array
{
    // ....
}
```

[こうする](https://phpstan.org/r/7dca66a0-2d85-4af1-a70e-74603850abe7 "Playground | PHPStan") と怒られなくなります。  
後者(`array{name: string, age: int, isActive: bool}`)のほうが、PHPDoc的に分かりやすいかなと思うのですが、仮に

```php
/**
 * @return array<string, string|int|bool>
 */
public function state(): array
{
    return [
        'name'       => $this->name,
        'age'        => $this->age,
        'isActive'   => $this->isActive,
        'birth'      => Carbon::create(2000, 1, 1),
    ];
}
```

こうなった場合、前者(`array<string, string|int|bool>`)では

```text
Method User::state() should return array<string, bool|int|string> but returns array<string, bool|Carbon\Carbon|int|string>.
```

このように怒ってくれますが、後者ではエラーが発生しません。ぴえん。

ただし、後者では

```php
/**
 * @return array{name: string, age: int, isActive: bool}
 */
public function state(): array
{
    return [
        'name'   => $this->name,
        'age'    => $this->age,
        'active' => $this->isActive,
    ];
}
```

こうなっていたら

```text
Method User::state() should return array{name: string, age: int, isActive: bool} but returns array{name: string, age: int, active: bool}.
```

と、怒ってくれます。  
これはどっちにも一長一短があると思いますので、プロジェクトで決めるといいかなと思います。  
※ どっちも対応してくれるやつってあるのかしら？？

### リストの場合

よくある、バリューにオブジェクトが詰まっているリストの場合ですが、これはかんたんで、

```php
<?php
/**
 * @param  User[] $users
 * @return User[]
 */
public function foo(array $users): array
{
    foreach ($users as $user) {
        var_dump($user->state());
    }

    return $users;
}
```

`Type[]` のように定義してあげると大丈夫です。他にも

- `array<Type>`
- `array<int, Type>`
- `non-empty-array<Type>`
- `non-empty-array<int, Type>`

これらの記法が使えます。

### array shapes記法

このあたりの記法を詳しく見るには、コンソールで実行するとこのエラーが出力されているところに、  
💡 See: [https://phpstan.org/blog/solving-phpstan-no-value-type-specified-in-iterable-type](https://phpstan.org/blog/solving-phpstan-no-value-type-specified-in-iterable-type)  
というのがあると思うので、そこを見ればなんとなく分かるかと思います。

例えば、 `Iterator系` はこういう記法が使えます。

- `Iterator<TKey, TValue>`
- `IteratorAggregate<TKey, TValue>`
- `Traversable<TKey, TValue>`

こういう記法は、 `array shapes記法` っていうみたいですね。知らなかったです。

ちなみに、 [ignoring-this-error](https://phpstan.org/blog/solving-phpstan-no-value-type-specified-in-iterable-type#ignoring-this-error "Solving PHPStan error \"No value type specified in iterable type\" | PHPStan") で逃げることも出来るようですが、

```text
parameters:
    checkMissingIterableValueType: false
```

逃げちゃダメだ。

### array<mixed> をなるべく避けたい

PHPってめちゃくちゃ連想配列を使いがちなので、 `array shapes記法` めっちゃ便利やんって思ったのですが、

- 要素をたくさん持っている
- 値の中にさらに配列を持っている
- 条件によって配列の中身が違う(キーが欠落していたり)
- etc...

みたいなのがあった場合、超絶ダルいです。 `array<mixed>` って書きたくなっちゃいます。  
たぶん、そうなったときはきちんと役割を決めてあげて、クラスにしてあげたほうがいいのかなと個人的には思います。  
もちろん、なんでもかんでもクラスにすればいいってもんじゃないと思うので、そこは適材適所って感じですかね。

でも、せっかくここまでやったのなら、 `array<mixed>` を書いたら負け？のような気もするのでリファクタチャンスだと思って対応するといいと思います。

## maxは？

もしかすると、level6まで対応できたらlevel8まではすんなり上げられるかもしれません。ですが、max(level9)にするとまた更に対応難易度が上がる感覚です。

渡す側の引数の型と受け取る側の引数の型が厳密に見られる感じです。うまく説明できないのですが、よくそこまで解析して突っ込んでくるなぁと感心してしまうレベルです。

自分の例で恐縮なのですが、レベルmaxでanalyzeすると、  
[laqu/Query.php#29](https://github.com/shimabox/laqu/blob/master/src/Analyzer/Query.php#L29 "laqu/Query.php at master · shimabox/laqu") において、第2引数の型は `array<int, int>` で定義されているが、  
[laqu/QueryAnalyzer.php#22](https://github.com/shimabox/laqu/blob/master/src/Analyzer/QueryAnalyzer.php#L22 "laqu/QueryAnalyzer.php at master · shimabox/laqu") で、 `array<int, mixed>` を渡しているでって怒られてしまいます。

```text
Parameter #2 $bindings of class Laqu\Analyzer\Query constructor expects array<int, int>, array<int, mixed> given.
```

対応せねば。

あとは、 [Rule Levels | PHPStan](https://phpstan.org/user-guide/rule-levels "Rule Levels | PHPStan") に `混合型に厳密であること` とあるので、 `mixed` に厳しいんじゃないかなと思います。  
そこまで大きなソースでmaxにチャレンジしたこと無いので正直分かりません。。

## まとめ

- レベル5からレベル6に上げると、 `return type has no value type specified in iterable type array` で怒られがち
	- これは連想配列を扱っている部分で配列の中身を定義していないのが原因
	- array shapes記法で対応する
- array shapes記法よい
	- php、連想配列使いがちだからドキュメント的にもよい
- `array<mixed>` は避けたい
	- こうなると意味ない
	- クラス化を考えたほうがいいかも
- Larastan(PHPStan)無しでは生きられない

## おわりに

Larastan(PHPStan)のレベルを、5から6(以上max未満)に上げたときの壁について書いてみました。  
最初に書いたとおりコンテキストもスコープも当てにならないかもしれませんが、何か少しでも参考になったものがあれば幸いですし、Larastan(PHPStan)を導入してみようかなと思ってくれたらうれしいです。  
あと、間違いがあれば優しく教えて下さい。

Larastanに限らず、静的解析ツール系は入れたほうが開発者は(導入当初は辛いかもしれませんが)楽になれるので積極的に導入していきましょう!!

## 参考にさせていただきました

- [level=0 から始める PHPStan(Larastan) 導入ガイド - Shin x Blog](https://blog.shin1x1.com/entry/getting-stated-with-phpstan "level=0 から始める PHPStan(Larastan) 導入ガイド - Shin x Blog")
- [PHPStanの型チェック機能をまとめる - Qiita](https://qiita.com/nishimura/items/f5fee3f1efb8294335cd "PHPStanの型チェック機能をまとめる - Qiita")

[0](https://qiita.com/shimabox/items/#comments)

[20](https://qiita.com/shimabox/items/df03dde8bd6db4733231/likers)

12

ストックを更新しました