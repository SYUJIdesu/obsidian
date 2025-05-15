---
title: "LaravelにLog viewerを追加する"
source: "https://www.kk-crap.info/add_log-viewer_to_laravel/"
author:
  - "[[まるお]]"
published: 2023-01-21
created: 2025-05-15
description: "laravelで開発を行う際、logを都度ファイルから見るのはめんどうなので、logを見られるGUIを探してみました正しく導入できると以下のような画面を出すことができます。実行環境Docker container php:7.4-apach"
tags:
  - "web"
  - "Laravel"
---
laravelで開発を行う際、logを都度ファイルから見るのはめんどうなので、logを見られるGUIを探してみました  

正しく導入できると以下のような画面を出すことができます。  

![](https://www.kk-crap.info/wp-content/uploads/2023/01/image-1-1024x210.png)

![](https://www.kk-crap.info/wp-content/uploads/2023/01/image-1024x420.png)

**実行環境**  
Docker container php:7.4-apache  
PHP: 7.4.33 (cli)

## ARCANEDEV/LogViewerのinstall

基本は公式ドキュメントに従います。  
[https://github.com/ARCANEDEV/LogViewer/blob/master/\_docs/1.Installation-and-Setup.md](https://github.com/ARCANEDEV/LogViewer/blob/master/_docs/1.Installation-and-Setup.md)  

## install

phpのversionに合わせてversionを指定してinstallします。php:7.4.3はlog-viewer:7.0.0で大丈夫のようでした。

```bash
composer require arcanedev/log-viewer:'7.0.0'
```

## config/app.phpの変更

installしたライブラリのclassをconfigに追加します

```php
'providers' => [
    ...
    Arcanedev\LogViewer\LogViewerServiceProvider::class,
],
```

configの変更の反映のため、以下のコマンドを実行します

```bash
php artisan log-viewer:publish
```

これで *config/log-viewer.php* が作成されるはずです。

## log出力の変更

デフォルトでは *storage/logs/<laravel-YYYY-MM-DD.log>* を取得するように設定されています。これはlaravelのlog出力の *daily* 設定に合わせてあるため、 *.env* ファイルを変更します。

```bash
APP_LOG=single
LOG_CHANNEL=daily
```

この状態で `php artisan tinker` などでログを出力してみると、 `laravel-2023-01-21.log` のようなlogファイルが作成されるはずです。

出力設定をdaily以外で扱いたい場合は、log-viewerの方の設定をいじってください。 `log-viewer.php` の `pattern` を適宜変更すれば大丈夫だと思います。  

```php
...
'pattern'       => [
        'prefix'    => Filesystem::PATTERN_PREFIX,    // 'laravel-'
        'date'      => Filesystem::PATTERN_DATE,      // '[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]'
        'extension' => Filesystem::PATTERN_EXTENSION, // '.log'
    ],
```

## 画面の確認

この状態で http://<laravel-URL>/log-viewer (デフォルトなら `[http://localhost:8000/log-viewer](http://localhost:8000/log-viewer)` )にアクセスして、以下の画面が表示されれば導入完了です。  

![](https://www.kk-crap.info/wp-content/uploads/2023/01/image-2-1024x344.png)![](https://www.kk-crap.info/wp-content/themes/cocoon-master/images/no-image-160.png)

[

Pytorch Tips

](https://www.kk-crap.info/pytorch-tips/ "Pytorch Tips")[![](https://www.kk-crap.info/wp-content/themes/cocoon-master/images/no-image-160.png)

phpでcall to undefined function が起きた時の対処法

](https://www.kk-crap.info/php-call-undefined-function/ "phpでcall to undefined function が起きた時の対処法")

[ホーム](https://www.kk-crap.info/)

- [ホーム](https://www.kk-crap.info/)
- 
- トップ
-

タイトルとURLをコピーしました