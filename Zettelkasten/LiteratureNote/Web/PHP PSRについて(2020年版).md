---
title: PHP PSRについて(2020年版)
source: https://qiita.com/tuuuuuuken/items/cd5f84ea0aa4374b0a91
author:
  - "[[tuuuuuuken]]"
published: 2020-11-18
created: 2025-05-22
description: アクシス Advent Calendar 2020 9日目担当のtuuuuuukenです!普段の業務ではPHPを書くことが多いのですが、PSR-4やPSR-2など個別の規約については知っているもののPSRそのものや他の規約についてはあまり知らなかったので調べてみました…
tags:
  - web
  - PHP
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から3年以上が経過しています。

[アクシス Advent Calendar 2020](https://qiita.com/advent-calendar/2020/mediaxis) 9日目担当のtuuuuuukenです!

普段の業務ではPHPを書くことが多いのですが、PSR-4やPSR-2など個別の規約については知っているものの  
PSRそのものや他の規約についてはあまり知らなかったので調べてみました。

## PSRとは

[PSR](https://www.php-fig.org/psr/) (*PHP Standards Recommendations*, PHP標準勧告)とは、 [PHP-FIG](https://www.php-fig.org/) (*The PHP Framework Interop Group*, PHPフレームワーク相互運用グループ)が策定しているPHPの規約です。  
PSRを定めることによってフレームワークやライブラリ間での相互連携を可能とし、PHP(とそのエコスシステム)を発展させるために存在しています。

その一覧は [https://www.php-fig.org/psr/](https://www.php-fig.org/psr/) 及び [https://github.com/php-fig/fig-standards](https://github.com/php-fig/fig-standards) で見ることができます。

また、 **PHP標準勧告** なんて名前がついているので必ず守るべき規約のようにも見えますが  
PHP-FIG(つまり公式団体ではない)が策定した規約ですので必ず守る必要はありません。

## PSRの構成

2020年12月現在、承認されたPSRは13個あり、4つの分類で構成されています。

1. オートローディング
2. インターフェース
3. HTTP
4. コーディングスタイル

## オートローディング

ファイルパスからクラスを自動ロードするための仕様について定められています。  
現状はPSR-4の1つのみで、PSR-0については非推奨となっています。

### PSR-4: Autoloading Standard

自動ロードのためのPHPの名前空間(namespace)及びクラス名の規約について定義されています。  
このようなフォーマットになっています。 `\<NamespaceName>(\<SubNamespaceNames>)*\<ClassName>`

## インターフェース

フレームワークで提供されることが多い機能(HTTPを除く)についてのインターフェース仕様について定められています。

### PSR-3: Logger Interface

ロガーに関するInterfaceについて定められたPSR。  
`debug`, `info`, `notice` など8つのログ出力メソッドや、それらのログレベルを受け取る `log` メソッドなどについて定義されています。  
有名なライブラリだと [Monolog](https://github.com/Seldaek/monolog/blob/master/src/Monolog/Logger.php) などが実装していますね。

### PSR-6: Caching Interface

itemの有効期限やキーのフォーマット、サポートすべきデータ型などについて定義されています。

### PSR-11: Container Interface

DIコンテナーの標準化について定義されています。  
インターフェースはとてもシンプルで `get` 及び `has` の2つが定義されています。  
また、面白いのが推奨される使用方法についても言及している点です。

> オブジェクトが独自の依存関係を取得できるように、ユーザーはコンテナをオブジェクトに渡すべきではありません。これは、コンテナがサービスロケーターとして使用されることを意味 します。これは一般的に推奨されないパターンです。

気になる方は以下を見てみてください。  
[https://www.php-fig.org/psr/psr-11/#13-recommended-usage](https://www.php-fig.org/psr/psr-11/#13-recommended-usage)  
[https://www.php-fig.org/psr/psr-11/meta/#4-recommended-usage-container-psr-and-the-service-locator](https://www.php-fig.org/psr/psr-11/meta/#4-recommended-usage-container-psr-and-the-service-locator)

### PSR-13: Hypermedia Links

ハイパーメディアリンクについて定義されているらしいです。が、いまいち使い道がわかりませんでした。  
phalconがPSR-13を実装しているようでしたのでそちらのドキュメントも読んでみましたがよくわからず...(これを気にphalcon初めてみるのも良さそうです)  
[https://docs.phalcon.io/4.0/ja-jp/html-link#html-link-psr-13](https://docs.phalcon.io/4.0/ja-jp/html-link#html-link-psr-13)

### PSR-14: Event Dispatcher

Observerパターンの仕組みを提供するためのインターフェースなどについて定義されています。

### PSR-16: Simple Cache

キャッシュライブラリ向けのインターフェースなどについて定義されています。  
PSR-6もキャッシュについて定義されていますが、PSR-16の方がよりシンプルみたいです。

> PSR-6はすでにこの問題を解決していますが、最も単純なユースケースで必要とされるものに対してはかなり形式的で冗長な方法で解決しています。このより単純なアプローチは、一般的なケースのための標準化された合理化されたインターフェースを構築することを目的としています。PSR-6から独立していますが、PSR-6との互換性をできるだけ簡単にするように設計されています。

## HTTP

PHPでHTTPを扱う上でよく利用する、HTTPクライアントやリクエストハンドラ、ミドルウェアなどについて定められています。

### PSR-7: HTTP message interfaces

HTTPのメッセージオブジェクト（HTTPリクエスト及びHTTPレスポンス)について定義されています。

### PSR-15: HTTP Server Request Handlers

リクエストハンドラー(MVCで言う所のC)について定義されています。  
PSR-7で定義されたHTTPリクエストを引数に受け取り、HTTPレスポンスを返すインターフェースについて定義されています。

### PSR-17: HTTP Factories

PSR-7で定義されたリクエスト及びレスポンスを生成するためのファクトリに必要なインターフェースについて定義されています。

### PSR-18: HTTP Client

HTTPクライアントについての定義です。  
簡単に言うとPSR-15の逆(つまりリクエストを送る側について)ですね。

## コーディングスタイル

コーディング規約について定義されています。PSR-2については上位互換のPSR-12が承認されたため非推奨となっています。

### PSR-1: Basic Coding Standard

最低限守るべきコーディング規約について定義されています。  
PHPタグ(`<?php` or `<?=`)についてやファイルエンコード(BOM無しUTF-8）などなど。

### PSR-12: Extended Coding Style

細かいコーディング規約及びコーディングスタイルについて定義されています。  
use文の順番(クラス、関数、定数の順)やextendsとimplementsは同じ行に1行で書きましょう、などなど。  
PSR-2と比べるとPHP7で追加された引数と戻り値の型宣言の規約などが追加されています。

## まとめ

使わないPSRについはあまり知る機会がないのでこの機会に知ることができて楽しかったです。  
PHP8がリリースされたので新たなPSRも投稿されそうですね。

明日はmichidaさんの「スキャフォールドツール作った話」です!  
それではみなさん良いPHPライフを!

---

おまけ

## PSR一覧

## ACCEPTED(承認済み)

| 番号 | タイトル | リンク |
| --- | --- | --- |
| 1 | Basic Coding Standard | [https://www.php-fig.org/psr/psr-1/](https://www.php-fig.org/psr/psr-1/) |
| 3 | Logger Interface | [https://www.php-fig.org/psr/psr-3/](https://www.php-fig.org/psr/psr-3/) |
| 4 | Autoloading Standard | [https://www.php-fig.org/psr/psr-4/](https://www.php-fig.org/psr/psr-4/) |
| 6 | Caching Interface | [https://www.php-fig.org/psr/psr-6/](https://www.php-fig.org/psr/psr-6/) |
| 7 | HTTP Message Interface | [https://www.php-fig.org/psr/psr-7/](https://www.php-fig.org/psr/psr-7/) |
| 11 | Container Interface | [https://www.php-fig.org/psr/psr-11/](https://www.php-fig.org/psr/psr-11/) |
| 12 | Extended Coding Style Guide | [https://www.php-fig.org/psr/psr-12/](https://www.php-fig.org/psr/psr-12/) |
| 13 | Hypermedia Links | [https://www.php-fig.org/psr/psr-13/](https://www.php-fig.org/psr/psr-13/) |
| 14 | Event Dispatcher | [https://www.php-fig.org/psr/psr-14/](https://www.php-fig.org/psr/psr-14/) |
| 15 | HTTP Handlers | [https://www.php-fig.org/psr/psr-15](https://www.php-fig.org/psr/psr-15) |
| 16 | Simple Cache | [https://www.php-fig.org/psr/psr-16](https://www.php-fig.org/psr/psr-16) |
| 17 | HTTP Factories | [https://www.php-fig.org/psr/psr-17](https://www.php-fig.org/psr/psr-17) |
| 18 | HTTP Client | [https://www.php-fig.org/psr/psr-18](https://www.php-fig.org/psr/psr-18) |

## DRAFT(草案)

| 番号 | タイトル | リンク |
| --- | --- | --- |
| 5 | PHPDoc Standard | [https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc.md](https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc.md) |
| 19 | PHPDoc tags | [https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc-tags.md](https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc-tags.md) |

## ABANDONED(放棄)

| 番号 | タイトル | リンク |
| --- | --- | --- |
| 8 | Huggable Interface | [https://github.com/php-fig/fig-standards/blob/master/proposed/psr-8-hug/](https://github.com/php-fig/fig-standards/blob/master/proposed/psr-8-hug/) |
| 9 | Security Advisories | [https://github.com/php-fig/fig-standards/blob/master/proposed/security-disclosure-publication.md](https://github.com/php-fig/fig-standards/blob/master/proposed/security-disclosure-publication.md) |
| 10 | Security Reporting Process | [https://github.com/php-fig/fig-standards/blob/master/proposed/security-reporting-process.md](https://github.com/php-fig/fig-standards/blob/master/proposed/security-reporting-process.md) |

## DEPRECATED（非推奨)

| 番号 | タイトル | リンク |
| --- | --- | --- |
| 0 | Autoloading Standard | [https://www.php-fig.org/psr/psr-0](https://www.php-fig.org/psr/psr-0) |
| 2 | Coding Style Guide | [https://www.php-fig.org/psr/psr-2](https://www.php-fig.org/psr/psr-2) |

[0](https://qiita.com/tuuuuuuken/items/#comments)

[30](https://qiita.com/tuuuuuuken/items/cd5f84ea0aa4374b0a91/likers)

15

ストックを更新しました