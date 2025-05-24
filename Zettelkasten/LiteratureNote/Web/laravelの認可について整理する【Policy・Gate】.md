---
title: laravelの認可について整理する【Policy・Gate】
source: https://qiita.com/Masataka_Sugia/items/72c0e680e23eca960ddb
author:
  - "[[Masataka_Sugia]]"
published: 2024-03-02
created: 2025-05-22
description: 初めに開発中に認可処理を行う必要があり初めてlaravelの認可の機能に触れた。これは調べて分かったことをなどを整理するための記事。認可 is 何?認可を調べると大体同じ文脈に認証というワ…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

## 初めに

開発中に認可処理を行う必要があり  
初めてlaravelの認可の機能に触れた。

これは調べて分かったことをなどを整理するための記事。

## 認可 is 何?

認可を調べると大体同じ文脈に認証というワードが出てくる。

**認証とは**  
認証とはアプリケーションが利用者本人であるかという検証のことですね  
emailとかでログインする時のプロセスのことですね。

**認可とは**  
認証後にアクセス権限に応じてITリソースへのアクセスを制御する仕組みのことですね  
例えば:有料会員になったら〇〇が使えるとか、その辺がこれに当たりますね。

## laravelにおいての認可の方法

laravelにおいて認可を行う方法には  
**Policy** と **Gate** がある。

どちらを使うべきか?と言うわけではなく  
用途に合わせて使うものですね。

> プリケーションを構築するときに、ゲートのみを使用するか、ポリシーのみを使用するかを選択する必要はありません。ほとんどのアプリケーションには、ゲートとポリシーが混在する可能性が高く、それはまったく問題ありません。ゲートは、管理者ダッシュボードの表示など、モデルやリソースに関連しないアクションに最も適しています。対照的に、特定のモデルまたはリソースのアクションを認可する場合は、ポリシーを使用する必要があります。

laravelのドキュメントにもこのように書かれていますね。

- ゲートはモデルやリソースに関連しないアクションに最も適しています。(管理者ダッシュボードの表示など)
- ポリシーは特定のモデルまたはリソースのアクションを認可する場合に適している。

と言ってますね。

## Gateについて

ゲートはモデルやリソースに関連しないアクションに最も適しています。(管理者ダッシュボードの表示など)  
例えば  
有料会員になったら新しい機能が画面に追加されるとかあると思います。  
これは大体画面にif文を入れて有料会員だったらこの機能を見せる、みたいに処理していますよね。  
このifの条件文をGateに任せるというものですね。

実際のコードを見てみます。

ユースケースとしては  
有料会員かどうかみたいな処理をすることにします。

実際に挙動の確認は取っていませんので大まかな流れだけ把握できることをゴールとしています。

### 作成

App\\Providers\\AuthServiceProvider.php

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * 全認証／認可サービスの登録
 */
public function boot(): void
{
    Gate::define('isPremium', function (User $user) {
        //有料会員だったらtrueじゃなかったfalse
        return (bool)$user->is_premium;
    });
}
```

大体こんな感じですかね。  
laravelのGateにおいて `User $user` は認証ユーザーがGateから渡されます。  
この時点でお!となりますね。

### 呼び出し

> アビリティを認可するためのゲートメソッド（allows、denis、check、any、none、authorize、can、cannot）

laravelのドキュメントを見るとこんなことが書かれています。  
これは呼び出しの方法の種類ですね。

**allows**  
指定したアビリティが許可されているかどうかを確認します。許可されていればtrueを返します。

**denies**  
指定したアビリティが拒否されているかどうかを確認します。拒否されていればtrueを返します。

**check**  
指定した一つ以上のアビリティが全て許可されているかどうかを確認します。全て許可されていればtrueを返します。

**any**  
指定した一つ以上のアビリティのうち、いずれかが許可されているかどうかを確認します。一つでも許可されていればtrueを返します。

**none**  
指定した一つ以上のアビリティが全て拒否されているかどうかを確認します。全て拒否されていればtrueを返します。

**authorize**  
指定したアビリティが許可されているかどうかを確認し、許可されていない場合は例外を投げます。これはアクションが許可されていることを確実にする場合に便利です。

例えば

.php

```php
if (Gate::allows('isPremium')) {
    // 有料会員の場合の処理
}

if (Gate::denies('isPremium')) {
    // 有料会員ではない場合の処理
}

// アクションが許可されていない場合は、例外を投げる
Gate::authorize('isPremium');
```

こんな感じで呼び出します。

別にGateを使わなくても  
特定のクラスに認可処理を書いて呼び出すことで最終的なやりたいこと(要件)を満たすことは可能かと思います。  
ただ  
laravelでの開発において認可に関する処理をGateにまとめることでシンプルを保つことができるのではと思いますね。  
(認可ロジックが散らからないと言う意味で)

もし

.php

```php
if ($user->is_premium === true)) {
    // 有料会員の場合の処理
}

if ($user->is_premium === false) {
    // 有料会員ではない場合の処理
}

