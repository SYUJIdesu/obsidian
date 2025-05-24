---
title: Swooleを使う前に知っておきたい基礎の基礎
source: https://qiita.com/prograti/items/e29210300232e4a3d8b7
author:
  - "[[prograti]]"
published: 2020-05-14
created: 2025-05-22
description: 近年Swooleに関する記事を見かける機会も増えてきました。LaravelやSymfonyといったフレームワークでもSwoole拡張のためのパッケージが登場しています。これらのフレームワークを利用す…
tags:
  - web
  - アーキテクチャ
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から5年以上が経過しています。

近年Swooleに関する記事を見かける機会も増えてきました。LaravelやSymfonyといったフレームワークでもSwoole拡張のためのパッケージが登場しています。これらのフレームワークを利用すればSwooleのことをあまり知らなくてもアプリケーションを構築できるでしょう。しかし、Swooleについての基礎知識がないまま構築してしまうと、本番リリース後に予期せぬ問題を抱え込んでしまうかもしれません。

そこで本記事ではSwooleの基礎の基礎についてまとめてみたいと思います。

## Swooleとは

Swooleを知らない方のために、まずはSwooleについて簡単に紹介します。

[![マスコット](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/9a0ed77c-e2f6-3161-0bde-3725608cae34.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F9a0ed77c-e2f6-3161-0bde-3725608cae34.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7805176aadd5bbaf410b4e11d37f3d81)

SwooleはC/C++で作られたPHP拡張モジュールで、スケーラブルなネットワークアプリケーションを構築するために設計された非同期・イベント駆動モデルのフレームワークです。複数の通信プロトコルを備えたネットワークサーバとクライアントモジュールを提供し、高パフォーマンスを求められるWebサービス、WebSocketサービス、IoT、リアルタイムコミュニケーション、ゲーム、マイクロサービスといった分野で活用されています。現在は分かりませんが、TencentやBaidu、Bilibiliなどで採用実績があるようです。

公式サイト: [https://www.swoole.co.uk/](https://www.swoole.co.uk/)  
GitHub: [https://github.com/swoole/swoole-src](https://github.com/swoole/swoole-src)

### パフォーマンス検証

詳しい説明に入る前に、まずはPHP-FPMとSwooleサーバ上でプログラムを実行した場合のパフォーマンスを検証してみます。検証にはDBにレコードを1件登録するだけの簡単なプログラムを使用します。

■PHP-FPM版

```php
<?php

$pdo = new PDO('mysql:dbname=test;host=localhost;charset=utf8mb4', 'user', 'pass');

$stmt = $pdo->prepare('INSERT INTO logs(ip_address, created_at) VALUES (?, ?)');
$stmt->bindValue(1, $_SERVER['REMOTE_ADDR']);
$stmt->bindValue(2, (new DateTime())->format('Y-m-d H:i:s'));
$stmt->execute();

echo "<h1>\nHello World.\n</h1>";
```

■Swoole版

server.php

```php
<?php

use Swoole\Http\Server;
use Swoole\Http\Response;
use Swoole\Http\Request;

$server = new Server('127.0.0.1', 9501);

$server->set([
    'hook_flags' => SWOOLE_HOOK_ALL,
    'enable_reuse_port' => true,
]);

$server->on('request', function (Request $request, Response $response) {
    $pdo = new PDO('mysql:dbname=test;host=localhost;charset=utf8mb4', 'user', 'pass');
    
    $stmt = $pdo->prepare('INSERT INTO logs(ip_address, created_at) VALUES (?, ?)');
    $stmt->bindValue(1, $request->server['remote_addr']);
    $stmt->bindValue(2, (new DateTime())->format('Y-m-d H:i:s'));
    $stmt->execute();
      
    $response->end("<h1>\nHello World.\n</h1>");
});

$server->start();
```

負荷テストツールを使ってそれぞれのプログラムにリクエストを送ってみます。

```shell
# 10スレッドで100コネクションを10秒間
wrk -t10 -c10 -d10s http://localhost/xxxxx
```

結果はPHP-FPMが432.84rps、Swooleが629.23rpsとなり、スループットを比較するとSwoole版の方がPHP-FPM版より良い結果になりました。

■PHP-FPM版

```shell
Running 10s test @ http://localhost/xxxxx
  10 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   229.17ms   49.01ms 608.46ms   91.19%
    Req/Sec    45.05     24.26   110.00     50.45%
  4333 requests in 10.01s, 863.21KB read
Requests/sec:    432.84
Transfer/sec:     86.23KB
```

■Swoole版

```shell
Running 10s test @ http://localhost/xxxxx
  10 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   158.09ms   31.68ms 665.99ms   91.21%
    Req/Sec    63.58     24.30   101.00     63.45%
  6300 requests in 10.01s, 1.12MB read
Requests/sec:    629.23
Transfer/sec:    114.29KB
```

このテストケースだけでパフォーマンスの優劣をつけることはできませんが、Swooleが多くのリクエストを処理できるという事実は確認できました。次に負荷テスト実行時のプロセスの様子を見てみましょう。以下は `htop` の画面をキャプチャしてアニメーションGIF化したものです。

■PHP-FPM版

今回はプロセスマネージャを `pm = dynamic` および `pm.max_children = 15` と設定しているため、テスト実行時に子プロセスが動的に最大値の15まで生成されています。CPU使用率はプロセス生成時を除けば子プロセスごとに2%前後で安定しています。子プロセスごとにメモリが割り当てられるため、子プロセス数の増加に比例してメモリ使用量も増加していきます（今回のケースだと起動時の約150MBから約440MBへ増加）。

[![php-fpm](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2Fb4d4202b-e1ad-76cf-7187-7076ccc3f4db.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=cc361a8feefd8688703bae20100f9f2b)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2Fb4d4202b-e1ad-76cf-7187-7076ccc3f4db.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=cc361a8feefd8688703bae20100f9f2b)

■Swoole版

Swooleサーバ起動時にマスタープロセスとスレッドおよび子プロセスが生成されています。テスト実行時のCPU使用率はスレッドが3%前後、子プロセスが6~7%となっています。スレッドや子プロセスの数に変化はありません。

[![Swoole](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F637270ef-5744-7a74-c387-cd9bb316b4f5.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e073e921f4fdef4b425732cba4562a0d)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F637270ef-5744-7a74-c387-cd9bb316b4f5.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e073e921f4fdef4b425732cba4562a0d)

## PHP-FPMとSwooleサーバの動作の違い

プロセスの使い方が異なることは確認できましたので、PHP-FPMとSwooleサーバの動作の違いについてもう少し詳しく見てみましょう。

### PHP-FPM

PHP-FPMやPHP-CLIはプロセスベースモデルで動作します。このモデルでは１つのプロセスで処理されるリクエストは１つだけで、１つのプロセスで複数のリクエストが並行して処理されることはありません。PHP-FPMはマルチプロセスで動作しますので、下図のように各プロセスが独立してシーケンシャルにリクエストを処理していきます。

[![マルチプロセス処理のイメージ](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/2f03078d-3359-d324-62e7-9fc21e8b7ede.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F2f03078d-3359-d324-62e7-9fc21e8b7ede.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1e72ba22a0486a923c465a808d8e6150)

PHPのライフサイクルは大まかに以下のようになっており、プロセスの開始・終了時に 1 と 5 がそれぞれ1回ずつ実行され、リクエストごとに2~4が繰り返される形になります。

1. php\_module\_startup（php.iniのロード、グローバル定数のセット 他）
2. php\_request\_startup（リクエスト情報のセット 他）
3. php\_execute\_script（PHPスクリプトのコンパイル、実行 他）
4. php\_request\_shutdown（出力バッファのフラッシュ、\_\_destructの呼出 他）
5. php\_module\_shutdown（グローバルオブジェクトの破棄 他）

[![PHPのライフサイクル](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/72124628-5f1f-f470-2031-df02a22587fd.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F72124628-5f1f-f470-2031-df02a22587fd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ffdc4175fd7b2d805eb861720793856c)

PHP-FPMは起動時にマスタープロセスを生成し、さらにphp-fpm.confの設定に従ってワーカープールごとにワーカープロセス（子プロセス）を生成します。マスタープロセスはワーカープロセスの状態監視やプロセスの生成、破棄を行い、ワーカープロセスがキューからリクエストを取り出して処理を行います。

[![PHP-FPMの処理モデル](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/934ff8ae-3ab8-412e-f041-ab61e9645aeb.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F934ff8ae-3ab8-412e-f041-ab61e9645aeb.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0752c5ddb9ea369f0e874efa09ea1a1a)

以上がPHP-FPMの動作の概要になります。

### Swoole

Swooleサーバは前述の負荷テストで用いたような起動スクリプトをPHP-CLIで実行して起動します。

```shell
$ php server.php
```

つまり、PHPのライフサイクルで見ると、 `php_execute_script` の所でサーバが起動し、実際のリクエストはPHP-FPMの時のようなstartup⇒execute⇒shutdownとは別のサイクルで逐次処理される形になります。

[![SwooleのPHPライフサイクル](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/1d3a5750-8f5a-b2cd-6c74-ae347af88014.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F1d3a5750-8f5a-b2cd-6c74-ae347af88014.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=446e92e9b6498380c420bc682e460ecb)

試しに簡単なPHP拡張モジュールを作成して確認してみます。まず、PHPのソースコードをダウンロードしてスケルトンを作成するスクリプトを実行します。

```shell
$ php php-src/ext/ext_skel.php --ext hook
```

次に出来あがったスケルトンを修正してリクエストのstartupとshutdownの時にメッセージを出力するようにします。

あとはこのモジュールをビルドしてインストールします。

```shell
$ phpize
$ ./configure
$ make
$ make install
```

PHP-FPMの方にアクセスしてみるとstartupとshutdownの時のメッセージが表示されました。

[![PHP-FPMへのアクセス](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/764d033e-d219-c84c-f92b-83ed67e0cbc7.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F764d033e-d219-c84c-f92b-83ed67e0cbc7.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=aec0f4ccfd821d3c40934e9f2c695d79)

次にSwooleサーバを起動し、HTTPリクエストを送ってみます。すると起動時にstartupのメッセージが表示された後は、何回HTTPリクエストを送ってもshutdownのメッセージは表示されません。

```shell
$  php server.php
request initalize
Hello World.
Hello World.
Hello World.
```

プロセスベースモデルでは１つのプロセスで複数のリクエストが同時並行で処理されないと説明しました。それ故に、冒頭のパフォーマンステストではPHP-FPMは多くの子プロセスを動的に生成して多数の同時並行なリクエストを処理していました。では、多くの子プロセスを生成していないSwooleはどのように処理しているのでしょう

## Swooleサーバのアーキテクチャ

Swooleサーバの動作を知る上では **Reactorパターン** に関する予備知識があった方が理解しやすいので、先にReactorパターンについて簡単に触れておきます。

### Reactorパターン

Reactorパターンとは同時並行なリクエストを効率よく処理するためのデザインパターンです。情報工学者のダグラス・C・シュミット氏は論文 [^1] の中で以下のように紹介しています。

> The Reactor design pattern handles service requests that are delivered concurrently to an application by one or more clients. Each service in an application may consist of serveral methods and is represented by a separate event handler that is responsible for dispatching service-specific requests. Dispatching of event handlers is performed by an initiation dispatcher, which manages the registered event handlers. Demultiplexing of service requests is performed by a synchronous event demultiplexer.  
> （意訳：Reactorパターンは１つ以上のクライアントから同時並行で送られるリクエストを処理するためのデザインパターンです。アプリケーション内のサービスメソッドは、それぞれ独立したイベントハンドラとして実装されます。イベントハンドラの起動はイベントハンドラを管理する開始ディスパッチャーによって行われます。リクエストの多重分離はイベントデマルチプレクサーによって行われます。）

言葉が少し難しいのでイメージしやすいように図を使って補足します。以下の図はHTTPリクエストを受け付けて処理を行うアプリケーションを表しています（分かり易いように簡略化しています）。

[![Reactorパターンの例](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/3df0662f-e0f0-b336-92af-deecb0a6728c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F3df0662f-e0f0-b336-92af-deecb0a6728c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3530cff9cb549f3d27ff613205ca0c13)

`Event Demultiplexer` は待ち受けポート（例：80/443）に送られてくるリクエストを監視し、処理可能になったリクエストから順番にInitiation Dispatcherに渡していきます。 `Initiation Dispatcher` はイベントの到着を待機し、Event Demultiplexerから送られてきたイベントに合わせて適切なEvent Handlerに処理を委譲します。ポイントは多重リクエストを **１つのプロセスで処理できる** ということです。マルチプロセスモデルと比べてプロセス数が少なくて済み、メモリ消費量を抑えることが出来ますし、頻繁なコンテキストスイッチによるオーバーヘッドもありません。

[![マルチプロセス vs Reactor](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/b22fc701-8e24-f5a1-3357-0eab3712dde4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2Fb22fc701-8e24-f5a1-3357-0eab3712dde4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=84b04e540d773b115ccca5575fab5b55)

### アーキテクチャ概要

Swooleサーバの主要なコンポーネントには以下の３つがあります。

1. Reactor
2. Worker
3. Manager

それぞれのコンポーネントについて見ていきましょう。

#### Reactor

Reactorとは文字通りReactorパターンを実装したコンポーネントでEvent DemultiplexerやInitiation Dispatcherの役割を担います。Swooleサーバは用途別に2つのReactorを使い分けており、ここでは便宜的に `Master Reactor` と `Worker Reactor` と呼ぶことにします。

Master Reactorはサーバ（HTTP/TCP/WebSocket）の待ち受けポートのI/Oを担当するReactorです。クライアントからリクエストを受信したらリクエスト受付用イベントハンドラを呼び出します。Worker ReactorはパイプのI/Oを担当します。パイプとはプロセス間通信（IPC）に使用する単方向のデータチャネルで、コマンド間の入出力を繋げるときに使用する無名パイプ（ `|` ）はよくご存知かと思います。後述するWorkerは子プロセスで動作しますが、マスタープロセスと子プロセス間の通信はパイプで行われ、そのI/OはWorker Reactorの担当です。それ以外にもサーバアプリケーション側でSwooleの非同期I/Oモジュールを利用して外部APIと通信したりファイルの読み書きを行った時のI/OもWorker Reactorが担当します。

[![Master ReactorとWorker Reactor](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/02fd2029-8907-cdf5-a873-3758281f4191.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F02fd2029-8907-cdf5-a873-3758281f4191.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d761e9e07b7e4217fdbe471b1137a4ec)

Master Reactorはメインアプリケーションスレッドで動作しますが、Worker Reactorはスレッドプールのスレッドで動作します。ただし、サーバの `single_thread` オプションを指定すれば、Worker Reactorをメインアプリケーションスレッドで動作させることもできます。なお、スレッドプール数（ `reactor_num` ）のデフォルト値はCPUのコア数となります。

```php
$server = new Server('127.0.0.1', 9501);

$server->set([
    'single_thread' => true,
]);
```

[![シングルスレッドオプション指定](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/0ed7f254-1a12-43d2-15cc-de2132d678bb.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F0ed7f254-1a12-43d2-15cc-de2132d678bb.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3b0ff77e9982ba0dfb84da193d35db81)

なお、Worker Reactorはマルチスレッドで動きますが、PHPは通常のNTS(Non Thread Safe)のままで支障はありませんので、あえてZTS(Zend Thread Safe)にする必要はありません。

#### Worker

WorkerはWorker Reactorから送られてきたイベントをもとに適切なイベントハンドラを実行する役割を担います。つまり、このワーカープロセスがユーザアプリケーションを実行するプロセスになります。なお、ワーカープロセス数（ `worker_num` ）のデフォルト値はCPUのコア数となります。

ユーザアプリケーションがワーカープロセスで動作しているのか実際に確認してみます。冒頭のパフォーマンステストのコードを以下のように負荷の高い処理に変更して実行します。

server.php

```php
<?php

use Swoole\Http\Server;
use Swoole\Http\Response;
use Swoole\Http\Request;

$server = new Server('127.0.0.1', 9501);

$server->set([
    'hook_flags' => SWOOLE_HOOK_ALL,
    'enable_reuse_port' => true,
]);

$server->on('request', function (Request $request, Response $response) {
    password_hash("Hello World", PASSWORD_BCRYPT, ['cost' => 12]);
});

$server->start();
```

テストした結果が以下になります。ワーカープロセスのCPU使用率が急激に上昇していることが確認できます。

[![高負荷時のワーカープロセス](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F8fbe56b8-9725-0323-0b1a-a310c042bd15.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=27b5d40bc2f4e8e86f4ec5b4e4e954b6)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F8fbe56b8-9725-0323-0b1a-a310c042bd15.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=27b5d40bc2f4e8e86f4ec5b4e4e954b6)

このようにユーザアプリケーションはPHP-FPMと同様にワーカープロセスで動作するわけですが、ここでいくつかPHP-FPMとの挙動の違いを確認してみたいと思います。

##### static変数・staticプロパティの扱いが異なる

PHP-FPMで以下のコードを繰り返し実行しても表示されるのは `1` です。

■PHP-FPM版

```php
<?php

static $count = 0;
echo ++$count;
```

では、Swooleサーバで以下のコードを繰り返し実行するとどうなるでしょう？

■Swoole版

server.php

```php
<?php
$server->on('request', function (Request $request, Response $response) {
    static $count = 0;
    $response->end(++$count);
});
```

答えは `1, 2, 3, ...`とカウントアップしていきます。通常static変数はリクエスト終了時に解放されるはずですが、Swooleでは解放されません。なぜこのような結果になってしまったかというと、前述したPHPのライフサイクルの違いが関係しています。PHP-FPMはリクエストの度に2~4が繰り返されます。しかし、Swooleサーバは起動時に2、3が実行された後はリクエストの度に2~4が繰り返されることはありません。

1. php\_module\_startup
2. php\_request\_startup
3. php\_execute\_script
4. php\_request\_shutdown
5. php\_module\_shutdown

static変数やstaticプロパティの解放は `php_request_shutdown` の中で実行されます [^2] 。そのためshutdownが実行されないSwooleでは後続のリクエストに引き継がれてしまいます。

Swooleサーバ上で動作するDIフレームワークやMVCフレームワークを使用する場合、使用するコンポーネントのスコープがシングルトンなのかリクエストなのか意識しておかないと、インスタンス変数が意図せずにリクエスト間で共有されてしまうということも発生しますので注意が必要です（ワーカープロセス間では共有されません）。

##### 拡張モジュールの処理結果が異なる

普段よく使用する `DateTime` はPHP コアに含まれている拡張モジュールです。ここで以下のようなコードを実行して結果を比較してみたいと思います。

■PHP-FPM版

```php
<?php

// 言語にタイ語が指定されていたらタイムゾーンをバンコクに変更する
if (isset($_GET['lang']) && $_GET['lang'] === 'th') {
    date_default_timezone_set('Asia/Bangkok');
}

echo date(DATE_RFC2822);
```

■Swoole版

server.php

```php
<?php
$server->on('request', function (Request $request, Response $response) {
    $lang = $request->get['lang'];
    // 言語にタイ語が指定されていたらタイムゾーンをバンコクに変更する
    if ($lang === 'th') {
        date_default_timezone_set('Asia/Bangkok');
    }
    $response->end(date(DATE_RFC2822));
});
```

まず、PHP-FPM版を実行してみると `lang` を指定しない場合は

```text
Wed, 01 Apr 2020 16:36:41 +0900
```

`lang=th` を指定した場合は

```text
Wed, 01 Apr 2020 14:36:59 +0700
```

となり、langの有/無を何回繰り返してもタイムゾーンはTokyoとBangkokで切り替わります。

今度はSwoole版を実行してみます。まず、 `lang` を指定しない場合は

```text
Wed, 01 Apr 2020 16:40:11 +0900
```

次に `lang=th` を指定した場合は

```text
Wed, 01 Apr 2020 14:41:08 +0700
```

ここまではPHP-FPMと同じですが、再度 `lang` 指定なしで表示すると

```text
Wed, 01 Apr 2020 14:42:50 +0700
```

タイムゾーンがBangkokのままになってしまいました。

なぜこのような結果になってしまったかというと、これもPHPのライフサイクルの違いが関係しています。拡張モジュールの [ソースコード](https://github.com/php/php-src/blob/3704947696fe0ee93e025fa85621d297ac7a1e4d/ext/date/php_date.c#L694) を確認すると分かるようにstartupとshutdownのタイミングで `timezone` をNULLにしていますが、Swooleではこのステップがないためtimezoneが後続のリクエストに引き継がれてしまいます。

php\_date.c

```c
PHP_RINIT_FUNCTION(date)
{
    if (DATEG(timezone)) {
        efree(DATEG(timezone));
    }
    DATEG(timezone) = NULL;
    DATEG(tzcache) = NULL;
    DATEG(last_errors) = NULL;

    return SUCCESS;
}

PHP_RSHUTDOWN_FUNCTION(date)
{
    if (DATEG(timezone)) {
        efree(DATEG(timezone));
    }
    DATEG(timezone) = NULL;
    if(DATEG(tzcache)) {
        zend_hash_destroy(DATEG(tzcache));
        FREE_HASHTABLE(DATEG(tzcache));
        DATEG(tzcache) = NULL;
    }
    if (DATEG(last_errors)) {
        timelib_error_container_dtor(DATEG(last_errors));
        DATEG(last_errors) = NULL;
    }

    return SUCCESS;
}
```

このように拡張モジュールによっては挙動が変わる場合がありますので注意が必要です。

##### Task Worker

ここまで、Workerの説明をしましたが、Workerにはこれとは別に `Task Worker` というものが存在します。Task Workerとは、時間のかかる遅いタスクを非同期で実行するためのワーカーで、ワーカープロセスとは別のプロセスで動作します。Task Workerにタスクがスローされた後、ワーカープロセスは後続の処理を続行することができ、タスクの完了は非同期でワーカープロセスに通知されます。

例えば、以下のようなコードを実行すると画面にはすぐにレスポンスが返って来ますが、5秒後にコンソールに'done'と表示されます。

server.php

```php
$server->set([
    'task_worker_num' => 2,
]);

$server->on('request', function (Request $request, Response $response) use ($server) {
    $server->task(null);
    $response->end("<h1>\nHello World.\n</h1>");
});

$server->on('task', function ($serv, $task_id, $worker_id, $data) {
    $pdo = new PDO('mysql:dbname=test;host=localhost;charset=utf8mb4', 'user', 'pass');
    $stmt = $pdo->prepare('select sleep(5)');
    $stmt->execute();
    return 'done';
});

// タスクにreturn値があった場合にのみ実行される
$server->on('finish', function ($serv, $task_id, $data) {
    echo $data . "\n";
});
```

WorkerとTask Worker間の通信はパイプが使用されますが、メッセージキュー（System V IPC）に変更することも出来ます。メッセージキューを使用した場合、Swooleサーバ起動時にキューに溜まっているメッセージはTask Workerによって自動的に処理されます。

server.php

```php
$server->set([
    'task_worker_num' => 2,
    'task_ipc_mode' => SWOOLE_IPC_MSGQUEUE,
    'message_queue_key' => 0x70001001,
]);
```

```shell
# キューにメッセージが溜まっている状態でサーバを停止
$ ipcs -q
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0x70001001 65536      vagrant    666        108          6
```

```shell
# サーバを再起動した後のキューの状態
$ ipcs -q
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0x70001001 65536      vagrant    666        0            0
```

Task Workerのプロセスはサーバ起動後に生成され、以降そのプロセスが再起動せずにタスクを処理し続けますが、リクエスト数の上限（ `max_request` ）を設定することで、上限に達した場合に再起動することが出来ます。サードパーティのライブラリにおけるメモリリークの回避策として利用すると良いでしょう。

server.php

```php
$server->set([
    'task_worker_num' => 2,
    'max_request' => 100,
]);
```

下図：Task Workerの再起動  
[![プロセスの再起動](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2Fdf2a44c5-37f7-5a01-3397-c9fd54c5daee.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=276cb006cce1236eb417a6e050e028a5)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2Fdf2a44c5-37f7-5a01-3397-c9fd54c5daee.gif?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=276cb006cce1236eb417a6e050e028a5)

#### Manager

ManagerはWorkerやTask Workerのプロセスプールを管理し、プロセスの生成や再起動を行うプロセスです。外部からシグナルを送信することでManagerを制御することも出来ます。

```php
<?php
// 全てのWorkerを再起動
Swoole\Process::kill(ManagerのプロセスID, SIGUSR1);

// Task Workerのみ再起動
Swoole\Process::kill(ManagerのプロセスID, SIGUSR2);
```

#### アーキテクチャまとめ

主要なコンポーネントのみの紹介でしたが、Swooleサーバのアーキテクチャを簡単にまとめると以下のような図になります。

[![アーキテクチャ図](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/741fb97a-dafc-4f94-9f54-43aa028aa038.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F741fb97a-dafc-4f94-9f54-43aa028aa038.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=fc1bf4d8cf0ef76294ee7702b41153ea)

[![プロセス詳細](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/ca64cce1-6eb4-7ea8-6055-66a90ec54462.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2Fca64cce1-6eb4-7ea8-6055-66a90ec54462.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=90704b48b5101b30710105014bb28278)

### コルーチン（Coroutine）

これまでSwooleサーバを中心に説明しましたが、コルーチンもSwooleの大きな特長の１つです。と言ってもコルーチン自体はSwoole独自のものではなく、昔からある概念で [Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%AB%E3%83%BC%E3%83%81%E3%83%B3) では以下のように説明されています。

> コルーチンとはプログラミングの構造の一種。サブルーチンがエントリーからリターンまでを一つの処理単位とするのに対し、コルーチンはいったん処理を中断した後、続きから処理を再開できる。接頭辞 co は協調を意味するが、複数のコルーチンが中断・継続により協調動作を行うことによる。  
> サブルーチンと異なり、状態管理を意識せずに行えるため、協調的処理、イテレータ、無限リスト、パイプなど、継続状況を持つプログラムが容易に記述できる。

コルーチンという言葉を初めて聞いた人にもイメージしやすいように大雑把に言い換えると、 `yield` を介してプログラム制御を転送できる一種のサブルーチンという感じになります。

ということで、yieldと言えばPHPにはジェネレータがありますので、簡単なサンプルを作ってみたいと思います。まずはジェネレータを使わない通常のプログラムです。以下のプログラムは外部APIを直列で5回実行しています。

server.php

```php
<?php

$serverSocket = @stream_socket_server('tcp://127.0.0.1:8080', $errno, $errstr, STREAM_SERVER_BIND | STREAM_SERVER_LISTEN);
if (! $serverSocket) {
    exit;
}
echo "Starting server...\n";

while ($clientSocket = @stream_socket_accept($serverSocket)) {
    $stream = stream_socket_client('tcp://freegeoip.app:443');
    stream_socket_enable_crypto($stream, true, STREAM_CRYPTO_METHOD_TLSv1_2_CLIENT);
    fwrite($stream, "GET /json/1.1.1.1 HTTP/1.1\r\nHost: freegeoip.app\r\nConnection: close\r\n\r\n");
    $content = stream_get_contents($stream);
    echo "1:" . strlen($content) . "\n";
    @stream_socket_shutdown($stream, STREAM_SHUT_WR);
    
    $stream = stream_socket_client('tcp://freegeoip.app:443');
    stream_socket_enable_crypto($stream, true, STREAM_CRYPTO_METHOD_TLSv1_2_CLIENT);
    fwrite($stream, "GET /json/1.0.0.1 HTTP/1.1\r\nHost: freegeoip.app\r\nConnection: close\r\n\r\n");
    $content = stream_get_contents($stream);
    echo "2:" . strlen($content) . "\n";
    @stream_socket_shutdown($stream, STREAM_SHUT_WR);
    
    $stream = stream_socket_client('tcp://freegeoip.app:443');
    stream_socket_enable_crypto($stream, true, STREAM_CRYPTO_METHOD_TLSv1_2_CLIENT);
    fwrite($stream, "GET /json/8.8.8.8 HTTP/1.1\r\nHost: freegeoip.app\r\nConnection: close\r\n\r\n");
    $content = stream_get_contents($stream);
    echo "3:" . strlen($content) . "\n";
    @stream_socket_shutdown($stream, STREAM_SHUT_WR);
    
    $stream = stream_socket_client('tcp://freegeoip.app:443');
    stream_socket_enable_crypto($stream, true, STREAM_CRYPTO_METHOD_TLSv1_2_CLIENT);
    fwrite($stream, "GET /json/8.8.4.4 HTTP/1.1\r\nHost: freegeoip.app\r\nConnection: close\r\n\r\n");
    $content = stream_get_contents($stream);
    echo "4:" . strlen($content) . "\n";
    @stream_socket_shutdown($stream, STREAM_SHUT_WR);
    
    $stream = stream_socket_client('tcp://freegeoip.app:443');
    stream_socket_enable_crypto($stream, true, STREAM_CRYPTO_METHOD_TLSv1_2_CLIENT);
    fwrite($stream, "GET /json/9.9.9.9 HTTP/1.1\r\nHost: freegeoip.app\r\nConnection: close\r\n\r\n");
    $content = stream_get_contents($stream);
    echo "5:" . strlen($content) . "\n";
    @stream_socket_shutdown($stream, STREAM_SHUT_WR);

    fwrite($clientSocket, "HTTP/1.1 200 OK\r\nConnection: close\r\nContent-Length: 7\r\n\r\nsuccess");
    @stream_socket_shutdown($clientSocket, STREAM_SHUT_WR);
}
```

このサーバプログラムに対してCLIからリクエストを送って実行時間を計測してみます。

■クライアント側

```shell
$ curl http://127.0.0.1:8080/ -s -o /dev/null -w  "%{time_starttransfer}\n"
2.478856
```

■サーバ側

```shell
$ php server.php
Starting server...
1:870
2:902
3:871
4:871
5:861
```

上から順番に送受信を繰りして約2.5秒ほど掛かりました。次にこのプログラムと同じ内容をジェネレータを使ったコルーチンで実装してみましょう（動作確認用の簡易実装なので細かい不具合や冗長的なコードはご容赦ください）。

少々長いですが、コードの後半部分がコルーチンの実装になります。

server.php

```php
<?php

class Server
{
    private $serverSocket;
    private $clientDeferred;
    private $readStreams = [];
    private $writeStreams = [];
    private $listenerId = 1;
    private $listeners = [];
    private $serverSocketListener;
    private $readListeners = [];
    private $writeListeners = [];
    private $immediateListeners = [];
    
    public function boot(callable $callback = null)
    {
        $this->setImmediate($this, $callback);
        $this->run();
    }
    
    private function run()
    {
        while (true) {
            $listeners = $this->immediateListeners;
            foreach ($listeners as $listener) {
                call_user_func($listener->disable);
                $result = ($listener->fn)($listener, $listener->value);
                if (is_null($result)) continue;
                handleCoroutine($result);
            }
            $this->selectStreams($this->readStreams, $this->writeStreams, empty($this->readListeners) && empty($this->writeListeners) ? 0 : null);
        }
    }
    
    public function listen(string $uri)
    {
        $this->serverSocket = @stream_socket_server($uri, $errno, $errstr, STREAM_SERVER_BIND | STREAM_SERVER_LISTEN);
        stream_set_blocking($this->serverSocket, false);
        
        $clientDeferred = &$this->clientDeferred;
        $this->serverSocketListener = $this->addListener('read', $this->serverSocket, static function ($listener, $stream) use (&$clientDeferred) {
            if (! $clientSocket = @stream_socket_accept($stream, 0)) return;
            
            $deferred = $clientDeferred;
            $clientDeferred = null;
            $deferred->resolve(new Socket($clientSocket));
            if (is_null($clientDeferred)) {
                call_user_func($listener->disable);
            }
        });
        
        call_user_func($this->serverSocketListener->disable);
    }
    
    public function accept()
    {
        if ($clientSocket = @stream_socket_accept($stream, 0)) {
            return new Fulfilled(new Socket($clientSocket));
        }
        $this->clientDeferred = new Deferred;
        call_user_func($this->serverSocketListener->enable);
        return $this->clientDeferred->promise();
    }
    
    public function addListener(string $type, $value, callable $fn)
    {
        $listener = new Listener;
        $listener->id = $this->listenerId++;
        $listener->type = $type;
        $listener->value = $value;
        $listener->fn = $fn;
        $listener->enable = function () use ($listener) {
            if ($listener->type === 'read') {
                $id = (int) $listener->value;
                $this->readListeners[$id] = $listener;
                $this->readStreams[$id] = $listener->value;
            } else if ($listener->type === 'write') {
                $id = (int) $listener->value;
                $this->writeListeners[$id] = $listener;
                $this->writeStreams[$id] = $listener->value;
            } else if ($listener->type === 'immediate') {
                $this->immediateListeners[$listener->id] = $listener;
            }
        };
        $listener->disable = function () use ($listener) {
            if ($listener->type === 'read') {
                $id = (int) $listener->value;
                unset($this->readListeners[$id], $this->readStreams[$id]);
            } else if ($listener->type === 'write') {
                $id = (int) $listener->value;
                unset($this->writeListeners[$id], $this->writeStreams[$id]);
            } else if ($listener->type === 'immediate') {
                unset($this->immediateListeners[$listener->id]);
            }
        };
        
        $this->listeners[$listener->id] = $listener;
        return $listener;
    }
    
    public function setImmediate($value, callable $fn)
    {
        $listener = $this->addListener('immediate', $value, $fn);
        call_user_func($listener->enable);
        return $listener;
    }
    
    private function selectStreams(array $readStreams, array $writeStreams, $timeout)
    {
        if (empty($readStreams) && empty($writeStreams)) return;
        if (@stream_select($readStreams, $writeStreams, $exceptStreams, $timeout) === false) return;
        
        foreach ($readStreams as $stream) {
            $fd = (int) $stream;
            if (!isset($this->readListeners[$fd])) continue;
            
            $listener = $this->readListeners[$fd];
            $result = ($listener->fn)($listener, $stream);
            if (is_null($result)) continue;
            handleCoroutine($result);
        }
        
        foreach ($writeStreams as $stream) {
            $fd = (int) $stream;
            if (!isset($this->writeListeners[$fd])) continue;
            
            $listener = $this->writeListeners[$fd];
            $result = ($listener->fn)($listener, $stream);
            if (is_null($result)) continue;
            handleCoroutine($result);
        }
    }
}

class Listener
{
    public $id;
    public $type;
    public $fn;
    public $value;
    public $enable;
}

class Socket
{
    private $socket;
    private $in;
    private $out;
    
    public function __construct($socket = null)
    {
        $this->socket = $socket;
        $this->in = new InputStream($this->socket);
        $this->out = new OutputStream($this->socket);
    }
    
    public function read()
    {
        return $this->in->read();
    }
    
    public function write(string $data)
    {
        return $this->out->write($data);
    }
    
    public function write_end(string $data)
    {
        $promise = $this->out->write($data);
        $out = $this->out;
        $promise->onResolve(function () use ($out) {
            $out->close();
        });

        return $promise;
    }
    
    public function close()
    {
        $this->in->close();
        $this->out->close();
    }
}

class OutputStream extends SplQueue
{
    private $socket;
    private $writeListener;
    
    public function __construct($socket)
    {
        stream_set_blocking($socket, false);
        stream_set_write_buffer($socket, 0);
        $this->socket = $socket;
        $out = &$this;
        $this->writeListener = server()->addListener('write', $this->socket, static function ($listener, $stream) use (&$out) {
            while (! $out->isEmpty()) {
                list($data, $writtenTotal, $deferred) = $out->shift();
                $written = fwrite($stream, $data);
                var_dump($written);exit;
                $written = (int) $written;
                if ($written === 0) {
                    $out->unshift([$data, $writtenTotal, $deferred]);
                    return;
                }
                
                if ($written < strlen($data)) {
                    $data = substr($data, $written);
                    $writes->unshift([$data, $writtenTotal + $written, $deferred]);
                    return;
                }
                
                $deferred->resolve($writtenTotal + $written);
            }

            if ($out->isEmpty()) {
                call_user_func($listener->disable);
            }
        });
    }
    
    public function write(string $data)
    {
        $written = 0;
        if ($this->isEmpty()) {
            $written = fwrite($this->socket, $data);
            $written = (int) $written;
            if ($written === strlen($data)) {
                return new Fulfilled($written);
            }
            $data = substr($data, $written);
        }
        $deferred = new Deferred;
        $this->push([$data, $written, $deferred]);
        call_user_func($this->writeListener->enable);
        return $deferred->promise();
    }
    
    public function close()
    {
        if ($this->socket) {
            @stream_socket_shutdown($this->socket, STREAM_SHUT_WR);
        }
        
        $this->socket = null;
    }
}

class InputStream
{
    private $socket;
    private $chunkSize;
    private $readListener;
    private $deferListener;
    private $deferred;
    
    public function __construct($socket, $chunkSize = -1)
    {
        stream_set_blocking($socket, false);
        stream_set_read_buffer($socket, 0);
        $this->socket = $socket;
        $this->chunkSize = $chunkSize;
        $deferred = &$this->deferred;
        $this->readListener = server()->addListener('read', $this->socket, static function ($listener, $stream) use (&$deferred) {
            $data = stream_get_contents($stream);
            call_user_func($listener->disable);
            $_deferred = $deferred;
            $deferred = null;
            $_deferred->resolve($data);
        });
        call_user_func($this->readListener->disable);
    }
    
    public function read()
    {
        $data = stream_get_contents($this->socket, $this->chunkSize);
        if ($data === '') {
            call_user_func($this->readListener->enable);
            $this->deferred = new Deferred;
            return $this->deferred->promise();
        }
        $this->deferred = new Deferred;
        $deferred = &$this->deferred;
        server()->setImmediate($data, static function ($data) use (&$deferred) {
            $_deferred = $deferred;
            $deferred = null;
            $_deferred->resolve($data);
        });
        return $this->deferred->promise();
    }
    
    public function close()
    {
        if ($this->socket) {
            @stream_socket_shutdown($this->socket, STREAM_SHUT_RD);
        }
        
        $this->socket = null;
    }
}

interface Promise
{
    public function onResolve(callable $resolver);
}

trait Resolvable
{
    private $resolved = false;
    private $result;
    private $resolver;
    
    public function onResolve(callable $resolver)
    {
        if ($this->resolved) {
            if ($this->result instanceof Promise) {
                $this->result->onResolve($resolver);
                return;
            }
            
            $result = $resolver($this->result);
            if (is_null($result)) return;
            handleCoroutine($result);
            return;
        }
        
        if (is_null($this->resolver)) {
            $this->resolver = $resolver;
            return;
        }
        
        if (! $this->resolver instanceof SplQueue) {
            $this->resolver = new class($this->resolver) extends SplQueue {
                public function __construct($resolver)
                {
                    $this->enqueue($resolver);
                }
                
                public function __invoke($value, $exception)
                {
                    foreach ($this as $resolver) {
                        $result = $resolver($value, $exception);
                        if ($result === null) continue;
                        handleCoroutine($result);
                    }
                }
            };
        }
        $this->resolver->enqueue($resolver);
    }
    
    private function resolve($value = null)
    {
        $this->resolved = true;
        $this->result = $value;
        if ($this->result instanceof Promise) {
            $this->result->onResolve($this->resolver);
            return;
        }
        
        $result = ($this->resolver)($this->result, null);               
        if (is_null($result)) return;
        handleCoroutine($result);
    }
    
    private function reject(Throwable $e)
    {
        $this->resolve(new Rejected($e));
    }
}

class Fulfilled implements Promise
{
    private $value;
    
    public function __construct($value = null)
    {
        $this->value = $value;
    }
    
    public function onResolve(callable $resolver)
    {
        $result = $resolver($this->value);
    }
}

class Rejected implements Promise
{
    private $exception;
    
    public function __construct($exception = null)
    {
        $this->exception = $exception;
    }
    
    public function onResolve(callable $resolver)
    {
        $result = $resolver(null, $this->exception);
    }
}

class Deferred
{
    private $promise;
    
    public function __construct()
    {
        $this->promise = new class implements Promise {
            use Resolvable {
                resolve as public;
                reject as public;
            }
        };
    }
    
    public function promise()
    {
        return new class($this->promise) implements Promise {
            private $promise;
            
            public function __construct(Promise $promise)
            {
                $this->promise = $promise;
            }

            public function onResolve(callable $resolver)
            {
                $this->promise->onResolve($resolver);
            }
        };
    }
    
    public function resolve($value = null)
    {
        $this->promise->resolve($value);
    }
    
    public function reject(Throwable $e)
    {
        $this->promise->reject($e);
    }
}

class CoroutineHandler implements Promise
{
    use Resolvable;
    
    public function __construct(Generator $coroutine)
    {
        try {
            $yielded = $coroutine->current();
            if (! $yielded instanceof Promise) {
                if (! $coroutine->valid()) {
                    $this->resolve($coroutine->getReturn());
                    return;
                }
                
                if (is_array($yielded)) {
                    $yielded = $this->bundle($yielded);
                }
            }
        } catch (Throwable $e) {
            $this->reject($e);
            return;
        }
        
        
        $resolver = function ($value) use ($coroutine, &$resolver) {
            try {
                $yielded = $coroutine->send($value);
                if (! $yielded instanceof Promise) {
                    if (! $coroutine->valid()) {
                        $this->resolve($coroutine->getReturn());
                        return;
                    }
                    
                    if (is_array($yielded)) {
                        $yielded = $this->bundle($yielded);
                    }
                }
                
                $yielded->onResolve($resolver);
            } catch (Throwable $e) {
                $this->reject($e);
            } 
        };
        
        $yielded->onResolve($resolver);
    }
    
    private function bundle($promises)
    {
        $deferred = new Deferred;
        $waiting = count($promises);
        $values = [];
        foreach ($promises as $key => $promise) {
            $promise->onResolve(function ($value) use (&$deferred, &$values, &$waiting, $key) {
                $values[$key] = $value;
                if (--$waiting === 0) {
                    $deferred->resolve($values);
                }
            });
        }
        return $deferred->promise();
    }
}

function handleCoroutine(Generator $coroutine)
{
    $handler = new CoroutineHandler($coroutine);
    $handler->onResolve(function ($value, $exception) {
        if ($exception) {
            throw $exception;
        }
    });
    return $handler;
}

function server()
{
    static $server;
    return $server ?? $server = new Server;
}

function request(string $url)
{
    $coroutine = function () use ($url) {
        extract(parse_url($url));
        $port = $port ?? ($scheme === 'https' ? 443 : 80);
        $path = $path ?? '/';
        $stream = stream_socket_client(sprintf('tcp://%s:%d', $host, $port), $errno, $errstr, null, STREAM_CLIENT_CONNECT | STREAM_CLIENT_ASYNC_CONNECT);
        if ($scheme === 'https') {
            stream_socket_enable_crypto($stream, true, STREAM_CRYPTO_METHOD_TLSv1_2_CLIENT);
        }
        $socket = new Socket($stream);
        yield $socket->write("GET {$path} HTTP/1.1\r\nHost: {$host}\r\nConnection: close\r\n\r\n");
        $data = yield $socket->read();
        $socket->close();

        return $data;
    };
    return handleCoroutine($coroutine()); 
}

server()->boot(function ($listener, $server) {
    $server->listen('tcp://127.0.0.1:8080');
    echo "Starting server...\n";
    while ($socket = yield $server->accept()) {
        $contents = yield [
            request('https://freegeoip.app/json/1.1.1.1'),
            request('https://freegeoip.app/json/1.0.0.1'),
            request('https://freegeoip.app/json/8.8.8.8'),
            request('https://freegeoip.app/json/8.8.4.4'),
            request('https://freegeoip.app/json/9.9.9.9'),
        ];
            
        foreach ($contents as $key => $content) {
            echo $key . ':' . strlen($content) . "\n";
        }

        yield $socket->write_end("HTTP/1.1 200 OK\r\nConnection: close\r\nContent-Length: 7\r\n\r\nsuccess");
    }
});
```

このプログラムの実行時間を計測してみます。

■クライアント側

```shell
$ curl http://127.0.0.1:8080/ -s -o /dev/null -w  "%{time_starttransfer}\n"
0.819793
```

■サーバ側

```shell
$ php server.php
Starting server...
0:870
1:902
2:871
4:861
3:871
```

前者のジェネレータを使わない実装に比べて1/3程度の時間で処理することが出来ました。この違いがどこにあるかというと、前者はネットワークI/O処理時にブロック状態となりプログラムの処理が進むことがありませんが、後者は [stream\_set\_blocking](https://www.php.net/manual/ja/function.stream-set-blocking.php) でストリームをノンブロッキングの状態にし、送信可能な接続から順次送信し、受信結果の読み取り可能な接続から順次処理を再開というように並行して複数の処理を進めているからです。

このようにプログラム制御によってCPUの占有状態を他の処理へ自発的に明け渡す方式を **ノン・プリエンプティブマルチタスク** （協調的マルチタスク）と呼びます。これと対になる方式として **プリエンプティブマルチタスク** というものがありますが、こちらはカーネルのスケジューラがタスクに応じてCPUの占有時間を調整しながら実行していきます。PHPの [Thread](https://www.php.net/manual/ja/class.thread.php) を使用したマルチスレッドプログラムは後者のプリエンプティブマルチタスク方式になります。

Swooleに独自のコルーチン実装が組み込まれたのはバージョン2系からで、それまでは非同期コールバックメソッドでコーディングするスタイルでした。

github.com/swoole/swoole-src/blob/v1.10.6/examples/mysql/real\_async.php

```php
<?php
$db = new swoole_mysql;
$server = array(
    'host' => '127.0.0.1',
    'user' => 'root',
    'password' => 'root',
    'database' => 'test',
);

$db->on('close', function() use($db) {
    echo "mysql is closed.\n";
});

$r = $db->connect($server, function ($db, $result)
{
    if ($result === false)
    {
        var_dump($db->connect_errno, $db->connect_error);
        die;
    }
    echo "connect to mysql server sucess\n";
    $sql = 'show tables';
    //$sql = "INSERT INTO \`test\`.\`userinfo\` (\`id\`, \`name\`, \`passwd\`, \`regtime\`, \`lastlogin_ip\`) VALUES (NULL, 'jack', 'xuyou', CURRENT_TIMESTAMP, '');";
    $db->query($sql, function (swoole_mysql $db, $r)
    {
        global $s;
        if ($r === false)
        {
            var_dump($db->error, $db->errno);
        }
        elseif ($r === true)
        {
            var_dump($db->affected_rows, $db->insert_id);
        }
        echo "count=" . count($r) . ", time=" . (microtime(true) - $s), "\n";
        //var_dump($r);
        $db->close();
    });
});
```

この方式はパフォーマンスは良いものの、コールバックが複数のレイヤーにネストされると保守性が著しく落ちるという欠点があります（いわゆるコールバック地獄）。コールバックとは異なる方法として、TencentのSwooleをベースにした [TSF](https://github.com/Tencent/tsf) というフレームワークが採用しているジェネレータを使ったコルーチンがあります。こちらは同期プログラミングのような形で非同期コードをコーディングすることができます。

github.com/Tencent/tsf/blob/master/examples/src/model/TestModel.php

```php
class TestModel {
     public function mysqlTest(){
         $sql = new Swoole\Client\MYSQL(array('host' => '127.0.0.1', 'port' => 3345, 'user' => 'root', 'password' => 'root', 'database' => 'test', 'charset' => 'utf-8',));
        $ret = (yield $sql ->query('show tables'));
        var_dump($ret);
        $ret = (yield $sql ->query('desc test'));
        var_dump($ret);
     }
}
```

github.com/Tencent/tsf/blob/master/examples/src/controller/TestController.php

```php
class TestController extends Controller {

    public function actionTest(){

        SysLog::info(__METHOD__, __CLASS__);
        $response = $this ->argv['response'];
        $res =(yield $this ->test());
        SysLog::debug(__METHOD__ ." res  == ".print_r($res, true), __CLASS__);
        $response ->end(" test response ");
        yield Swoole\Coroutine\SysCall::end('test for syscall end');
    }
    
    private function test(){

        $test  = new TestModel();
        $res = (yield $test ->udpTest());
        SysLog::info(__METHOD__ . " res == " .print_r($res, true), __CLASS__);
        if ($res['r'] == 0) {

            //yield success
            SysLog::info(__METHOD__. " yield success data == " .print_r($res['data'], true), __CLASS__);
            yield $res;
        }
        else{

            //yield failed
            SysLog::error(__METHOD__ . " yield failed res == " .print_r($res, true), __CLASS__);
            yield array(
                'r' => 1,
                'error_msg' => 'yield failed',
                 );
        }
    }
}
```

しかし、切り替えが必要なロジックの全てに `yield` を付けなければならず、処理が複雑になればなるほどコーディングミスを犯しやすくなります。そこでSwoole2.x以降ではジェネレータベースのコルーチンではなくネイティブコルーチンが実装されました。

以下はSwooleのコルーチンを使った簡単なサンプルです。

```php
<?php
$cid = go(function () {
    echo "coro 1 start\n";
    Co::yield();
    echo "coro 1 end\n";
});
echo "main 1\n";
go(function ($cid) {
    echo "coro 2 start\n";
    Co::resume($cid);
    echo "coro 2 end\n";
}, $cid);
echo "main 2\n";

// 実行結果
coro 1 start
main 1
coro 2 start
coro 1 end
coro 2 end
main 2
```

`go` 関数（ `swoole_coroutine_create` 関数のエイリアス）はコルーチンを作成するためのSwooleのビルトイン関数です。コルーチンの中で `co::yield` が呼ばれるとそこで一旦処理を中断しコルーチンから抜けます。そして、 `co::resume` が呼ばれると、再びコルーチンの中に戻ってきて中断した位置から処理を再開しています。

動きは何となくイメージできたと思いますが、Swooleのコルーチンがどのような仕組みになっているのか理解を深めるために、もう少し詳しく見てみましょう。

#### PHPのスタック管理

Swooleのコルーチンの仕組みを理解するためにはPHPのスタック管理について知っておく必要がありますので、はじめに簡単に触れておきます。PHPはZendエンジンのコンパイラで中間バイトコード（ `OpCode` [^3] ）に変換され、仮想マシンで実行されます。例えば先ほどのサンプルプログラムのOpCodeをデバッグ出力すると以下のようになります。

```shell
$ php -dopcache.enable_cli=1 -d opcache.opt_debug_level=0x10000 test.php

$_main: ; (lines=13, args=0, vars=1, tmps=5)
    ; (before optimizer)
    ; /home/vagrant/code/benchmark/swoole/test3.php:1-83
L0 (2):     INIT_FCALL 1 96 string("go")
L1 (2):     T1 = DECLARE_LAMBDA_FUNCTION string("")
L2 (6):     SEND_VAL T1 1
L3 (6):     V2 = DO_FCALL_BY_NAME
L4 (6):     ASSIGN CV0($cid) V2
L5 (7):     ECHO string("main 1
")
L6 (8):     INIT_FCALL 2 112 string("go")
L7 (8):     T4 = DECLARE_LAMBDA_FUNCTION string("")
L8 (12):    SEND_VAL T4 1
L9 (12):    SEND_VAR CV0($cid) 2
L10 (12):   DO_FCALL_BY_NAME
L11 (13):   ECHO string("main 2
")
L12 (83):   RETURN int(1)

{closure}: ; (lines=7, args=1, vars=1, tmps=1)
    ; (before optimizer)
    ; /home/vagrant/code/benchmark/swoole/test3.php:8-12
L0 (8):     CV0($cid) = RECV 1
L1 (9):     ECHO string("coro 2 start
")
L2 (10):    INIT_STATIC_METHOD_CALL 1 string("co") string("resume")
L3 (10):    SEND_VAR_EX CV0($cid) 1
L4 (10):    DO_FCALL
L5 (11):    ECHO string("coro 2 end
")
L6 (12):    RETURN null

{closure}: ; (lines=5, args=0, vars=0, tmps=1)
    ; (before optimizer)
    ; /home/vagrant/code/benchmark/swoole/test3.php:2-6
L0 (3):     ECHO string("coro 1 start
")
L1 (4):     INIT_STATIC_METHOD_CALL 0 string("co") string("yield")
L2 (4):     DO_FCALL
L3 (5):     ECHO string("coro 1 end
")
L4 (6):     RETURN null
```

上記は `OpArray` [^4] というOpCodeのセットをダンプ出力しているもので、実行される関数の情報になります。 `$_main` は最上位のメインルーチンを表し、疑似関数として表現されています。 `{closure}` はgo関数の引数として渡しているコールバック関数です。

ZendVMが上記のようなOpCodeを実行する訳ですが、処理の過程で関数の引数や局所変数（関数内だけで使用される変数）、呼び出し元のアドレスなどをメモリ領域に記録します。PHPにはこれらの情報（ **スタックフレーム** ）を記憶するためのスタック（ **VMスタック** ） [^5] が用意されています。VMスタックは256KBに初期化され、容量が不足した場合は新しいスタックが自動的に追加され、リンクリストの関係で関連付けられます。スタックフレームのメモリレイアウトは以下のようになっており、関数が実行されるたびにVMスタックにプッシュされます。

[![スタックフレームレイアウト](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/9db4144c-9d2e-ae13-c9fa-13d39714f085.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F9db4144c-9d2e-ae13-c9fa-13d39714f085.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7f87dd3285120d06ea4d4ed1894024a5)

スタックフレームの先頭には `zend_execute_data` [^6] という構造体が割り当てられます。 `opline` は現在実行しているOpCodeを指し、初期化時にOpArrayの開始位置にポイントされます。 `prev_execute_data` は前のスタックフレームへのポインタで、現在のスタックの実行が完了すると実行ポインタが指す先がこのフレームに変わります。

以下のプログラムを例に実行フローを辿ってみましょう。

```php
function f1() {
    return 1 + f2();
}

function f2() {
    return 1;
}

echo f1();
```

OpCodeを出力すると以下のようになります。

```shell
$_main: ; (lines=6, args=0, vars=0, tmps=1)
    ; (before optimizer)
    ; /home/vagrant/code/benchmark/swoole/test3.php:1-93
L0 (15):    NOP
L1 (19):    NOP
L2 (23):    INIT_FCALL 0 112 string("f1")
L3 (23):    V0 = DO_UCALL
L4 (23):    ECHO V0
L5 (93):    RETURN int(1)

f2: ; (lines=2, args=0, vars=0, tmps=0)
    ; (before optimizer)
    ; /home/vagrant/code/benchmark/swoole/test3.php:19-21
L0 (20):    RETURN int(1)
L1 (21):    RETURN null

f1: ; (lines=5, args=0, vars=0, tmps=2)
    ; (before optimizer)
    ; /home/vagrant/code/benchmark/swoole/test3.php:15-17
L0 (16):    INIT_FCALL_BY_NAME 0 string("f2")
L1 (16):    V0 = DO_FCALL_BY_NAME
L2 (16):    T1 = ADD int(1) V0
L3 (16):    RETURN T1
L4 (17):    RETURN null
```

このOpCodeは以下のようなステップで実行されます。

1. メインルーチン（ `$_main` ）のスタックフレームをVMスタックにプッシュ
2. 実行フレームへのポインタ `EG(current_execute_data)` をメインルーチンのスタックフレームにポイント。現在のスタックフレームのoplineへのポインタ `EX(opline)` をOpArrayの開始位置にポイント。
3. `EX(opline)+1` しながらOpArray全体が実行されるまで次のOpCodeを実行
	1. メインルーチンの `INIT_FCALL` で関数f1のスタックフレームをVMスタックにプッシュ。関数f1のスタックフレームの `prev_execute_data` にメインルーチンのスタックフレームをポイント。
	2. メインルーチンの `DO_UCALL` で `EG(current_execute_data)` を関数f1のスタックフレームにポイント。 `EX(opline)` を関数f1のOpArrayの開始位置にポイント。 `EX(opline)+1` しながらOpArray全体が実行されるまで次のOpCodeを実行。
		1. 関数f1の `INIT_FCALL_BY_NAME` で関数f2のスタックフレームをVMスタックにプッシュ。関数f2のフレームの `prev_execute_data` に関数f1のスタックフレームをポイント。
		2. 関数f1の `DO_FCALL_BY_NAME` で `EG(current_execute_data)` を関数f2のスタックフレームにポイント。 `EX(opline)` を関数f2のOpArrayの開始位置にポイント。 `EX(opline)+1` しながらOpArray全体が実行されるまで次のOpCodeを実行。
		3. 全てのOpCodeを実行したら `EG(current_execute_data)` を `prev_execute_data` （関数f1のスタックフレーム）にポイント。関数f2のスタックフレームを解放し、実行位置を関数f1の `opline+1` に戻す
	3. 全てのOpCodeを実行したら `EG(current_execute_data)` を `prev_execute_data` （メインルーチンのスタックフレーム）にポイント。関数f1のスタックフレームを解放し、実行位置をメインルーチンの `opline+1` に戻す
4. メインルーチンのスタックフレームを解放し、実行が終了

[![実行ステップ](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/e4e7d5cb-f483-bd39-0aec-f023d1a062bb.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2Fe4e7d5cb-f483-bd39-0aec-f023d1a062bb.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=60713f54b4ebd809f36d8b73abba650d)

#### Swooleのスタック管理

PHPのスタック管理の概要を説明しましたので、改めて最初のコルーチンのサンプルプログラムを見てみましょう。

```php
<?php
$cid = go(function () {
    echo "coro 1 start\n";
    Co::yield();
    echo "coro 1 end\n";
});
echo "main 1\n";
go(function ($cid) {
    echo "coro 2 start\n";
    Co::resume($cid);
    echo "coro 2 end\n";
}, $cid);
echo "main 2\n";

// 実行結果
coro 1 start
main 1
coro 2 start
coro 1 end
coro 2 end
main 2
```

`yield` の実行で処理を中断し、 `resume` の実行で処理を再開していました。先ほど説明したVMスタックの実行フローを踏まえると、コルーチンの切り替えは主にスタックフレームの保存と復元で実現できることが推測できます。では、どのような仕組みになっているのか確認してみましょう。

コルーチンの作成は `go` 関数を介して行います。go関数を実行するとコルーチン用のVMスタックが作成されます。ここでは先ほどのVMスタックと区別するために便宜的にコルーチンスタックと呼ぶことにします。各コルーチンがそれぞれ自身のスタックを持ち、スタックのデフォルトサイズは8KBです。

コルーチンスタックにはコルーチン切り替え時のスタックおよびスタックフレームの保存・復元に使用するデータ格納領域があり、 `php_coro_task` という構造体で定義されています。

[![コルーチンスタック](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/8f8d23f7-04a7-a65d-1ffa-7b3242233022.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F8f8d23f7-04a7-a65d-1ffa-7b3242233022.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=44b35cb9848a2a75ad54b9db3e79a73f)

構造体に定義されている `vm_stack_` で始まる項目はグローバルスタックへのポインタを指します。関数実行の際にスタックフレームの作成・解放が行われることは説明しましたが、これはグローバルスタックに対して行われます。よって、スタックの切り替えを正常に行うためにはこの情報を持っておく必要があります。 `execute_data` はスタックフレームを復元する上で必要となる情報です。

go関数でコルーチンスタックが作成されると、グローバルスタックがコルーチンスタックに切り替わります。そして、go関数の引数で渡したクロージャからスタックフレームが作成され、コルーチンスタックにプッシュされます。

スタックフレームがプッシュされるとコルーチンスタックの実行、つまりクロージャの実行が開始されます。クロージャの実行が終了するとコルーチンスタックが解放され、 `php_coro_task` からスタックおよびスタックフレームの復元が行われます。なお、メインルーチンとコルーチンの切り替えではコルーチンスタックのphp\_coro\_taskではなく、静的変数 [^7] に割り当てられたphp\_coro\_taskが使用されます。下の図の `main task` が静的変数に該当し、これを使用してグローバルスタックをコルーチンスタックからVMスタックに切り替え、go関数の次へ処理を進めます。

[![コルーチンスタックからメインルーチンへ](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/424da2f0-a5ad-132e-b52a-80d945c70778.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F424da2f0-a5ad-132e-b52a-80d945c70778.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ee96941ee9ac96a5594658858e3d546a)

次にサンプルプログラムのようにコルーチンで `yield` と `resume` が実行された場合のスタックの変化を確認してみましょう。yieldが実行されるとphp\_coro\_taskは現在のコルーチンスタックおよびexecute\_dataにポイントされ、main taskを使ってコルーチンスタックからVMスタックに切り替わります。そして、resumeが実行されると引数で渡されたコルーチンIDに該当するコルーチンのphp\_coro\_taskを使ってコルーチンの切り替えを行います。yieldで中断した処理がresumeで再開し、クロージャの実行が終了するとコルーチンスタックが解放され、main taskではなくresumeを呼び出したコルーチンの `php_coro_task` からスタックおよびスタックフレームの復元が行われます。

下の図はyieldを実行した状態です。php\_coro\_taskに保存した後にmain taskを使ってメインルーチンへの切り替えを行います。

[![yield](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/aa1fcb72-7597-2238-535d-06552c25bbc8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2Faa1fcb72-7597-2238-535d-06552c25bbc8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d58582237ffd19b1f5cbee35b86639e2)

こちらの図はresumeを実行した状態です。コルーチン１のphp\_coro\_taskを使ってコルーチン２からコルーチン１への切り替えを行い、コルーチン１の処理が終了するとコルーチン２のphp\_coro\_taskを使ってコルーチン１からコルーチン２へ切り替えを行います。コルーチン２の処理が終了するとmain taskを使ってメインルーチンへの切り替えを行います。

[![resume](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/4797839b-8d09-daa3-631e-92f2d0f0d6c1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F4797839b-8d09-daa3-631e-92f2d0f0d6c1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=c3ff6accca6c102ea28f170285509b01)

以上がSwooleのスタック管理の概要になります。少しだけ補足すると、SwooleではCスタックとPHPスタックの両方を管理しています。これは `array_walk` や `array_map` 、 `ReflectionFunction::invoke` などがZend API（C関数） [^8] から直接ユーザ関数を呼び出しており、ユーザ関数の中でコルーチンの切り替えを行うとスタックフレームが解放されてしまい復元できなくなるからです。そのため、Swoole4.xでコルーチンの実装が見直されデュアルスタック管理に変更されています。

[![デュアルスタック管理](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/184074/1def95ff-7960-6752-d16b-72e7b1ab563e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F184074%2F1def95ff-7960-6752-d16b-72e7b1ab563e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6f9f996dc3aa2dc43e8b21a0a02b1982)

#### コルーチンのスケジューリング

ここまでは `yield` と `resume` で明示的にコルーチンを切り替える例を使って説明してきましたが、コルーチンの切り替えはI/Oイベントをトリガーにして暗黙的に行うこともできます。

```php
Co\run(function() {
    $cid = go(function() {
        echo "start coroutine 1\n";
        Co::yield();
        echo "end coroutine 1\n";
    });
    go(function() { 
        $db = new Swoole\Coroutine\Mysql;
        $server = [
            'host'     => '127.0.0.1',
            'user'     => 'user',
            'password' => 'pass',
            'database' => 'test'
        ];
        echo "before connect\n";
        $db->connect($server);
        echo "after connect\n";
        echo "before prepare\n";
        $stmt = $db->prepare('SELECT sleep(2)');
        echo "after prepare\n";
        echo "before execute\n";
        $stmt->execute();
        echo "after execute\n";
    });
    go(function($cid) {
        echo "start coroutine 2\n";
        Co::resume($cid);
        echo "end coroutine 2\n";
    }, $cid);
    go(function() {
        echo "before sleep\n";
        Co::sleep(1);
        echo "after sleep\n";
    });
});

// 実行結果
start coroutine 1
before connect
start coroutine 2
end coroutine 1
end coroutine 2
before sleep
after connect
before prepare
after prepare
before execute
after sleep
after execute
```

上記の例を見るとDBへの接続やクエリの実行などI/Oによる待ちが発生している箇所でコルーチンが切り替わっていることが分かります。上記はDB接続でしたが、この他にもHTTP通信やファイルの読み書きといったI/Oでも同様に切り替えることができます。そして、これらのI/Oの監視を行うのがSwooleサーバの章で触れた **Reactor** です。

ReactorはI/Oの監視にOSのイベント通知機能（ [epoll](https://linuxjm.osdn.jp/html/LDP_man-pages/man2/epoll_ctl.2.html) / [poll](https://linuxjm.osdn.jp/html/LDP_man-pages/man2/poll.2.html) / [select](https://linuxjm.osdn.jp/html/LDP_man-pages/man2/select.2.html) / [kqueue](https://man.openbsd.org/kqueue) のいずれか）を使用します。このイベント通知はファイルディスクリプタを介して行われます。

ファイルディスクリプタとは [Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E8%A8%98%E8%BF%B0%E5%AD%90) によると以下のように説明されています。

> 一般にファイル記述子は、オープン中ファイルの詳細を記録しているカーネル内データ構造（配列）へのインデックスである。POSIXでは、これをファイル記述子テーブルと呼び、各プロセスが自身のファイル記述子テーブルを持つ。ユーザーアプリケーションは抽象キー（＝ファイル記述子）をシステムコール経由でカーネルに渡し、カーネルはそのキーに対応するファイルにアクセスする。アプリケーション自身はファイル記述子テーブルを直接読み書きできない。  
> UNIX系システムでは、ファイル記述子がファイルだけでなく、ディレクトリ、ブロックデバイスやキャラクターデバイス（スペシャルファイルとも呼ぶ）、ソケット、FIFO（名前付きパイプ）、名前なしパイプなどのカーネルオブジェクトを汎用的に参照するのに使われる。

例えば [stream\_socket\_server](https://www.php.net/manual/ja/function.stream-socket-server.php) といったストリーム関数の返り値は `resource` 型（実体は `php_stream` ）ですが、その中にはソケットのファイルディスクリプタも情報として含まれており [^9] 、それを使ってデータの送受信を行います [^10] 。 [fopen](https://www.php.net/manual/ja/function.fopen.php) のようなファイルシステム関数も同様にファイルディスクリプタを使用してファイルの読み書きを行います。

Reactorにはファイルディスクリプタの種類 [^11] ごとにイベントハンドラが登録されており、イベントの通知を受けたら該当のイベントハンドラを実行します。例えば、先程のサンプルプログラムの `$db->connect($server)` の箇所は以下のようなステップで実行されます。

1. コルーチン作成時のReactor初期化において、ファイルディスクリプタ種別 `SW_FD_CORO_SOCKET` （コルーチン内ソケット通信）の `SW_EVENT_READ` （読み込みイベント）および `SW_EVENT_WRITE` （書き込みイベント）に対するイベントハンドラを登録する
2. ノンブロッキングモードでソケットを生成し、ソケット接続（connect）を実行する
3. ソケット接続実行時にReactorにソケットのファイルディスクリプタを `SW_FD_CORO_SOCKET` および `SW_EVENT_WRITE` でイベント登録する（ [epoll\_ctl](https://linuxjm.osdn.jp/html/LDP_man-pages/man2/epoll_ctl.2.html) でファイルディスクリプタと `EPOLLOUT` イベントを関連付ける）
4. コルーチンの `yield` を実行して現在のコルーチンを中断し、コルーチンを切り替える
5. [epoll\_wait](https://linuxjm.osdn.jp/html/LDP_man-pages/man2/epoll_wait.2.html) で `EPOLLOUT` イベントを待ち、イベントを受け取ったらイベントハンドラを実行する
6. イベントハンドラでコルーチンの `resume` を実行し、コルーチンを再開する

このような切り替えがネットワークI/Oが発生するたびに暗黙的に実行されます。なお、前出のサンプルでは `Co\run` を介して処理を実行しています。このCo\\runについて少し触れておきます。Co\\runの実体は `Swoole\Coroutine\Scheduler` です。

```php
namespace Swoole\Coroutine {
    function run(callable $fn, ...$args)
    {
        $s = new Scheduler();
        $s->add($fn, ...$args);
        return $s->start();
    }
}
```

SchedulerはI/Oイベントやタイマーイベントの監視を開始する役割を果たします。例えば下記の１番目と２番目のコードはほぼ同義になります（Swoole公式では１番目の書き方を推奨）。

```php
Co\run(function() {
    go(function() {
        Co::sleep(1);
        echo "Done 1\n";
    });
    go(function() {
        Co::sleep(1);
        echo "Done 2\n";
    });
});
```

```php
go(function() {
    go(function() {
        Co::sleep(1);
        echo "Done 1\n";
    });
    go(function() {
        Co::sleep(1);
        echo "Done 2\n";
    });
});
swoole_event_wait();
```

`swoole_event_wait` の部分がイベントループでイベントの監視を行っています。ただ、swoole\_event\_waitがなくても上記のコードは動作します。これはコルーチン作成時に同様の処理を [シャットダウン関数](https://www.php.net/manual/ja/function.register-shutdown-function.php) として登録しているためです。ただし、シャットダウン関数はPHPライフサイクルの `php_request_shutdown` で実行されるため、例えば以下のようなコードだと `Co::sleep` が動作しません。

```php
$serverSocket = @stream_socket_server('tcp://127.0.0.1:8080', $errno, $errstr, STREAM_SERVER_BIND | STREAM_SERVER_LISTEN);
if (! $serverSocket) {
    exit;
}
echo "Starting server...\n";

while ($clientSocket = @stream_socket_accept($serverSocket)) {
    go(function() {
        go(function() {
            Co::sleep(1);
            echo "Done 1\n";
        });
        go(function() {
            Co::sleep(1);
            echo "Done 2\n";
        });
    });
}
```

公式サイトに掲載されているサンプルコードはコルーチンの書き方が統一されておらず、どの書き方が正しいのか分かりにくいのですが、Swooleサーバ外でコルーチンを使うときはCo\\runを併せて使うようにすると良いかと思います。

**タイムアウトの設定**

外部APIの実行やSQLの実行で処理に時間がかかってレスポンスが長時間返ってこないケースは多々あります。そのような場合、タイムアウトを設定することで一定時間経過したら監視をキャンセルしてコルーチンの処理を再開することが可能です。タイムアウトを設定するには `socket_timeout` 、 `socket_reade_timeout` 、 `socket_write_timeout` のいずれかのオプションを指定します。

```php
// タイムアウトを1秒に設定
Co::set([
    'socket_timeout' => 1
]);

Co\run(function() {
    ....

    $db->connect($server);
    $stmt = $db->prepare('SELECT sleep(2)');
    $ret = $stmt->execute(); // タイムアウト時のretはfalse

    // executeでタイムアウトしたらここから処理を再開する
    doSomething();
});
```

**プリエンプティブスケジューリング**

以下のようなプログラムを実行すると `Co::sleep` の後にexitフラグを更新してループを抜けるように見えますが、実際は無限ループになってしまいます。これはwhileのループ処理にCPUを占有されてしまい、Reactorのイベントループが処理できなくなるためです（Reactorがsleepのタイマー処理を行う）。

```php
Co\run(function() {
    $exit = false;
    while (true){
        $stats = Co::stats();
        $num = $stats['coroutine_num'];
        if ($num < 3){
            go(function () use(&$exit){
                echo "cid:" . Co::getCid() . " start\n";
                Co::sleep(1);
                echo "cid " . Co::getCid() . " end\n";
                $exit = true;
            });
        }
        if ($exit) {
            break;
        }
    }
    echo "main end\n";
});

// 実行結果
cid:2 start
cid:3 start
```

このように１つのコルーチンがCPUを占有して、他のコルーチンがCPUタイムスライスを取得できない状態になるのを回避するのがプリエンプティブスケジューリング機能です。この機能を有効にするには `enable_preemptive_scheduler` プションを指定するか、php.iniに `swoole.enable_preemptive_scheduler=1` を追加します。

```php
Co::set([
    'enable_preemptive_scheduler' => true
]);

Co\run(function() {
    ....
});

// 実行結果
cid:2 start
cid:3 start
cid 2 end
cid 3 end
main end
```

どのようにしてWhileループからコルーチンに処理を切り替えているのか簡単に触れておきますと、ZendVMの割り込みハンドラ（ `zend_interrupt_function` ）を利用しています。割り込みハンドラは `EG(vm_interrupt)` が 1 の時に１度だけ実行される関数でOpCodeが実行されるタイミングで割り込み処理を行います。つまりwhileループ内のOpCodeが実行されるタイミングで割り込みハンドラに指定した関数が実行されるため、無限ループであっても割り込み処理を行うことができます。

割り込み処理の対象はコルーチン内の処理を開始もしくは最後に再開してから `10ms` を超過したコルーチンで、現在ストップしている次の箇所から処理を再開するようになっています（上記の例だと `Co::sleep` の後から処理を再開してexitフラグを更新するためループを抜けることができる）。

10ms超過という条件のため以下のようなケースだとプリエンプティブスケジューリングの有無によって結果が変わってきます。使用する場合は十分に注意してください。

```php
Co\run(function() {
    go(function () {
        echo "cid:" . Co::getCid() . " start\n";
        usleep(10000);
        echo "cid " . Co::getCid() . " end\n";
    });
    
    echo "main end\n";
});

// 実行結果（enable_preemptive_scheduler === false）
cid:2 start
cid 2 end
main end

// 実行結果（enable_preemptive_scheduler === true）
cid:2 start
main end
cid 2 end
```

#### PHPビルトイン関数のノンブロッキング化

例えばコルーチン内でPDOを使ってDBにアクセスすると、I/O操作が完了するまで待機状態となってしまいます。

```php
Co\run(function() {
    go(function () {
        echo "cid:" . Co::getCid() . " start\n";
        $pdo = new PDO('mysql:dbname=test;host=localhost;charset=utf8mb4', 'user', 'pass');
        $stmt = $pdo->prepare('select sleep(2)');
        $stmt->execute();
        echo "cid " . Co::getCid() . " end\n";
    });
    
    echo "main end\n";
});

// 実行結果
cid:2 start
cid 2 end
main end
```

正常に動作はするのですが、これではせっかくのコルーチンの並行性が失われてしまいます。そのため、Swooleではコルーチンに対応したMySQLクライアントやHTTPクライアント、ファイルシステムAPIなどが用意されています。しかし、OSSのライブラリの中でPDOを使っていてどうしても変更できないというケースも多々あります。そのような問題を解決するために、Swooleではファイルやソケットに関する関数をフックしてコルーチン用の処理に差し替え、ノンブロッキング化する仕組みを持っています。

例えば、先程のPDOをコルーチンに対応させるには以下のように `Swoole\Runtime::enableCoroutine` を追加します。

```php
Swoole\Runtime::enableCoroutine(); // Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_UNIX)でも可
Co\run(function() {
    go(function () {
        echo "cid:" . Co::getCid() . " start\n";
        $pdo = new PDO('mysql:dbname=test;host=localhost;charset=utf8mb4', 'user', 'pass');
        $stmt = $pdo->prepare('select sleep(2)');
        $stmt->execute();
        echo "cid " . Co::getCid() . " end\n";
    });
    
    echo "main end\n";
});

// 実行結果
cid:2 start
main end
cid 2 end
```

Swoole\\Runtime::enableCoroutineは以下の引数を指定できます。

```php
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_TCP); // stream（TCPソケット）
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_UDP); // stream（UDPソケット）
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_UNIX); // stream（UNIXストリームソケット）
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_UDG); // stream（UNIXドメインソケット）
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_SSL); // stream（SSLソケット）
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_TLS); // stream（TLSソケット）
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_STREAM_FUNCTION); // stream_select,stream_socket_pair
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_FILE); // ファイルシステム
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_SLEEP); // sleep,usleep,time_nanosleep,time_sleep_until
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_PROC); // proc_open,proc_close,proc_get_status,proc_terminate
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_CURL); // curl_init,curl_setopt,curl_setopt_array,curl_exec,curl_getinfo,curl_errno,curl_error,curl_reset,curl_close,curl_multi_getcontent
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_BLOCKING_FUNCTION); // gethostbyname,exec,shell_exec
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL); // SWOOLE_HOOK_CURLを除く全て
Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL | SWOOLE_HOOK_CURL); // 全て
Swoole\Runtime::enableCoroutine(); // SWOOLE_HOOK_ALL
Swoole\Runtime::enableCoroutine(false); // 全てのフックを無効化
```

上記のサンプルではPDOを使用しましたが、 **ノンブロッキングに対応しているのはMySQLのみ** です。PostgreSQLを使用する場合は [Swoole\\Coroutine\\Postgres](https://github.com/swoole/ext-postgresql) を使用する必要があります。

## まとめ

今回は基礎の基礎ということで仕組みの部分に重点を置いてご紹介させていただきました。Swooleの使い方やテクニックの紹介ではなかったので、Swooleのメリットがあまり伝わらなかったかもしれません。しかし、SwooleをベースにしたWebアプリケーションフレームワークを使ってみると、Swooleを使うメリットを大いに感じることができると思います。

しかし、一方でSwooleには普段のPHP開発にはない注意すべきポイントも色々あります。例えば、普段よく使うOSSのライブラリやベンダー提供のモジュール（例：決済接続モジュール）で実はブロッキングI/Oが発生していたということはよくあります。１つのスレッド内で動くコルーチンは１つだけで、複数のコルーチンが並行で動くことはありません。そのため、コルーチン内でのブロッキングI/OはWebアプリケーションのパフォーマンスに大きな影響を与える可能性があります。冒頭のパフォーマンス検証のプログラムをブロッキングI/OとノンブロッキングI/Oで比較するとスループットに約４倍も差がでました。

Swooleの導入には事前の入念な検証および技術要件の整理が必要となるでしょう。本記事が少しでもお役に立てれば幸いです。

[0](https://qiita.com/prograti/items/#comments)

[87](https://qiita.com/prograti/items/e29210300232e4a3d8b7/likers)

67

ストックを更新しました

[^1]: [http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf](http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf)

[^2]: [https://github.com/php/php-src/blob/960318ed95d17bd30c2896e2f3189ebffb965dce/Zend/zend\_execute\_API.c#L253](https://github.com/php/php-src/blob/960318ed95d17bd30c2896e2f3189ebffb965dce/Zend/zend_execute_API.c#L253)

[^3]: [https://github.com/php/php-src/blob/19e886d9d823a84e55a462c344e75e2e0707d294/Zend/zend\_compile.h#L136](https://github.com/php/php-src/blob/19e886d9d823a84e55a462c344e75e2e0707d294/Zend/zend_compile.h#L136)

[^4]: [https://github.com/php/php-src/blob/19e886d9d823a84e55a462c344e75e2e0707d294/Zend/zend\_compile.h#L395](https://github.com/php/php-src/blob/19e886d9d823a84e55a462c344e75e2e0707d294/Zend/zend_compile.h#L395)

[^5]: [https://github.com/php/php-src/blob/ac0853eb265784c4238af652de9c54c883ffa99f/Zend/zend\_execute.h#L155](https://github.com/php/php-src/blob/ac0853eb265784c4238af652de9c54c883ffa99f/Zend/zend_execute.h#L155)

[^6]: [https://github.com/php/php-src/blob/36935e42ea/Zend/zend\_compile.h#L484](https://github.com/php/php-src/blob/36935e42ea/Zend/zend_compile.h#L484)

[^7]: [https://github.com/swoole/swoole-src/blob/master/swoole\_coroutine.cc#L97](https://github.com/swoole/swoole-src/blob/master/swoole_coroutine.cc#L97)

[^8]: [https://github.com/php/php-src/blob/718e55c3e045a3b786749e0fbedda7f0ab444907/Zend/zend\_execute\_API.c#L642](https://github.com/php/php-src/blob/718e55c3e045a3b786749e0fbedda7f0ab444907/Zend/zend_execute_API.c#L642)

[^9]: [https://github.com/php/php-src/blob/5d6e923d46a89fe9cd8fb6c3a6da675aa67197b4/main/php\_streams.h#L188](https://github.com/php/php-src/blob/5d6e923d46a89fe9cd8fb6c3a6da675aa67197b4/main/php_streams.h#L188)

[^10]: [https://github.com/php/php-src/blob/f078bca729/main/streams/xp\_socket.c](https://github.com/php/php-src/blob/f078bca729/main/streams/xp_socket.c)

[^11]: [https://github.com/swoole/swoole-src/blob/7e035b72d4bbd9e2696c92748d5a296c601de9a0/include/swoole.h#L338](https://github.com/swoole/swoole-src/blob/7e035b72d4bbd9e2696c92748d5a296c601de9a0/include/swoole.h#L338)