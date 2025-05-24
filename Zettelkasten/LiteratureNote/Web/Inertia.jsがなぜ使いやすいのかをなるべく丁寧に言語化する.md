---
title: Inertia.jsがなぜ使いやすいのかをなるべく丁寧に言語化する
source: https://qiita.com/itokoooooh/items/142a1d6f0aba7c5d798e
author:
  - "[[itokoooooh]]"
published: 2024-12-11
created: 2025-05-22
description: Inertia.jsとはInertia.jsはLaravelやRailsなどのバックエンドフレームワーク（フルスタックフレームワークと称した方がいいかもしれません）と、ReactやVueのようなフ…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

## Inertia.jsとは

[Inertia.js](https://inertiajs.com/) はLaravelやRailsなどのバックエンドフレームワーク（フルスタックフレームワークと称した方がいいかもしれません）と、ReactやVueのようなフロントエンドフレームワークの中間の橋渡しのような役割を持つミドルウェアです。

当初 Laravel×React with Inertia 前提でプロダクトを作成してみたのですが、  
扱うのはそんなに難しくなく、やり方さえ覚えればとても簡単にSPAアプリケーションを作成することができます。

## コレ、ベンリ、スキ・・・

しかし、個人的にLaravel×Reactの構成自体が初めてで、その中でいきなりInertiaも一緒に触ってしまったので、便利なことはわかるけど具体的に何が嬉しいかを言えない悲しきモンスターになってしまいました。

## このままではいけない

ただただ目の前にある便利なものを使っていくだけでは、本当にその技術がいいのか悪いのかを判断することはできません。

ただのモンスターではない、技術選定のできるモンスターを目指していきたい手前、今回はInertia.jsが具体的にどういう部分で開発に恩恵をもたらしてくれるのか、これまでのスタイルと何が異なるのかというところを言語化することを目指していきます。

### 本記事で扱わないこと

**Inertia.jsの中の詳しい機構**  
Inertia.js自体の中身の機構もとても気にはなっているのですが、そこまで話しだすと記事が大きくなりすぎるので、 これは別の記事でやる予定です。

**Laravel×React以外の組み合わせ**  
この構成以外でInertiaを利用したことがないため、今回はバックエンドでLaravel、フロントエンドでReactを利用するという前提に立って話します。

## Inertiaを使わない場合

Inertiaの何が便利であるかを確かめるために、段階を追っていきましょう

## Laravelだけで作るとどうなるか

（React使ってないやん、というツッコミは置いておいて）

Laravelはフルスタックフレームワークなので、単体でwebアプリケーションを作成することももちろんできます。  
Laravelではbladeというテンプレートエンジンを使用することができます。以下は典型的なbladeファイルの例です。ほとんどhtmlを書くように書くことができます。

top.blade.php

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $title }}</title>
</head>
<body>
    <h1>{{ $title }}</h1>
    <p>{{ $description }}</p>
</body>
</html>
```

複数ページが存在するwebアプリケーションの場合、ページの数だけこのようなbladeファイルを作成して、URLのパスごとに対応するコントローラーの処理を呼び出します。

web.php

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\TopPageController;

// ルートにアクセスされたときに、TopPageControllerのindexメソッドを読み出す
Route::get('/', [TopPageController::class, 'index']);
```

TopPageController.php

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TopPageController extends Controller
{
    public function index()
    {
        $data = [
            'title' => 'こんにちは、Laravel!',
            'description' => 'これはBladeテンプレートのサンプルです。'
        ];

        return view('top', $data);
    }
}
```

viewメソッドを実行する際に  
①どのビューを生成するのか  
②どんなデータを渡すのか  
を指定して返却することで動的なデータを持つページを作成することができます。

### 何がつらいのか

このように、Laravelだけでも十分webアプリケーションを作ることはできるのですが、以下のようなつらみがあります。

**ページに動きをつけにくい**  
データが動的に表示されるのはいいのですが、それだけだとやや物足りないですよね？  
もっとページ内でDOMを操作したり、SPAのようにページ遷移しないでいろんな処理をしたい場合、bladeだけでそれを表現するのはなかなか難しいです。  
[Livewire](https://livewire.laravel.com/) という、bladeと組み合わせて動的な処理を簡単に書くことができるライブラリもありますが、webでUIを作るならReactやVueなどのJavaScriptで書かれたフロントエンドフレームワークを利用したいですよね。

※Livewireを使ったことがなかったので軽く調べてみたのですが、思ったよりいろんなことができそうで論がブレそうなので、一旦ここはみなかったことにします（ [この記事](https://qiita.com/sgrs38/items/ee36ecc18fd0b7b3ba38) がわかりやすそうだった）。

## LaravelとReact(without Inertia)で作るとどうなるか

いろいろ方法はあるかと思いますが、一例として [こちらのサイト](https://juno-engineer.com/article/laravel-react-vite-spa/) のような構成を挙げてみます。

この記事内では正確な立ち上げ方法は記載しません。  
あくまで雰囲気を掴むためのものだと思ってください。

この構成の場合、Laravel側ではビューを返すルーティングをページごとには定義せず、 **基本的にすべてのパスへのアクセスから同じbladeファイルを返却するようにします。**

web.php

```php
<?php

