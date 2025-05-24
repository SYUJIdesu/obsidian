---
title: Inertia.jsを使用したモダンなSPA開発
source: https://zenn.dev/shunjuio/articles/152dba0dc01bae
author:
  - "[[Zenn]]"
published: 2025-01-28
created: 2025-05-15
description: 
tags:
  - web
  - Laravel
---
[合同会社春秋テックブログ](https://zenn.dev/p/shunjuio) [Publicationへの投稿](https://zenn.dev/faq#what-is-publication)

23

5[tech](https://zenn.dev/tech-or-idea)

## はじめに

現在携わっているプロジェクトでは、LaravelとVue.jsを使用しており、その中でInertia.jsを導入しています。  
プロジェクトを通じて、その利便性や効率化の面でどのように役立つのかを学んだので、今回はInertia.jsの特徴や、実際の使用方法について、自身の学びをアウトプットを交えて紹介していきたいと思います。  
特に、従来のアーキテクチャとの違いや、Inertia.jsを使用することで得られるメリットに焦点を当てて説明したいと思います。

## Inertia.jsとは？

Inertia.jsは、公式サイトの説明によれば、「従来のサーバー駆動型Webアプリケーションの構築への新しいアプローチ」で「現代のモノリス」として位置づけられています。  
具体的には、サーバーサイドとクライアントサイドの間でAPIを作成せずに、状態を管理しながらフロントエンドとバックエンドを接続する方法を提供します。  
Vue.js、React、Svelteなどのフレームワークと組み合わせて利用されることが一般的です。

## フレームワークなのか

Inertia.jsは、厳密にはフレームワークではなく、軽量なライブラリです。  
ページ遷移をSPAのようにスムーズに処理してくれたり、サーバーから受け取ったデータを、クライアント側で簡単に扱えるようにしてくれます。  
個人的には、LaravelやRailsといったサーバーサイドと、Vue.jsやReactなどのクライアントサイドをつなぐ“接着剤”のようなイメージを持っています。

## 従来のSPA開発における課題とInertia.jsの利点

### 従来のSPAとAPIの課題

一般的に、SPAを構築する際には次のようなプロセスが必要でした。

#### 1\. バックエンドでAPIを作成する

サーバーサイドでデータを提供するためのREST APIやGraphQL APIを設計・構築する。

#### 2\. フロントエンドでAPIを使ってデータを取得する。

フロントエンド（Vue.jsやReactなど）で、APIを呼び出してデータを取得し、そのデータを元にUIを動的に生成する。

上記のプロセスには、次のような負担が伴います。

- APIの設計と実装（エンドポイント、認証、バリデーションの処理など）
- フロントエンドとバックエンド間のデータフォーマットの統一
- バックエンドとフロントエンドで重複するロジック（認証やバリデーション）

特に小規模から中規模のプロジェクトでは、これらの作業が多すぎて、かえって開発効率が落ちることがあります。

## Inertia.jsがもたらす解決策

Inertia.jsは、上記のような従来のSPA開発で発生する負担を軽減します。APIの設計やデータ通信の手間を省き、フロントエンドとバックエンドの連携をシンプルにする方法を提供します。具体的には、以下のような仕組みを採用しています。

#### APIの設計と実装

Inertia.jsでは、APIの設計やエンドポイントを意識する必要がありません。サーバーサイドで生成したデータをそのままフロントエンドに渡す仕組みを提供しています。  
たとえば、Laravelを使った場合、以下のようにデータを簡単に送信できます。

```php
// Laravelコントローラ例
public function index() {
    $users = User::all();
    return Inertia::render('Users/Index', [
        'users' => $users
    ]);
}
```

このコードでは、サーバー側で取得したデータを指定したフロントエンドのコンポーネントに直接渡しています。(具体的な実装方法については後ほど解説します。)  
まるでBladeファイルを使う感覚でデータをフロントエンドに渡せるため、APIの設計や実装にかかる手間や時間を大幅に削減できます。

#### フロントエンドとバックエンド間のデータフォーマットの統一

Inertia.jsでは、サーバーサイドからそのままデータを渡すため、フロントエンドとバックエンド間でフォーマットを意識する必要がありません。  
Laravelのコントローラで生成したデータを直接VueやReactのコンポーネントに `props` として渡すことが可能で、フロントエンドでは、 `props` として受け取ったデータをそのまま利用できます。

#### バックエンドとフロントエンドで重複するロジックの解消

Inertia.jsでは、バリデーションや認証といったロジックをすべてバックエンドに集約し、結果をフロントエンドにそのまま渡します。たとえば、Laravelを利用した場合、以下のようにバリデーションをバックエンドで実行します。

```php
public function store(Request $request) {
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users',
    ]);

    User::create($validated);

    return redirect()->route('users.index');
}
```

エラーメッセージもInertia.jsを通じてフロントエンドに自動的に渡されるため、フロントエンドではバリデーションの実装は不要で、受け取ったエラーをそのまま表示するだけで済みます。

以上のように、Inertia.jsはこれまでのSPAで抱えていた多くの課題を解消し、シンプルで効率的な開発環境を提供します。特に、API設計やデータフォーマット、ロジックの重複といった開発プロセスのボトルネックを解消することで、チーム全体の生産性を高めることが可能です。

## Inertia.jsの仕組み

ここでは、Inertia.jsがどのようにSPAのようなページ遷移を実現しているのか、その仕組みを詳しく解説します。

### 1.Inertiaの<Link>コンポーネントによるページ遷移の管理

Inertia.jsの `<Link>` コンポーネントを使用すると、クリック時にHTTPリクエストが [`XHR`](https://developer.mozilla.org/ja/docs/Web/API/XMLHttpRequest) として送信され、ブラウザがページを再読み込みすることなく遷移します。これにより、フルページリロードが回避され、ページ間遷移が速くなります。  
実際には、 `<Link>` タグをクリックすると、リクエストのヘッダーに `X-Inertia: true` というカスタムヘッダーが追加されます。  
このヘッダーが、Inertia.jsがバックエンドとの通信でページの遷移を管理するための合図になります。

### 2.InertiaがLaravelに送るリクエストとそのレスポンス

通常、Laravelでページを返す場合、Viewをレンダリングし、HTML（DOM）を返します。しかし、Inertia.jsを使った場合、リクエストヘッダーに `X-Inertia: true` が付与されると、レスポンスとしてHTMLではなく、JSON形式でデータが返されることになります。

### レスポンスの挙動

- **初回アクセス時： `X-Inertia: true` ヘッダーがない場合**  
	通常通り、LaravelはViewを返し、ブラウザはHTML（DOM）を表示します。  
	例えば、 `return view('users.index', compact('users'));`のように、ユーザーの一覧ページを返すと、HTMLがそのまま表示されます。

![](https://storage.googleapis.com/zenn-user-upload/8ef97eba8c87-20250128.png)

- **2回目以降のリクエスト： `X-Inertia: true` ヘッダーがある場合**  
	`<Link>` コンポーネントなどをクリックして画面遷移を行う際、リクエストヘッダーに  
	`X-Inertia: true` が含まれます。このリクエストに対し、サーバーは必要なデータをJSON形式で返します。  
	![](https://storage.googleapis.com/zenn-user-upload/86eaddfcdb8b-20250128.png)

これにより、ページ遷移の際に新しいHTMLを生成するのではなく、必要なデータのみをJSONとして返し、フロントエンド側で動的にコンテンツを更新します。

## Inertia.jsの動作の流れ

1. フロントエンドで `<InertiaLink>` がクリックされる。
2. JavaScriptはページ遷移を阻止し、XHRリクエストをバックエンドに送信。
3. リクエストには `X-Inertia: true` ヘッダーが付与される。
4. バックエンドは、 `X-Inertia: true` を確認し、JSON形式で必要なデータを返す。
5. フロントエンドでは、受け取ったデータを元にページを動的に更新する。

これにより、従来のSPAのようにフロントエンドとバックエンドでAPIを個別に設計することなく、シンプルに動的なページ遷移を実現できます。

## Inertia.jsの基本

この章では、Inertia.jsの基本的な仕組みや実装方法を解説するとともに、実際のプロジェクトで使用された具体例を交えながら紹介していきます。

## データの受け渡し

Inertia.jsでは、バックエンドとフロントエンド間でデータを受け渡すために、 `Inertia::render` メソッドを使用します。このメソッドを使うことで、従来のAPIを使ってデータを取得する手間を省き、ページを表示する際に必要なデータを簡単に渡すことができます。

以下はLaravel側で `Inertia::render` を使ってVueコンポーネントにデータを渡し、フロントエンドでそのデータを利用する方法です。

#### Laravelでの実装

`UserController` でユーザー情報をビューに渡すコード例です。

```php
use Inertia\Inertia;

class UserController extends Controller
{
    public function show(User $user)
    {
        return Inertia::render('User/Show', [
          'user' => $user
        ]);
    }
}
```

### Inertia::render

- 第一引数にはVueコンポーネント（User/Show）を指定します。  
	これは `resources/js/Pages/User/Show.vue` にマッピングされます。
- 第二引数には、コンポーネントに渡したいデータを配列形式で指定します。  
	ここでは、userオブジェクトを渡しています。

#### Vue.jsでの実装

次に、Laravelから渡されたデータを受け取るVueコンポーネントを作成します。

```html
<script setup>
import Layout from './Layout'
import { Head } from '@inertiajs/vue3'

// Laravelから渡されたデータをpropsとして受け取る
defineProps({ user: Object })
</script>

<template>
  <Layout>
    <Head title="ユーザー詳細" />
    <!-- コンテンツ表示 -->
    <h1>ユーザー詳細</h1>
    <p>{{ user.name }}さん、Inertiaアプリへようこそ！</p>
  </Layout>
</template>
```

- **defineProps**  
	Laravelから渡されたデータは、 `props` として受け取ります。この例では、userオブジェクトを受け取り、コンポーネント内で利用しています。

#### 実行結果

1. `UserController@show` アクションが呼び出されると、 `user` データを含むレスポンスがVueコンポーネントに送信されます。
2. Vueコンポーネントでは、 `props` で受け取ったデータを利用して、ユーザー名やその他の情報を表示できます。
3. 最終的に以下のようなページがレンダリングされます。
	```html
	<h1>ユーザー詳細</h1>
	<p>山田太郎さん、Inertiaアプリへようこそ！</p>
	```

## 非同期通信

Inertia.jsでは、従来のSPAのようにページを遷移する際に非同期通信を使います。  
`router.visit()` メソッドを使うことで、ページの遷移がスムーズに行われ、再読み込みなしで新しいページを表示できます。

例えば、リンクをクリックして新しいページを非同期で読み込むには、次のように記述します。

上記のコードでは、 `navigateToPost` メソッドを使って、 `/posts/{id}` というURLに非同期で遷移します。これにより、ページ遷移がスムーズに行われ、従来のフルページリロードが不要になります。

`router.visit()` を使うと、フォーム送信やファイルアップロードも簡単に実装できます。特にPOSTリクエストを送る方法として、以下の2つのアプローチがあります。

- `router.visit()` メソッドで `method: 'post'` を指定する
- `router.post()` メソッドを使用する

どちらも、送信するデータはそれぞれのメソッドで指定できます。

#### 1\. router.visit()メソッドを使用する方法

`router.visit()` を使ってPOSTリクエストを送る場合は、 `method: 'post'` を指定し、データは `data` オプションで渡します。

```html
<script setup>
import { useInertia } from '@inertiajs/vue3';

const { visit } = useInertia();

visit('/Bookmark/store', {
    method: 'post',
    data: {
        title: 'Yahoo!',
        url: 'https://www.yahoo.co.jp/',
    },
});
</script>
```

この方法では、 `visit()` メソッドを使いながら、 `POST` メソッドを指定してデータを送信します。

#### 2\. router.post()メソッドを使用する方法

`router.post()` は、フォームデータを送信する際に便利な専用のメソッドです。  
以下のように、送信するURLとデータを指定できます。

```html
<script setup>
import { useInertia } from '@inertiajs/vue3';

const { post } = useInertia();

const submitBookmark = () => {
  post('/Bookmark/store', {
    title: 'Yahoo!',
    url: 'https://www.yahoo.co.jp/',
  });
};
</script>
```

この方法はシンプルで直感的に使用でき、 `POST` メソッドでデータを送信します。

## フォーム送信

Inertia.jsでは、フォーム送信を簡単に行うための `Form` クラスを提供しています。  
このクラスを使うことで、フォームの状態管理や送信処理を効率的に行うことができ、従来のSPAでよくある面倒なフォーム送信処理を簡潔にします。  
以下はフォームヘルパーを利用してフォーム送信の実装をした例です。

```html
<script setup>
//useFormをインポート
import { useForm } from '@inertiajs/vue3';

//フォームデータの初期化
const form = useForm({
    name: props.name,
    age: props.age,
    address: props.address,
});

const submit = () => {
    form.post(route('bookmark.store'));
}
</script>
```

フォーム送信に使えるメソッドは、HTTPメソッドを指定する `submit()` や、各HTTPメソッド用の `get()`,`post()`,`put()`,`patch()`,`delete()` が用意されています。

```js
form.submit(method, url, options)
form.get(url, options)
form.post(url, options)
form.put(url, options)
form.patch(url, options)
form.delete(url, options)
```

#### transform()メソッド

`transform()` メソッドを使用すると、フォームのデータを送信前に加工できるため、ユーザーが入力したデータをサーバーに送信する前に処理や検証を行うことができます。  
例えば、名前の値を大文字に変換したり、日付のフォーマットを変えたりすることが可能です。

```html
<script setup>
//useFormをインポート
import { useForm } from '@inertiajs/vue3';

//フォームデータの初期化
const form = useForm({
    name: props.name,
    email: props.email,
    birth_date: props.birth_date, // ユーザーの生年月日
});

// submit メソッド
const submit = () => {
  form.transform((data) => ({
    name: data.name.trim().toUpperCase(),  // nameの値をトリムし、大文字に変更
    email: data.email.trim().toLowerCase(),  // emailの値をトリムし、小文字に変更
    birth_date: new Date(data.birth_date).toISOString(), // 生年月日をISOフォーマットに変換
  }))
  .put(route('your.route.name', { id: props.id }));
};
</script>
```

この例では、以下の変更が行われます。

- `name` フィールドは、前後の空白を削除した後、大文字に変換されます。
- `email` フィールドも同様に空白を削除し、小文字に変換されます。
- `birth_date` は `Date` オブジェクトに変換し、ISO 8601 フォーマット（例: 2025-01-26T00:00:00Z）に変換されます。

このように、 `transform()` メソッドを使用すると、送信前にデータを加工でき、サーバーサイドでの処理が効率的になります。

## まとめ

この記事では、Inertia.jsが提供する「密結合（モノリス）」アーキテクチャの利点に焦点を当て、従来のAPIベースのアーキテクチャに比べてどれほど開発が効率化されるかを紹介しました！  
しかし、Inertia.jsのアプローチが全てのプロジェクトに最適というわけではありません。プロジェクトによっては、バックエンドとフロントエンドをAPIで分けて疎結合にするアーキテクチャにも大きなメリットがあります。  
今回はInertia.jsの利点を中心に解説しましたが、今後は疎結合のアーキテクチャを使った開発のメリットやそのアプローチについても学び、さらに深掘りして記事にしていけたらと思います。📖

最後までご覧いただき、ありがとうございました🙇♂️

23

5