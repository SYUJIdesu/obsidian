---
title: "[Laravel] ソフトデリートと取得、復元、完全削除"
source: https://qiita.com/shinya_aa/items/b7c6c068a90ec7bdcd73
author:
  - "[[shinya_aa]]"
published: 2019-03-21
created: 2025-05-22
description: "#物理削除と論理削除データベースの削除には物理削除と論理削除があります。言葉はどうでもいいのですが、２つには大きな違いがあります。それは、復元可能かどうかということです。この復元可能な削除、い…"
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から5年以上が経過しています。

## 物理削除と論理削除

データベースの削除には物理削除と論理削除があります。言葉はどうでもいいのですが、２つには大きな違いがあります。  
それは、復元可能かどうかということです。  
この復元可能な削除、いわゆるゴミ箱に入っている状態にする削除方法が論理削除でソフトデリートになります。

## とりあえずデータベースを作る

データベースを作っていきましょう。

```terminal
php artisan make:migration create_contents_table --create=contents
```

ここではcontentsテーブルを作成しています。

create\_contents\_table.php

```php
public function up()
    {
        Schema::create('contents', function (Blueprint $table) {
            $table->increments('id');
            $table->text('body');
            $table->timestamps();

            $table->softDeletes();
        });
    }
```

textデータ型で、bodyカラムを作っている単純なデータベースです。  
最後の `$table->softDeletes();`がポイントです。  
このテーブルの要素に削除処理をするときはソフトデリートだよと、定義しています。

## 付随したモデルを作る

次に、データベースに付随したモデルを作りましょう。

```terminal
php artisan make:model Content
```

Content.php

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Content extends Model
{
    use SoftDeletes;

    protected $dates = ['deleted_at'];

    protected $fillable = [
        'body'
    ];
}
```

モデルでは、 `$fillable` でbodyをいじれるように設定しています。  
`use SoftDeletes` でソフトデリートを使用できるようにしており、削除した日がわかるように `$dates` でdeleted\_atカラムを作成しています。

ここまできたら `migrate` しましょう。

```terminal
php artisan migrate
```

## 中身が空なのでデータを入れる

現在は中身が空なので適当にデータを３つぐらい入れてみましょう。

web.php

```php
use App\Content;

Route::get('/contents/insert', function() {
    Content::create([
        'body' => 'Laravel is PHP framework!!',
    ]);
});

Route::get('/contents/insert2', function() {
    Content::create([
        'body' => 'PHP is programming language!!',
    ]);
});

Route::get('/contents/insert3', function() {
    Content::create([
        'body' => 'programming is a lot of fun!!',
    ]);
});
```

これで指定のURLにアクセスするとデータが入ります。

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/286470/f4a9bff7-41b7-90a6-d2fd-99c1da7e4448.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F286470%2Ff4a9bff7-41b7-90a6-d2fd-99c1da7e4448.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=da6eac6f0943973d6eb783ca2db7242a)

いけてますね。 `deleted_at` は現在NULLになっています。

## ソフトデリートと取得

本題のソフトデリートをしてみましょう。

web.php

```php
Route::get('/contents/softdelete', function() {
    Content::find(1)->delete();
});
```

idが１の要素を削除します。

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/286470/5fabb2bf-ac69-dd56-f63a-39b7fc5f96d4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F286470%2F5fabb2bf-ac69-dd56-f63a-39b7fc5f96d4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0722e4739f458aaa33797841742ee695)

いけましたね。削除した日が表示されておりデータベースにはあるけど取得はできません。

次は削除した要素を取得しましょう。

web.php

```php
Route::get('/contents/softdelete/get', function() {
    $content = Content::onlyTrashed()->whereNotNull('id')->get();

    dd($content);
});
```

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/286470/e645f5d9-622b-3fbe-154d-e4c39178adfa.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F286470%2Fe645f5d9-622b-3fbe-154d-e4c39178adfa.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1209655033932f65f29790dbab2821f3)

いけていますね。

## 他のメソッドも紹介

`onlyTrashed()` は削除済みの要素のみ取得するメソッドですが他のメソッドもあります。  
また完全削除や、復元のメソッドも紹介します。

```php
//削除済みのデータも含める
Content::withTrashed()->whereNotNull('id')->get();

//完全削除
Content::onlyTrashed()->whereNotNull('id')->forceDelete();

//復元
Content::onlyTrashed()->whereNotNull('id')->restore();
```

## まとめ

早くlaravelを使いこなせるようになりたいです。。。

[2](https://qiita.com/shinya_aa/items/#comments)

コメント一覧へ移動

[160](https://qiita.com/shinya_aa/items/b7c6c068a90ec7bdcd73/likers)

いいねしたユーザー一覧へ移動

152