// アクションが許可されていない場合は、例外を投げる
if ($user->is_premium === false) {
    throw new Exception('許可されていません。');
}
```

こういうコードだった場合  
動くけど、プロダクトとのいろんなところで認可処理が散らかりますよね。(保守しんどそう)

特に今回のユースケースである有料会員かどうかは  
プロダクト全体に影響するものだと思うので  
ロジックを一箇所にまとめておきたい。ですね。

### マルチログインをしているシステムだったらどうするのか

記事を書いているときに気になったので調べました。

マルチログインとは  
ようは  
一般ユーザーと管理者のログインなど複数のユーザータイプがある場合のことですよね。

テーブルとしては  
一般ユーザーは `customerTable`  
管理者は `adminTable`  
とかになりますかね。

Gateは認証ユーザーを `User $user` で提供するという話でしたが  
おそらくデフォルトの `guard` の設定での認証ユーザーを返してくると思います。

ので、

管理者に対しての認可を行いたい場合は  
Gate内部でログインユーザーを直接取得する必要がありそうです。

管理者なのでユースケースとしては  
master権限かどうかと言うふうにしてみます。

App\\Providers\\AuthServiceProvider.php

```php
use Illuminate\Support\Facades\Auth;

Gate::define('isMaster', function () {
    // 特定のガードのユーザーを取得
    $admin = Auth::guard('admin')->user();

    return $admin->role === 'master';
});
```

こんな感じになりますかね。  
認証ユーザーを自動で返してくれるといううまみは無くなりますが  
認可処理を一箇所にまとめておくというという意味では  
まだいいですかね。

以上がGateに関しての大枠ですかね。  
Gateの細かい使い方はドキュメントに詳しく書かれています。

僕が理解しているのはここまでですので  
詳しくは公式を参照ください。

## Policyについて

> ポリシーは、特定のモデルまたはリソースに関する認可ロジックを集めたクラスです。アプリケーションがブログの場合、App\\Models\\Postモデルと投稿の作成や更新などのユーザーアクションを認可するためのPostモデルと対応するApp\\Policies\\PostPolicyがあるでしょう。

とドキュメントに書いていますね。

qiitaで記事を書いているので  
この記事をPostとします。  
例えば僕が他の人の記事を編集できないようにしたり削除できないようにしたり制御する仕組みですね。  
僕(ログインユーザー)と特定のモデル(qiitaの記事)に関する認可ロジックですね。

App/Policies/PostPolicy.php

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * 指定した投稿をユーザーが更新可能かを判定
     */
    public function update(?User $user, Post $post): bool
    {
        return $user?->id === $post->user_id;
    }
}
```

```text
$php artisan make:policy PostPolicy --model=Post
```

こんな感じでモデル名を指定することでそのモデルに対するPolicyクラスを作成することができます。

> Artisanコンソールを介してポリシーを生成するときに--modelオプションを使用した場合、はじめからviewAny、view、create、update、delete、restore、forceDeleteアクションのメソッドが用意されます。

これらのメソッドがlaravelから自動で提供されますが  
それ以外の任意のメソッドを定義することもできます。

policyは [リソースコントローラー](https://readouble.com/laravel/10.x/ja/controllers.html#resource-controllers) と相性が良く作られていて

| コントローラメソッド | ポリシーメソッド |
| --- | --- |
| index | viewAny |
| show | view |
| create | create |
| store | create |
| edit | update |
| update | update |
| destroy | delete |

このような紐づきをしてくれるそうです。  
アプリケーション次第ではすごく便利そうですね。

App/Http/Controllers/PostController.php

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Post;

class PostController extends Controller
{
    /**
     * コントローラインスタンスの生成
     */
    public function __construct()
    {
        $this->authorizeResource(Post::class, 'post');
    }
}
```

こうすることでコントローラーのアクションに対して  
対応するモデルのpolicyアクションが呼び出されるのですね。

モデルとPolicyの紐付けは  
命名規則によって解釈されています。

App/Http/Controllers/PostController.php

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * 指定した投稿を更新
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if ($request->user()->cannot('update', $post)) {
            abort(403);
        }

        // 投稿を更新…

        return redirect('/posts');
    }
}
```

このコードではどのPolicyクラスのupdateを呼ぶのか  
と言う指定をしていません。

これは私た$postのモデルクラスによって  
どのPolicyなのかと言うのを解釈してくれています。

明示的に指定する方法としては  
`AuthServiceProvider` 内の `$policies` プロパティにポリシーを登録します。  
これにより、LaravelはPostモデルに対する認可チェックが必要な場合、PostPolicyを使用することを知ります。

AuthServiceProvider.php

```php
protected $policies = [
    Post::class => PostPolicy::class,
];
```

これなくても動きましたがないと気持ち悪いので入れたいお気持ちですね。

### middlwareでも使える

web.php

```php
Route::put('/post/{post}', function (Post $post) {
    // 現在のユーザーは投稿を更新可能
})->middleware('can:update,post');
```

> この例では、canミドルウェアに２つの引数を渡します。１つ目は認可するアクションの名前であり、２つ目はポリシーメソッドに渡すルートパラメーターです。この場合、暗黙のモデルバインディングを使用しているため、App\\Models\\Postモデルがポリシーメソッドに渡されます。ユーザーが特定のアクションを実行する権限を持っていない場合、ミドルウェアは403ステータスコードのHTTPレスポンスを返します。

省略した書き方もできるみたい

web.php

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 現在のユーザーは投稿を更新可能
})->can('update', 'post');
```

これ知らんかった。。

## 終わり

ドキュメントを見るとGateの説明ででPolicyみたいな使い方(モデルに対する制御)をしていたり  
若干まだ混乱していますが

僕の中での解釈としては

- モデルに対する認可はPolicy
- モデルに関係のない認可はGate

この使いわけで一旦落ち着いています。

僕の解釈が間違っていたり  
間違いに気づいた場合ご指摘お願いいたします。

## 参考記事

[0](https://qiita.com/Masataka_Sugia/items/#comments)

[7](https://qiita.com/Masataka_Sugia/items/72c0e680e23eca960ddb/likers)

2

ストックを更新しました