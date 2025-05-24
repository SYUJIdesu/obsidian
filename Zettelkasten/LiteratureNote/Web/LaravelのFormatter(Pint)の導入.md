---
title: LaravelのFormatter(Pint)の導入
source: https://qiita.com/aosan/items/333d048f412bc293dc53
author:
  - "[[aosan]]"
published: 2023-07-20
created: 2025-05-22
description: 1. はじめにPHPのFormatterで有名なPHP-CS-Fixerがありますが、LaravelではPintというFormatterがv9.3から標準搭載されました。https://gith…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

## 1\. はじめに

PHPのFormatterで有名な `PHP-CS-Fixer` がありますが、Laravelでは `Pint` というFormatterがv9.3から標準搭載されました。

本記事では `Pint` の使い方とプリセットの解説をしたいと思います。

## 2\. 対象読者

- Laravel開発者
- コードの品質を向上させたいと考えている方
- チーム開発で一貫性を保つための方法を探している方

## 3\. 目次

- [1\. はじめに](https://qiita.com/aosan/items/#1-%E3%81%AF%E3%81%98%E3%82%81%E3%81%AB)
- [2\. 対象読者](https://qiita.com/aosan/items/#2-%E5%AF%BE%E8%B1%A1%E8%AA%AD%E8%80%85)
- [3\. 目次](https://qiita.com/aosan/items/#3-%E7%9B%AE%E6%AC%A1)
- [4\. インストール方法](https://qiita.com/aosan/items/#4-%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E6%96%B9%E6%B3%95)
- [5\. プリセット `laravel`](https://qiita.com/aosan/items/#5-%E3%83%97%E3%83%AA%E3%82%BB%E3%83%83%E3%83%88laravel)
	- [5-1. **フォーマットとインデントに関する設定**](https://qiita.com/aosan/items/#5-1-%E3%83%95%E3%82%A9%E3%83%BC%E3%83%9E%E3%83%83%E3%83%88%E3%81%A8%E3%82%A4%E3%83%B3%E3%83%87%E3%83%B3%E3%83%88%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B%E8%A8%AD%E5%AE%9A)
	- [5-2. **クラス、関数、制御構造に関する設定**](https://qiita.com/aosan/items/#5-2-%E3%82%AF%E3%83%A9%E3%82%B9%E9%96%A2%E6%95%B0%E5%88%B6%E5%BE%A1%E6%A7%8B%E9%80%A0%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B%E8%A8%AD%E5%AE%9A)
	- [5-3. **構文と命名に関する設定**](https://qiita.com/aosan/items/#5-3-%E6%A7%8B%E6%96%87%E3%81%A8%E5%91%BD%E5%90%8D%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B%E8%A8%AD%E5%AE%9A)
	- [5-4. **コメントとドキュメンテーションに関する設定**](https://qiita.com/aosan/items/#5-4-%E3%82%B3%E3%83%A1%E3%83%B3%E3%83%88%E3%81%A8%E3%83%89%E3%82%AD%E3%83%A5%E3%83%A1%E3%83%B3%E3%83%86%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B%E8%A8%AD%E5%AE%9A)
	- [5-5. **Laravelに関する設定**](https://qiita.com/aosan/items/#5-5-laravel%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B%E8%A8%AD%E5%AE%9A)
- [6\. Pint実行](https://qiita.com/aosan/items/#6-pint%E5%AE%9F%E8%A1%8C)
- [7\. さいごに](https://qiita.com/aosan/items/#7-%E3%81%95%E3%81%84%E3%81%94%E3%81%AB)
- [8\. 参考サイト](https://qiita.com/aosan/items/#8-%E5%8F%82%E8%80%83%E3%82%B5%E3%82%A4%E3%83%88)

## 4\. インストール方法

Laravel v9.3以降の場合は `Pint` は標準搭載されていますが、それより以前のバージョンではPintをインストールする必要があります。  
インストールは以下のComposerコマンドを実行します。

```bash
$ composer require --dev laravel/pint
```

次に、Pintの設定ファイル `pint.json` をプロジェクトのルートディレクトリに作成します。  
Pintでは現在下記の３つのプリセットが用意されています。

- laravel
- psr12
- symfony

例えば、 `laravel` のプリセットを利用する場合、 `pint.json` は以下のようにします。

```json
{
    "preset": "laravel"
}
```

## 5\. プリセットlaravel

プリセット `laravel` のルールは [こちらのリポジトリ](https://github.com/laravel/pint/blob/main/resources/presets/laravel.php) に記載があります。  
設定されているルールが何を表しているのかを簡単にまとめました。

### 5-1. フォーマットとインデントに関する設定

| 設定キー | 説明 |
| --- | --- |
| 'array\_indentation' => true | 配列の各行をインデントします |
| 'binary\_operator\_spaces' => \['default' => 'single\_space'\] | 二項演算子の前後にスペースを1つ挿入します |
| 'blank\_line\_after\_namespace' => true | 名前空間宣言の後に空行を挿入します |
| 'blank\_line\_after\_opening\_tag' => true | 開始タグの後に空行を挿入します |
| 'blank\_line\_before\_statement' => \['statements' => \['continue','return'\]\] | 'continue'と'return'の前に空行を挿入します |
| 'blank\_lines\_before\_namespace' => true | 名前空間宣言の前に空行を挿入します |
| 'cast\_spaces' => true | 型キャストの前後にスペースを挿入します |
| 'concat\_space' => \['spacing' => 'none'\] | 文字列結合演算子の前後にスペースを挿入しません |
| 'declare\_equal\_normalize' => true | declareステートメントの等号前後のスペースを正規化します |
| 'declare\_parentheses' => true | declareステートメントの丸括弧の前後のスペースを正規化します |
| 'indentation\_type' => true | インデントのタイプを統一します（タブまたはスペース） |
| 'linebreak\_after\_opening\_tag' => true | 開始タグの後で改行します |
| 'line\_ending' => true | ファイルの終わりの改行を統一します |
| 'method\_argument\_space' => \['on\_multiline' => 'ignore'\] | メソッドの引数間のスペースを無視します（マルチライン） |
| 'method\_chaining\_indentation' => true | メソッドチェーンのインデントを統一します |
| 'multiline\_whitespace\_before\_semicolons' => \['strategy' => 'no\_multi\_line'\] | セミコロンの前のマルチライン空白を禁止します |
| 'no\_extra\_blank\_lines' => \['tokens' => \['extra','throw','use'\]\] | 余分な空行、'throw'または'use'の後の空行を削除します |
| 'no\_leading\_import\_slash' => true | インポートの先頭のスラッシュを削除します |
| 'no\_leading\_namespace\_whitespace' => true | 名前空間の前にある空白を削除します |
| 'no\_multiline\_whitespace\_around\_double\_arrow' => true | マルチラインのダブルアローの周りの空白を削除します |
| 'no\_singleline\_whitespace\_before\_semicolons' => true | セミコロンの前の単一行空白を削除します |
| 'no\_trailing\_comma\_in\_singleline' => true | 単一行配列の最後の要素の後のカンマを削除します |
| 'no\_trailing\_whitespace' => true | 行の終わりの空白を削除します |
| 'no\_trailing\_whitespace\_in\_comment' => true | コメントの終わりの空白を削除します |
| 'no\_whitespace\_before\_comma\_in\_array' => true | 配列内のカンマの前の空白を削除します |
| 'no\_whitespace\_in\_blank\_line' => true | 空行内の空白を削除します |
| 'space\_after\_semicolon' => true | セミコロンの後にスペースを挿入します |
| 'trailing\_comma\_in\_multiline' => \['elements' => \['arrays'\]\] | マルチライン配列の最後の要素の後にカンマを挿入します |
| 'trim\_array\_spaces' => true | 配列内の空白を削除します |
| 'whitespace\_after\_comma\_in\_array' => true | 配列のカンマの後に空白を挿入します |

### 5-2. クラス、関数、制御構造に関する設定

| 設定キー | 説明 |
| --- | --- |
| 'class\_attributes\_separation' => \['elements' => \['const' => 'one', 'method' => 'one', 'property' => 'one', 'trait\_import' => 'none'\]\] | クラス属性（const、method、property、trait\_import）間の区切り行数を設定します |
| 'class\_definition' => \['multi\_line\_extends\_each\_single\_line' => true, 'single\_item\_single\_line' => true, 'single\_line' => true\] | クラス定義の形式を制御します |
| 'control\_structure\_braces' => true | 制御構造の中括弧の位置を調整します |
| 'control\_structure\_continuation\_position' => \['position' => 'same\_line'\] | 制御構造の続行位置を同じ行にします |
| 'elseif' => true | elseとifを組み合わせたelseifを使用します |
| 'function\_declaration' => true | 関数宣言のスペースや改行を調整します |
| 'include' => true | includeとinclude\_onceを正しい形にします |
| 'no\_unneeded\_control\_parentheses' => \['statements' => \['break', 'clone', 'continue', 'echo\_print', 'return', 'switch\_case', 'yield'\]\] | 不要な制御構造の括弧を削除します |
| 'no\_unneeded\_curly\_braces' => true | 不要な中括弧を削除します |
| 'return\_type\_declaration' => \['space\_before' => 'none'\] | 戻り値の型宣言の前にスペースを挿入しません |
| 'single\_class\_element\_per\_statement' => \['elements' => \['const', 'property'\]\] | 一つのステートメントに一つのクラス要素（const、property）を持たせます |
| 'single\_import\_per\_statement' => true | 一つのステートメントに一つのimportを持たせます |
| 'visibility\_required' => \['elements' => \['method', 'property'\]\] | メソッドとプロパティの可視性を必須にします |

### 5-3. 構文と命名に関する設定

| 設定キー | 説明 |
| --- | --- |
| 'array\_syntax' => \['syntax' => 'short'\] | 配列のシンタックスを短い（short）形式にします（例：\[1, 2, 3\]） |
| 'clean\_namespace' => true | 名前空間をクリーンアップします |
| 'compact\_nullable\_typehint' => true | nullable型ヒントをコンパクトにします |
| 'constant\_case' => \['case' => 'lower'\] | 定数を小文字にします |
| 'encoding' => true | ファイルエンコーディングを検証し、UTF-8に設定します |
| 'full\_opening\_tag' => true | 短い形式のPHP開始タグ（<?）をフルの形式（<?php）に変更します |
| 'fully\_qualified\_strict\_types' => true | 厳密な型で完全修飾名を使います |
| 'general\_phpdoc\_tag\_rename' => true | 一部のPHPDocタグを他のものにリネームします |
| 'heredoc\_to\_nowdoc' => true | 可能な場合、heredocをnowdocに変換します |
| 'increment\_style' => \['style' => 'post'\] | インクリメント/デクリメントのスタイルを後置にします |
| 'integer\_literal\_case' => true | 整数リテラルのケースを小文字にします |
| 'lambda\_not\_used\_import' => true | 使用されていないlambda関数のインポートを削除します |
| 'list\_syntax' => true | 配列の分割代入のシンタックスを短い形にします |
| 'lowercase\_cast' => true | キャストを小文字にします |
| 'lowercase\_keywords' => true | PHPキーワードを小文字にします |
| 'lowercase\_static\_reference' => true | 静的参照の'static'キーワードを小文字にします |
| 'magic\_method\_casing' => true | マジックメソッドの名前を小文字にします |
| 'magic\_constant\_casing' => true | マジック定数を小文字にします |
| 'no\_alias\_functions' => true | 関数のエイリアスを元の名前に戻します |
| 'no\_alias\_language\_construct\_call' => true | 言語構造のエイリアスを元の名前に戻します |
| 'no\_alternative\_syntax' => true | 代替構文（例えば "endif;"）を使用しないようにします |
| 'no\_binary\_string' => true | バイナリ文字列を禁止します |
| 'no\_mixed\_echo\_print' => \['use' => 'echo'\] | echoとprintの混在を禁止し、echoのみを使用します |
| 'normalize\_index\_brace' => true | 配列と文字列の添字に一貫した中括弧スタイルを使用します |
| 'not\_operator\_with\_successor\_space' => true | 否定演算子の後のスペースを追加します |
| 'nullable\_type\_declaration' => true | NULL可能な型の宣言を許可します |
| 'nullable\_type\_declaration\_for\_default\_null\_value' => \['use\_nullable\_type\_declaration' => false\] | デフォルト値がnullの場合のNULL可能な型の宣言を使用しません |
| 'object\_operator\_without\_whitespace' => true | オブジェクト演算子の前後の空白を削除します |
| 'ordered\_imports' => \['sort\_algorithm' => 'alpha'\] | importをアルファベット順に並べ替えます |
| 'psr\_autoloading' => false | PSRオートローディング規格に従います（falseなら適用しない） |

### 5-4. コメントとドキュメンテーションに関する設定

| 設定キー | 説明 |
| --- | --- |
| 'general\_phpdoc\_tag\_rename' => true | 一部のPHPDocタグを他のものにリネームします |
| 'no\_blank\_lines\_after\_phpdoc' => true | PHPDocの後の空行を削除します |
| 'no\_empty\_phpdoc' => true | 内容のないPHP |
| 'no\_superfluous\_phpdoc\_tags' => \['allow\_mixed' => true, 'allow\_unused\_params' => true\] | 不要なPHPDocタグを削除しますが、mixedと未使用のパラメータは許可します |
| 'phpdoc\_indent' => true | PHPDocのインデントを修正します |
| 'phpdoc\_inline\_tag\_normalizer' => true | PHPDocのインラインタグを正規化します |
| 'phpdoc\_no\_access' => true | [@access](https://qiita.com/access "access") タグの使用を禁止します |
| 'phpdoc\_no\_package' => true | [@package](https://qiita.com/package "package") タグの使用を禁止します |
| 'phpdoc\_no\_useless\_inheritdoc' => true | 無用な [@inheritdoc](https://qiita.com/inheritdoc "inheritdoc") タグの使用を禁止します |
| 'phpdoc\_order' => \['order' => \['param', 'return', 'throws'\]\] | PHPDocタグを特定の順序で並べ替えます |
| 'phpdoc\_scalar' => true | 非複合スカラータイプの修飾を削除します |
| 'phpdoc\_single\_line\_var\_spacing' => true | 単一行の [@var](https://qiita.com/var "var") PHPDocアノテーションのスペーシングを修正します |
| 'phpdoc\_summary' => false | PHPDocの要約が必要な場合があります（falseなら要求しない） |
| 'phpdoc\_to\_comment' => false | PHPDocを通常のコメントに変換します（falseなら変換しない） |
| 'phpdoc\_tag\_type' => \['tags' => \['inheritdoc' => 'inline'\]\] | [@inheritdoc](https://qiita.com/inheritdoc "inheritdoc") タグをインラインで使用します |
| 'phpdoc\_trim' => true | PHPDocの前後の不要な空白を削除します |
| 'phpdoc\_types' => true | PHPDocのタイプを小文字にします |
| 'phpdoc\_var\_without\_name' => true | [@var](https://qiita.com/var "var") タグから変数名を削除します |
| 'single\_line\_comment\_style' => \['comment\_types' => \['hash'\]\] | 一行コメントのスタイルを変更します（#を使用） |

### 5-5. Laravelに関する設定

| 設定キー | 説明 |
| --- | --- |
| 'Laravel/laravel\_phpdoc\_alignment' => true | LaravelのPHPDocのアライメントを調整します |

## 6\. Pint実行

設定ファイル作成後、下記のコマンドで実行できます。

```bash
$ ./vendor/bin/pint
```

また、特定のファイルやディレクトリを対象に実行することも可能です！

```bash
$ ./vendor/bin/pint app/Models
$ ./vendor/bin/pint app/Models/User.php
```

## 7\. さいごに

他のプリセットも用意されているため、是非どういうルールが設定されているか確認してみてください。  
私はPintのルールベースはプリセットを利用し、特定のルールのみを上書きする使い方を検討してみたいと思います。

## 8\. 参考サイト

- [Laravel Pint 9.x Laravel](https://readouble.com/laravel/9.x/ja/pint.html)
- [PHP-CS-Fixer Configurator](https://mlocati.github.io/php-cs-fixer-configurator/#version:3.22)
- [pint](https://github.com/laravel/pint)

[0](https://qiita.com/aosan/items/#comments)

[18](https://qiita.com/aosan/items/333d048f412bc293dc53/likers)

10

ストックを更新しました