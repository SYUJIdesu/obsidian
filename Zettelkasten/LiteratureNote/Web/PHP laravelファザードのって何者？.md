---
title: PHP laravel ファサードの前のって何者？
source: https://qiita.com/miriwo/items/d8694a8d380e6a933891
author:
  - "[[miriwo]]"
published: 2024-07-09
created: 2025-05-22
description: 概要laravelでファサードを使う時にuse宣言せずともをつけることで直接ファサードを呼び出す事ができる。これがどういうことなのか簡単にまとめる。内容<?phpuse Illumina…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

## 概要

laravelでファサードを使う時にuse宣言せずとも `\` をつけることで直接ファサードを呼び出す事ができる。  
これがどういうことなのか簡単にまとめる。

## 内容

```php
<?php

use Illuminate\Support\Facades\Validator;

Validator::resolver();
```

と

```php
<?php

\Validator::resolver();
```

は全く同じ動作をする。

`\Validator::`って記載するとuse宣言で `use Illuminate\Support\Facades\Validator;`って書かなくてもそのまま使える。  
ファサードは使いやすいように\\を用いることで省略呼び出しする事ができる。

ちなみにこんな書き方もできる。

```php
<?php

use \Validator;

Validator::resolver();
```

今まで書いた全て同じ動作をする。  
気にするならチーム内でどの書き方をするのかを決めておいたほうが良さそう。

基本無いと思うけどlaravelのバージョンアップでファサードのNamespaceが変わる可能性もあるので下記の書き方が自分は好み

```php
<?php

use \Validator;

Validator::resolver();
```

[1](https://qiita.com/miriwo/items/#comments)

コメント一覧へ移動

[0](https://qiita.com/miriwo/items/d8694a8d380e6a933891/likers)

いいねしたユーザー一覧へ移動

3