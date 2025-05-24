---
title: Laravelで200%お世話になるであろうUploadedFileクラスを見てみよう
source: https://qiita.com/mashirou_yuguchi/items/14d3614173c114c30f02
author:
  - "[[mashirou_yuguchi]]"
published: 2019-06-06
created: 2025-05-22
description: はじめにオーバーに言いました。許して。ファイルのアップロード周りで要件を満たすためにあれやこれやと悩み続けたのでメモ件忘備としてそれっぽいエントリになる予定Laravel5.5で調べたのは仕事…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から5年以上が経過しています。

[@mashirou\_yuguchi](https://qiita.com/mashirou_yuguchi)

投稿日

## はじめに

オーバーに言いました。許して。

ファイルのアップロード周りで要件を満たすためにあれやこれやと悩み続けたのでメモ件忘備としてそれっぽいエントリになる予定  
Laravel5.5で調べたのは仕事でLaravel5.5を使っているから流れです。5.7とか5.8の調査は手付かず

## UploadedFileクラスって何

9割9部9里 [公式ドキュメント](https://laravel.com/api/5.8/Illuminate/Http/UploadedFile.html) に書いてあるので、ほとんどはそこを見ればたいていのことは解決する。  
けど実際の挙動がどうなっているのかって話になると見てみないと分からないはずなので見てみようず。

僕は閃光☆HANABI団が大好きなので花火の動画をアップロードするついでに色々と見ていこうと思います。（激ウマギャグ）

検証用なので凝ったことはせず単純なメソッドにデバッグ関数を当て込んで中身を取り出していきましょう

php

```php
use Illuminate\Http\Request;

public function index(Request $request)
{
    dd($request->file('file'));
}
```

```text
UploadedFile {#325 ▼
  -test: false
  -originalName: "FileFlower.mp4"
  -mimeType: "video/mp4"
  -size: 2742587
  -error: 0
  #hashName: null
  path: "/tmp"
  filename: "phpgYvI3h"
  basename: "phpgYvI3h"
  pathname: "/tmp/phpgYvI3h"
  extension: ""
  realPath: "/tmp/phpgYvI3h"
  aTime: 2019-06-04 18:25:32
  mTime: 2019-06-04 18:25:32
  cTime: 2019-06-04 18:25:32
  inode: 131923
  size: 2742587
  perms: 0100600
  owner: 1000
  group: 33
  type: "file"
  writable: true
  readable: true
  executable: false
  file: true
  dir: false
  link: false
}
```

ぱらりと出てきました。  
色々と細かく書いてあるのですが、ここからデータを取得するにあたってこのUploadedFileには便利な機能がいくつかあります。

```php
vendor\laravel\framework\src\Illuminate\Http\UploadedFile.php
vendor\symfony\http-foundation\File\UploadedFile.php
```

主にこの2つが処理の中核になっています。  
が、実際に使用する際は `Illuminate\Http\UploadedFile` のみを呼び出すような形となります。  
`vendor\symfony\http-foundation\File\UploadedFile.php` は `Illuminate\Http\UploadedFile` の継承前クラスになるので、実際に使用すると1つのファイルから2つのファイルのクラス内メソッドが使用できるようになります。

この機能はRequestの時点で使えるようになっているのですが、

`vendor\laravel\framework\src\Illuminate\Http\Request.php` とか `vendor\symfony\http-foundation\Request.php` を追っていくと

追っかけたログです。最終的なのは以下

vendor\\laravel\\framework\\src\\Illuminate\\Http\\Request.php

```php
/**
 * Create a new Illuminate HTTP request from server variables.
 *
 * @return static
 */
public static function capture()
{
    static::enableHttpMethodParameterOverride();

    return static::createFromBase(SymfonyRequest::createFromGlobals());//createFromGlobalsを掘る
}
```

vendor\\symfony\\http-foundation\\Request.php

```php
/**
 * Creates a new request with values from PHP's super globals.
 *
 * @return static
 */
public static function createFromGlobals()
{
    // With the php's bug #66606, the php's built-in web server
    // stores the Content-Type and Content-Length header values in
    // HTTP_CONTENT_TYPE and HTTP_CONTENT_LENGTH fields.
    $server = $_SERVER;
    if ('cli-server' === \PHP_SAPI) {
        if (\array_key_exists('HTTP_CONTENT_LENGTH', $_SERVER)) {
            $server['CONTENT_LENGTH'] = $_SERVER['HTTP_CONTENT_LENGTH'];
        }
        if (\array_key_exists('HTTP_CONTENT_TYPE', $_SERVER)) {
            $server['CONTENT_TYPE'] = $_SERVER['HTTP_CONTENT_TYPE'];
        }
    }

    $request = self::createRequestFromFactory($_GET, $_POST, [], $_COOKIE, $_FILES, $server);
    //createRequestFromFactoryを掘る
    if (0 === strpos($request->headers->get('CONTENT_TYPE'), 'application/x-www-form-urlencoded')
        && \in_array(strtoupper($request->server->get('REQUEST_METHOD', 'GET')), ['PUT', 'DELETE', 'PATCH'])
    ) {
        parse_str($request->getContent(), $data);
        $request->request = new ParameterBag($data);
    }

    return $request;
}
```

vendor\\symfony\\http-foundation\\Request.php

```php
/**
 * Clones a request and overrides some of its parameters.
 *
 * @param array $query      The GET parameters
 * @param array $request    The POST parameters
 * @param array $attributes The request attributes (parameters parsed from the PATH_INFO, ...)
 * @param array $cookies    The COOKIE parameters
 * @param array $files      The FILES parameters
 * @param array $server     The SERVER parameters
 *
 * @return static
 */
public function duplicate(array $query = null, array $request = null, array $attributes = null, array $cookies = null, array $files = null, array $server = null)
{
    $dup = clone $this;
    if (null !== $query) {
        $dup->query = new ParameterBag($query);
    }
    if (null !== $request) {
        $dup->request = new ParameterBag($request);
    }
    if (null !== $attributes) {
        $dup->attributes = new ParameterBag($attributes);
    }
    if (null !== $cookies) {
        $dup->cookies = new ParameterBag($cookies);
    }
    if (null !== $files) {
        $dup->files = new FileBag($files);//FileBagをインスタンスとして起動しているのでこれを掘ってみる
    }
    if (null !== $server) {
        $dup->server = new ServerBag($server);
        $dup->headers = new HeaderBag($dup->server->getHeaders());
    }
    $dup->languages = null;
    $dup->charsets = null;
    $dup->encodings = null;
    $dup->acceptableContentTypes = null;
    $dup->pathInfo = null;
    $dup->requestUri = null;
    $dup->baseUrl = null;
    $dup->basePath = null;
    $dup->method = null;
    $dup->format = null;

    if (!$dup->get('_format') && $this->get('_format')) {
        $dup->attributes->set('_format', $this->get('_format'));
    }

    if (!$dup->getRequestFormat(null)) {
        $dup->setRequestFormat($this->getRequestFormat(null));
    }

    return $dup;
}
```

vendor\\symfony\\http-foundation\\FileBag.php

```php
/**
 * Converts uploaded files to UploadedFile instances.
 *
 * @param array|UploadedFile $file A (multi-dimensional) array of uploaded file information
 *
 * @return UploadedFile[]|UploadedFile|null A (multi-dimensional) array of UploadedFile instances
 */
protected function convertFileInformation($file)
{
    if ($file instanceof UploadedFile) {
        return $file;
    }

    $file = $this->fixPhpFilesArray($file);
    if (\is_array($file)) {
        $keys = array_keys($file);
        sort($keys);

        if ($keys == self::$fileKeys) {
            if (UPLOAD_ERR_NO_FILE == $file['error']) {
                $file = null;
            } else {
                $file = new UploadedFile($file['tmp_name'], $file['name'], $file['type'], $file['size'],$file['error']);
                //ここでUploadedFileインスタンスを起動している
            }
        } else {
            $file = array_map([$this, 'convertFileInformation'], $file);
            if (array_keys($keys) === $keys) {
                $file = array_filter($file);
            }
        }
    }

    return $file;
}
```

このあたりでUploadedFileにつながるようになっています。  
ただしこのファイルをあれこれ操作していかないと行けない時は別処理においたりなどする必要があると思います。  
（FileServiceとかFileUseCase的なやつからファイルの操作を行うなど）  
その際はUploadedFileを呼び出して操作できるようにしようという話ですね。

## 実際の動作

では動作面を見ていきましょう。  
ファイルに直接の操作を行っているのは `vendor\symfony\http-foundation\Request.php` で、  
ファイルに動的な操作を行っているのは `vendor\laravel\framework\src\Illuminate\Http\Request.php` となります。

vendor\\symfony\\http-foundation\\Request.phpにまつわるもの

```php
$request->file('file')->store($path);
$request->file('file')->storePublicly($path);
$request->file('file')->storePubliclyAs($path);
$request->file('file')->storeAs($path,$name);
```

vendor\\laravel\\framework\\src\\Illuminate\\Http\\Request.phpにまつわるもの

```php
$request->file('file')->getPathname();
$request->file('file')->getClientOriginalName();
$request->file('file')->getClientMimeType();
$request->file('file')->guessClientExtension();
$request->file('file')->getClientSize();
$request->file('file')->getError();
$request->file('file')->isValid();
$request->file('file')->move($directory);
$request->file('file')->getMaxFilesize();
$request->file('file')->getErrorMessage();
```

だいたいこの辺りになったかと思います。

この辺についてですが、日本語訳版の [Laravelドキュメント](https://readouble.com/laravel/5.8/ja/filesystem.html) さんだとほぼ乗っていません。  
`store` や `storeAs` は掲載されているので、最低限の実装には影響はありませんが、少し凝ったことをしようとすると中々調べるのに時間がかかってしまうと思います。

実際、僕も調べた当初はgetClientOriginalName()を見つけるなどに1日かけてしまうなどでした。

では中身の挙動について説明していきます。

### $request->file('file')->store($path);

結果的に

```php
$this->storeAs($path, $this->hashName(), $this->parseOptions($options));
```

このように返すのですが、注目点は `$this->hashName()` です。  
このhashNameはUploadedFile内に入っている `#hashName: null` がここに該当します。  
明示的に `$request->file('movie')->hashName()` としておくと

```text
"vUzcs4SlLQ3MN2QzGK4EFfEruoG0r5glmWFlxA74.mp4"
```

このようにhash化されたファイルに置き換えられます。  
こういったデータの使い方としては、明示的にファイルをマスクする必要がある場合（特に画像を外部に公開するなど）  
このようにどういったデータだったかを明文化しないようにすることで秘匿性が増すことができます。  
もしくはダブリデータの回避等という考えもあります。  
どちらも一意性を保つことを目的にされているという感じです。

### $request->file('file')->storePublicly($path);

ぱっと見て `store` と大差ないのですが、 **Publicly** という単語でpublicなデータになるのかなぁみたいな感じになると思います。  
ソースの説明を見る限りでも  
本文 `Store the uploaded file on a filesystem disk with public visibility`  
google先生訳:`一般公開されているファイルシステムのディスクにアップロードしたファイルを保存します`  
ということで、公開用のファイルとしてアップロードするという処理になります。  
一見便利な風に見えて、なぜ説明として使われないのかというと、  
`$request->file('file')->store($path);`は  
`$request->file('file')->store($path, 'public');とすることができるからです。`  
つまり、storeでできることを明示化しただけのものと受け取ることもできるわけです。

### $request->file('file')->storeAs($path,$name);

このstoreAsこそ、laravelのStorage妖精、あるいはStorage神となる中核の処理です。  
行っている処理というのが `store` の一意性のあるファイルに置き換えず、自分で好きな命名に置き換えることのできるすぐれものになります。  
`store` の処理を見てみるとわかりますが、内部的にはstoreAsを使っているのがわかります。

vendor\\laravel\\framework\\src\\Illuminate\\Http\\UploadedFile.php

```php
/**
 * Store the uploaded file on a filesystem disk.
 *
 * @param  string  $path
 * @param  string  $name
 * @param  array|string  $options
 * @return string|false
 */
public function storeAs($path, $name, $options = [])
{
    $options = $this->parseOptions($options);

    $disk = Arr::pull($options, 'disk');

    return Container::getInstance()->make(FilesystemFactory::class)->disk($disk)->putFileAs(
        $path, $this, $name, $options
    );
}
```

このように、storeAsではputFileAsを使って出力処理を行っていることがわかります。  
putFileAsは `vendor\laravel\framework\src\Illuminate\Filesystem\FilesystemAdapter.php` で処理の説明がありますが、その辺りは話がそれてしまうので割愛します。  
どうしても知りたい場合は [こちら](https://laravel.com/api/5.5/Illuminate/Filesystem/FilesystemAdapter.html) からお調べくださいませ。

### $request->file('file')->storePubliclyAs($path);

これも実態は `$request->file('file')->storePublicly($path);`と同じで、明示的にpublicとして一般公開可能なファイルとして展開するように出力する処理になります。

### $request->file('file')->getPathname();

字のごとく、パス名を取得するとありますが、厳密には **tmpの仮ファイル名を取得する** になります。  
UploadedFileで取得したものを見てみると、pathnameと明記されたものがあり、そちらを取得するようになります。

```php
pathname: "/tmp/phpUnSKpP"
```

こちら、自分自身もあまり意識せずに使用していたのですが、クソ真面目に調べた結果、Laravelの機能ではなくPHPの関数であることがわかりました。  
[https://www.php.net/manual/ja/directoryiterator.getpathname.php](https://www.php.net/manual/ja/directoryiterator.getpathname.php)  
あまりにも自然に使われていたのでLaravel便利やなーとか思ってたんですけど全然違いましたね。

なおこちらの活用方法は未だ見いだせていません。  
正式にputするときに命名として吐き出すとかそのくらいだけど……これでも一応ハッシュっぽいファイル名だし。（.mp4どこいったんだろ）

### $request->file('file')->getClientOriginalName();

オリジナル名を取得するということですが、こちらもUploadedFileからオリジナルの名前を取ることになります。

```php
-originalName: "FileFlower.mp4"
```

内部処理としては `vendor\symfony\http-foundation\File\UploadedFile.php` で行われていて、かつコンストラクタで結果を取りに行っているようです。

vendor\\symfony\\http-foundation\\File\\UploadedFile.php

```php
$this->originalName = $this->getName($originalName);
```

### $request->file('file')->getClientMimeType();

mimetypeを取得するので、今回の場合 `video/mp4` が帰ってくる想定です。

```php
"video/mp4"
```

内部処理としては `vendor\symfony\http-foundation\File\UploadedFile.php` で行われていて、かつコンストラクタで結果を取りに行っているようです。

vendor\\symfony\\http-foundation\\File\\UploadedFile.php

```php
$this->mimeType = $mimeType ?: 'application/octet-stream';
```

### $request->file('file')->guessClientExtension();

ファイルの端子を調べてくれるすごいやつ。

```php
"mp4"
```

内部処理としては `vendor\symfony\http-foundation\File\UploadedFile.php` で行われています。

vendor\\symfony\\http-foundation\\File\\UploadedFile.php

```php
public function guessClientExtension()
{
    $type = $this->getClientMimeType();
    $guesser = ExtensionGuesser::getInstance();

    return $guesser->guess($type);
}
```

`getClientOriginalExtension();`という関数があり、同じような結果を取得できるのですが、  
[Laravel4.2](https://laravel.com/docs/4.2/requests#files) と [Laravel5.5](https://laravel.com/docs/5.5/requests#files) では情報が変わっています。

laravel5.5

```php
$request->file('file')->extension();
```

どこからextensionで取りに行けんだよという感じですが、これで取れるようです。  
こちらの挙動についてはいつか書きます。

### $request->file('file')->getClientSize();

ファイルのサイズを取得する多分いいやつ。

内部処理としては `vendor\symfony\http-foundation\File\UploadedFile.php` で行われていて、かつコンストラクタで結果を取りに行っているようです。

vendor\\symfony\\http-foundation\\File\\UploadedFile.php

```php
$this->size = $size;
```

実際用途としてはファイルサイズの制御で使うとは思いますが、byteで取得するので数字をしっかりと明示的にしておかないとダメですね。  
100MBの場合は102400Byteとかになります。

### $request->file('file')->getError();

アップロードのエラー周りの処理になります。  
とくに使うことはないですが、内部実装的には結構使ってる。

### $request->file('file')->isValid();

アップロードができたかどうかの成否判定を行っています。  
これも特に実装面で使うことはないです。逆に使い方あったら教えてください。

### $request->file('file')->move($directory);

ファイルの移動をするための処理です。

vendor\\symfony\\http-foundation\\File\\UploadedFile.php

```php
public function move($directory, $name = null)
{
    if ($this->isValid()) {
        if ($this->test) {
           return parent::move($directory, $name);
        }

        $target = $this->getTargetFile($directory, $name);

        set_error_handler(function ($type, $msg) use (&$error) { $error = $msg; });
        $moved = move_uploaded_file($this->getPathname(), $target);
        restore_error_handler();
        if (!$moved) {
            throw new FileException(sprintf('Could not move the file "%s" to "%s" (%s)', $this->getPathname(), $target, strip_tags($error)));
        }

        @chmod($target, 0666 & ~umask());

        return $target;
    }

    throw new FileException($this->getErrorMessage());
}
```

ファイルの移動を行うなどで便利かなって思ったんですが、Storage::moveがあるからそっちでいいだろってなりました。

### $request->file('file')->getMaxFilesize();

ファイルの最大アップロード数を取得してくれる結構いいやつ？？

内部処理としては `vendor\symfony\http-foundation\File\UploadedFile.php` で行われています。

php.iniから `upload_max_filesize` を取得するみたいなことが実装されているけど、これ実際それ以上のファイルデータを投入されたら元も子もないのでは？  
僕的には使い所を見出したいけど脳みそが追っついてないやつ。

### $request->file('file')->getErrorMessage();

エラー結果を表示するやつ

```php
"The file "FileFlower.mp4" was not uploaded due to an unknown error."
```

これも実際つかうことは無いと思います。  
だいたい他の処理を行っているときに失敗したりして、その時にエラーでここを挟むような感じになるのかなっておもいました。（小並感）

## 感想

日本語ドキュメントも公式ドキュメントも漁ってみたんですが、本当にこの辺の処理って書いてないんですかね？（調べ方が下手だったパターン）

途中までは実際に使ったことのある関数が多かったのでたのしかったのですが、途中から使わない関数のところに入っていくと本当に使わねえなこれってなることが多々……。  
書き方が大幅に変わっていたりそもそも直接使わずとも呼び出せるようになっている処理が混ざっていてほーとかはーとか唸ってました。

久々に実装する内容から逆読みして処理の流れを掘りましたが、Laravelってややこしいことをほとんどせずに直線で書いている感じがしてまた好感が上がりました。

多分まだ見きれてない&他のFacadeから挟んでいるものをありそうなのでその辺りも今後調べていきたいですね。  
~~5.8の変わり具合を調べたくない~~

[0](https://qiita.com/mashirou_yuguchi/items/#comments)

[97](https://qiita.com/mashirou_yuguchi/items/14d3614173c114c30f02/likers)

78

ストックを更新しました