use Illuminate\Support\Facades\Route;

Route::get('{any}', function () {
    return view('index');
})->where('any','.*');
```

どんなパスでも返却される `index.blade.php` は以下のようになっています。

index.blade.php

```html
<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="csrf-token" content="{{ csrf_token() }}">
    
        <title>{{ config('app.name', 'Laravel') }}</title>
        
        <!-- フォントの読み込み -->
        <link rel="dns-prefetch" href="//fonts.gstatic.com">
        <link href="https://fonts.bunny.net/css?family=Nunito" rel="stylesheet">
    
        <!-- vite経由でのスクリプトの読み込み -->
        @vite(['resources/js/app.jsx'])
    </head>
    <body>
        <div id="app"></div>
    </body>
</html>
```

`@vite()` で指定されている `resources/js/app.jsx` というファイルがフロント側のエントリポイントになり、以下のような記述になっています。

resources/js/app.jsx

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './components/App';

const rootElement = document.getElementById('app');

if (rootElement) {
    const root = ReactDOM.createRoot(rootElement);
    root.render(<App />);
}
```

vite経由でこのファイルを `index.blade.php` に読み込ませることで、  
`<div id="app"></div>` で指定されていた要素配下に対してReactアプリケーションが作成されます（この場合 `<App />` コンポーネントがレンダリングされる）。

`<App />` コンポーネント配下では、Laravel側で行わなかったパスごとのルーティングを  
React Routerを利用して制御します。

App.jsx

```jsx
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Top from './pages/Top';
import About from './pages/About';
import Contact from './pages/Contact';
import NotFound from './pages/NotFound';

const App = () => {
    return (
        <Router>
            <Routes>
                <Route path="/" element={<Top />} />
                <Route path="/about" element={<About />} />
                <Route path="/contact" element={<Contact />} />
                <Route path="*" element={<NotFound />} />
            </Routes>
        </Router>
    );
};

export default App;
```

このように記述することで、特定のパスの際には対応するコンポーネントをレンダリングさせるような制御をフロント側で行うことができるようになります。

### 何がつらいのか

Laravel側で制御していたルーティングをフロント側で制御するようになったことで、フロントではReactが使えるようになりました。異なるパス間の遷移でもReact Routerの仕組みを使えば再度htmlを送信してもらう必要はありません（いわゆるSPAです）。  
しかし、これにて落着と思いきや大事な部分が抜けていますね。

**データの取得処理を別で書かないといけない**

Laravelのみで画面遷移ごとにbladeを返却していた際にはデータも一緒に渡すことができていました。

```php
// example.blade.phpの中で使うデータも一緒に渡せる
return view('example', $data);
```

しかしReact Routerの場合、表示するべきデータを一緒に渡してくれはしません。あくまでデータを表示する枠組みを管理しているだけです。

```jsx
// どの画面を表示するかは指定できるが、データは取得してくれない
<Route path="/" element={<Home />} />
```

これはつまり、 **データ取得の処理は別で書く必要** があるということです。

データ取得の処理はいろいろ考えられますが、今回の場合はLaravel側にAPIを設定してデータを取得する例を挙げておきます。

まずAPIのルーティングを設定します。

api.php

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\TopPageController;

Route::get('/top-data', [TopPageController::class, 'index']);
```

次にルーティングに対応したコントローラーを用意します。

TopPageController.php

```php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class TopPageController extends Controller
{
    public function index()
    {
        $data = [
            'title' => 'こんにちは、Laravel×React!',
            'description' => 'これは表示に使用するデータのサンプルです。',
        ];

        // viewではなくデータだけを返却する
        return response()->json($data);
    }
}
```

実際にフロントエンドからAPI経由でデータを取得します。

Top.jsx

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const Top = () => {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    // 初回読み込み時にデータ取得用APIからデータ取得
    useEffect(() => {
        const fetchData = async () => {
            try {
                const response = await axios.get('/api/top-data');
                // 取得できたらstateに保存
                setData(response.data);
            } catch (err) {
                setError('Failed to load data');
            } finally {
                setLoading(false);
            }
        };

        fetchData();
    }, []);

    if (loading) return <p>Loading...</p>;
    if (error) return <p>{error}</p>;

    return (
        // 取得できたデータを表示
        <div>
            <h1>{data.title}</h1>
            <p>{data.description}</p>
        </div>
    );
};

export default Top;
```

Laravelのみを使っていたときにはviewと表示するデータを一緒に渡せていたのですが、この方法ではそれぞれの処理が別々に書かれることになり、わかりにくいです。  
APIがリソースごとに独立しているのならまだいいですが、上記の例のように各ページの初回ロード用のデータ取得APIを作り始めたら、ほとんど二度手間になってしまいます。

## Inertiaを使うとどうなるのか

（やっとここまで来ました...）  
ここまでの変遷をまとめると

**Laravelのみの場合**  
viewとデータを同時に渡せるが、ページに動きをつけにくかったり表示内容を変更する際に逐一ページ遷移を伴わないといけない場合が多い。

**Laravel×React without Inertiaの場合**  
view側でReactを使えるようになったことでページに動きをつけるのが簡単になり、SPA的な画面遷移も可能になったが、viewを初回読み込みする際のデータ取得処理を別で書かないといけない

表にするとこんな感じです（それぞれ全く出来ないというわけではないのですが、わかりやすさ重視で◯×でつけています）。

|  | ページに動きをつける | 初回読み込み時にviewとデータを同時に渡す |
| --- | --- | --- |
| Laravelのみ | × | ◯ |
| Laravel×React without Inertia | ◯ | × |

ではInertiaを使うとどうなるのかというと、（流れでなんとなくわかるかもしれませんが）こんな感じです。

|  | ページに動きをつける | 初回読み込み時にviewとデータを同時に渡す |
| --- | --- | --- |
| Laravelのみ | × | ◯ |
| Laravel×React without Inertia | ◯ | × |
| Laravel×React with Inertia | ◯ | ◯ |

Inertiaを使うと、 **素のLaravelでviewとデータを一緒に返却するかのような形でReactにデータを渡すことができるようになるのです！便利！**

## 具体的にどうやるのか

ルーティングの書き方は同じです。

web.php

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\TopPageController;

// /topにアクセスされたときに、TopPageControllerのindexメソッドを読み出す
Route::get('/top', [TopPageController::class, 'index']);
```

コントローラー内で処理を返却するところの処理が少し異なります。

TopPageController.php

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Inertia\Inertia;

class TopPageController extends Controller
{
    public function index()
    {
        $data = [
            'title' => 'こんにちは、Laravel×React!',
            'description' => 'これはInertiaのサンプルです。'
        ];

        return Inertia::render('Top', $data);
    }
}
```

といっても、データの整形の仕方自体はほとんど一緒で、最終的にreturnされるところの関数だけがInertia特有のものになっていますね。  
第一引数では `Top` という文字列を受け取り、これがどのReactコンポーネントを描画するべきなのかというのを指定しています。  
第二引数にはLaravelのview関数と同じようにページ内で表示したいデータを渡しています。

`Top` コンポーネントがどうなるのかも見てみましょう。

Top.jsx

```jsx
import React from 'react';

const Top = ({data}) => {

    return (
        // 取得できたデータを表示
        <div>
            <h1>{data.title}</h1>
            <p>{data.description}</p>
        </div>
    );
};

export default Top;
```

useEffectを利用して初回ロード時にデータ取得の処理を書いていたのがごっそりなくなって、代わりにTopコンポーネントのpropsという形でデータを取得することが出来ています。

Inertiaを使うことで、ほとんど **Laravelのview関数を返却するのと同じ感覚でReactのコンポーネントにデータを渡すことができます** 。  
ここまでの経緯を踏まえると、使いやすさが伝わったのではないでしょうか？（伝わってますよね？）

## 最後に

Inertiaにはまだまだ便利な機能があり、今回紹介した機能はInertiaのコアな機能のみとなっています。気になる方はInertiaの [公式サイト](https://inertiajs.com/) を見てみてください。  
次はInertiaの内部機構か、機能の紹介をしてみるつもりです（たぶん）。

[0](https://qiita.com/itokoooooh/items/#comments)

[18](https://qiita.com/itokoooooh/items/142a1d6f0aba7c5d798e/likers)

4