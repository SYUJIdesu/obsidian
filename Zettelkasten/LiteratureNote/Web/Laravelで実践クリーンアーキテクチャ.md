---
title: Laravelで実践クリーンアーキテクチャ
source: https://qiita.com/nrslib/items/aa49d10dd2bcb3110f22
author:
  - "[[nrslib]]"
published: 2019-03-14
created: 2025-05-22
description: この記事を書くにあたって Laravel について色々サポートしてくれた皆さまに向けてお礼申し上げます。ありがとうございました。本記事はクリーンアーキテクチャに対する理解を深めていただくために、「…
tags:
  - web
  - オブジェクト指向
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から3年以上が経過しています。

この記事を書くにあたって Laravel について色々サポートしてくれた皆さまに向けてお礼申し上げます。ありがとうございました。

本記事はクリーンアーキテクチャに対する理解を深めていただくために、「実践クリーンアーキテクチャ」の内容を Laravel で実装して解説するという内容になっています。  
記事のゴールは「クリーンアーキテクチャに対する理解を深めてもらう」というものです。つまり、この実装の形は一例に過ぎません。

## はじめに

皆さんクリーンアーキテクチャはご存知でしょうか。  
そう、こんな図のアレです。  
[![clean.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/f9f3dee0-8ea8-8cc9-2f9b-311f238b3db5.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Ff9f3dee0-8ea8-8cc9-2f9b-311f238b3db5.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b0a6ad43f9460b3dae1ea6aeaccda7fa)  
The Clean Architecture: [https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

クリーンアーキテクチャといえばこちらの象徴的な図をまずは思い浮かべるでしょう。  
この図をパッと眺めてみると非常に抽象的な図にみえてしまうので「実際どんなコードにすればいいの？」と感じるというのは否めません。  
しかしながら実をいうとこれはそのまま実装に落とし込むことが可能だったりします。  
であれば実装例を作ったらいいのかな、というわけでこちらの図を元に実装したものが次のふたつの記事です。

実装クリーンアーキテクチャ： [https://qiita.com/nrslib/items/a5f902c4defc83bd46b8](https://qiita.com/nrslib/items/a5f902c4defc83bd46b8)  
実践クリーンアーキテクチャ： [https://nrslib.com/clean-architecture/](https://nrslib.com/clean-architecture/)

この二つは同じものに見えますが、タイトルによく目を凝らしてみると "実「装」クリーンアーキテクチャ" と "実「践」クリーンアーキテクチャ" と異なっているのです。  
つまり実は違うもの。

具体的に何が違うのか。

実「装」の方は図を忠実に再現しましょうというものです。  
そして、実「践」の方は一部改変しています。（文章中でもそのことについて触れています。）

もちろん改変したのには理由があります。  
もともとクリーンアーキテクチャはすべてのソフトウェアに通じるアーキテクチャとして紹介されていました。  
しかしながら、これを Web アプリケーションにおいて採用しようとするとどうしても扱いにくい部分があります。それは具体的には表示にともなう部分なのですが、これがどうにも MVC フレームワークを利用したときに相性が悪い。（どう相性が悪いかは実践クリーンアーキテクチャの記事および当記事にて。）

この件についてはよい方策がないか考えてみたのですが、この部分についてをそのまま忠実にしなくてもクリーンな構造は保たれるという結論に達したため、実践クリーンアーキテクチャの形で記事を公開しました。

で、この記事はなにかというと、C# で書かれていた実践クリーンアーキテクチャを PHP (Laravel) で実践するとこうなります、というサンプルの解説です。

## コード

このあとの解説は次のサンプルコードを元にお話しいたします。  
[https://github.com/nrslib/LaraClean](https://github.com/nrslib/LaraClean)

余裕があれば実際のコードと照らし合わせながら確認するとよいでしょう。

## 同心円の図

図がすべてというわけではありませんが、せっかく象徴的な図がありますので、この図に沿って実装の紹介と解説を行っていきます。

早速、同心円の図をご覧ください。  
こちらの図の円形部分ですね。  
[![clean.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/f9f3dee0-8ea8-8cc9-2f9b-311f238b3db5.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Ff9f3dee0-8ea8-8cc9-2f9b-311f238b3db5.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b0a6ad43f9460b3dae1ea6aeaccda7fa)  
さて、図をご覧になれば分かるとおり、多くの用語が登場しています。  
いきなり多くの用語に囲まれると、それらひとつひとつが簡単なものであったとしても混乱を起こすものです。  
ここではなるべく登場人物が一人ずつ増えていくように解説をしていきます。

## Entities

[![entities.PNG](https://qiita-image-store.s3.amazonaws.com/0/293368/54afe2b9-ab90-3c6d-5d54-e66d8a7688ee.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F54afe2b9-ab90-3c6d-5d54-e66d8a7688ee.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=51d871770583aa02be9fd96776d59fd8)  
エンティティという用語を見ると、もしかしたらドメイン駆動設計のエンティティが頭に思い浮かぶ方もいらっしゃるかもしれません。  
それ以外にもデータベースのテーブルをオブジェクトに起こしたものなどが思い浮かぶこともあるでしょう。

クリーンアーキテクチャにおけるエンティティはそれらのいずれとも異なります。

エンティティはビジネスルールをカプセル化したオブジェクトです。  
いわゆるドメインモデルと言われるもので、ドメイン駆動設計のエンティティと比べるとその定義幅はかなり広いです。

このあたりの考察を詳しく確認したい方は下記 URL へどうぞ。  
ドメイン駆動設計のエンティティとクリーンアーキテクチャのエンティティ: [https://nrslib.com/clean-ddd-entity/](https://nrslib.com/clean-ddd-entity/)

さて、今回のアプリケーションでエンティティとして用意しているのは次のディレクトリに存在するものです。

- packages\\Domain\\Domain

具体的には User や UserId などがこれにあたります。  
モデルとしては実装が貧弱なのですが、エンティティそれ自体は主題ではないのでこれで次の項目に進みましょう。

## Gateways

[![gateways.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/5af028b4-83ff-6f0f-81a7-81637bb950ed.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F5af028b4-83ff-6f0f-81a7-81637bb950ed.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9b0ed53abc09eb1b8ae0715e6e6c2f1f)  
今回のアプリケーションではリポジトリとして用意されているものが該当しています。

リポジトリを訳すと倉庫とかそういった意味になります。  
ソフトウェアの文脈におけるリポジトリはデータの倉庫、つまりデータ永続化を担うオブジェクトです。

そういったゲートウェイにあたるものは次のモジュールです。

- packages\\Domain\\Domain\\User\\UserRepositoryInterface
- packages\\Infrastructure\\User\\UserRepository
- packages\\InMemoryInfrastructure\\User\\InMemoryUserRepository

なかなかいろいろな名前空間のモジュールが出てくるので難しく思えるかもしれませんが、ひとつひとつはとても簡単なものです。  
早速解説に入ります。

### packages\\Domain\\Domain\\User\\UserRepositoryInterface

```php
interface UserRepositoryInterface
{
    /**
     * @param User $user
     * @return mixed
     */
    public function save(User $user);

    /**
     * @param UserId $id
     * @return User
     */
    public function find(UserId $id);

    /**
     * @param int $page
     * @param int $size
     * @return mixed
     */
    public function findByPage($page, $size);
}
```

リポジトリの interface はデータ永続化手続きの決まりごとです。

- ある特定のデータ（ここでは User）のデータを保存する際にはどのような保存をするのか。（save メソッド。）
- 保存されたデータを取得するにはどのような手段があるのか。（find, findByPage メソッド。）

こういった決まりごとを表しています。

このようにデータを保存・再構築することを指してデータの永続化といいます。

なお、データの永続化というと、データベースとかファイルなどの直接的な保存媒体を思い浮かべるかもしれません。  
しかし、ここでのデータの永続化はそれに限りません。  
これがどういうことなのかはこの先の InMemoryUserRepository で詳しく解説をします。  
いったんは次の項目へ進みましょう。

### packages\\Infrastructure\\User\\UserRepository

```php
class UserRepository implements UserRepositoryInterface
{
    /**
     * @param User $user
     * @return mixed
     */
    public function save(User $user)
    {
        DB::table('users')
            ->updateOrInsert(
                ['id' => $user->getId()],
                ['name' => $user->getName()]
            );
    }

    /**
     * @param UserId $id
     * @return User
     */
    public function find(UserId $id)
    {
        $user = DB::table('users')->where('id', $id->getValue())->first();

        return new User($id, $user->name);
    }

    /**
     * @param int $page
     * @param int $size
     * @return mixed
     */
    public function findByPage($page, $size)
    {
        // TODO: Implement findByPage() method.
    }
}
```

見て分かるとおり、データをデータベースに保存しています。（一部未実装。）  
UserRepository はデータベースに対してデータの永続化を行うオブジェクトです。

### packages\\InMemoryInfrastructure\\User\\InMemoryUserRepository

```php
class InMemoryUserRepository implements UserRepositoryInterface
{
    private $db = [];

    /**
     * @param User $user
     * @return mixed
     */
    public function save(User $user)
    {
        $this->db[$user->getId()->getValue()] = $user;
        var_dump($this->db);
    }

    /**
     * @param UserId $id
     * @return User
     */
    public function find(UserId $id)
    {
        $found = $this->db[$id->getValue()];
        return $this->clone($found);
    }

    /**
     * @param User $user
     * @return User
     */
    private function clone(User $user){
        $cloned = new User($user->getId(), $user->getName());
        return $cloned;
    }

    /**
     * @param int $page
     * @param int $size
     * @return mixed
     */
    public function findByPage($page, $size)
    {
        $start = ($page - 1) * $size;
        return array_slice($this->db, $start, $size);
    }
}
```

InMemoryUserRepository は `$db` というデータベースに見立てた連想配列に対してデータを保存しています。  
クラス名が示しているとおり、このオブジェクトはメモリを利用してデータの永続化を行っています。  
少し前にお話したデータベースでもファイルでもないというのはこちらのモジュールのことです。

リポジトリはデータの保存と再構築ができれば、それがデータベースでもファイルでもメモリでも、媒体はなんでもよいのです。

この InMemoryRepository を利用すればデータベースを用意することなく、アプリケーションを仮動作させることが可能になります。  
それはとても素晴らしいことだと思いませんか。

## UseCases

[![usecases.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/61b4765f-9f14-36ae-ca4a-9eb7b3a583af.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F61b4765f-9f14-36ae-ca4a-9eb7b3a583af.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=daf2220955525a9c9aa56bec45c6b01e)  
次はユースケースです。

ユースケースはアプリケーションの API です。  
アプリケーションが何をできるのかというのを表します。  
よって、アプリケーションとして可能なことを表現している interface とその実装がこれにあたります。

具体的には次のモジュール群です。

- ユーザ作成処理
- packages\\UseCase\\User\\Create\\UserCreateUseCaseInterface
- packages\\Domain\\Application\\User\\UserCreateInteractor
- packages\\MockInteractor\\User\\MockUserCreateInteractor
- ユーザ取得処理
- packages\\UseCase\\User\\GetList\\UserGetUseCaseInterface
- packages\\Domain\\Application\\User\\UserGetListInteractor
- packages\\MockInteractor\\User\\MockUserGetInteractor

これらの処理は操作が異なるだけで構成は同じですので、ここではユーザ作成処理に絞ってモジュールを確認していきましょう。

#### packages\\UseCase\\User\\Create\\UserCreateUseCaseInterface

```php
interface UserCreateUseCaseInterface
{
    /**
     * @param UserCreateRequest $request
     * @return UserCreateResponse
     */
    public function handle(UserCreateRequest $request);
}
```

ユーザを作成する処理の interface です。  
メソッドとそのメソッドが受け取るデータと返却するデータを定義しているだけです。

なお、次のように戻り値のない形こそがクリーンアーキテクチャ本来の形です。

```php
interface UserCreateUseCaseInterface
{
    /**
     * @param UserCreateRequest $request
     * @return void
     */
    public function handle(UserCreateRequest $request);
}
```

サンプルコードが戻り値のあるメソッドになっているのは、一部改変の影響です。  
どうしてこのような形になっているのかについては Controllers の節で詳しく解説します。

さて UserCreateUseCaseInterface の解説に戻りましょう。  
このオブジェクトの `handle` というメソッド名は任意の名前でも問題ないです。  
今回のようにメソッドインジェクションのような便利な機能があれば、メソッド名は interface ごとに異なっても問題ないでしょう。  
（記事：実践クリーンアーキテクチャでは Bus パターンという手段を取ったのでここは合わせる必要がありました。）

ユーザを作成したいとき、クライアントは `UserCreateUseCaseInterface->handle` を呼び出すことになります。

### packages\\Domain\\Application\\User\\UserCreateInteractor

```php
class UserCreateInteractor implements UserCreateUseCaseInterface
{
    /**
     * @var UserRepositoryInterface
     */
    private $userRepository;

    /**
     * UserCreateInteractor constructor.
     * @param UserRepositoryInterface $userRepository
     */
    public function __construct(UserRepositoryInterface $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    /**
     * @param UserCreateRequest $request
     * @return UserCreateResponse
     */
    public function handle(UserCreateRequest $request)
    {
        $userId = new UserId(uniqid());
        $createdUser = new User($userId, $request->getName());
        $this->userRepository->save($createdUser);

        return new UserCreateResponse($userId->getValue());
    }
}
```

`UserCreateUseCaseInterface` を実装したオブジェクトです。

`UserCreateInteractor` はエンティティを協調させ、ユースケースを達成するスクリプトを記述します。  
具体的には与えられたパラメータからモデル（User）を生成して、GateWays に所属するリポジトリにデータ永続化の依頼を行います。

なお、繰り返しになりますが、本来であれば戻り値がないのが正しい形です。  
後ほど詳しくお話ししますので、今段階ではここに改変ポイントの余波があるとご認識ください。

### packages\\MockInteractor\\User\\MockUserCreateInteractor

```php
class MockUserCreateInteractor implements UserCreateUseCaseInterface
{

    /**
     * @param UserCreateRequest $request
     * @return UserCreateResponse
     */
    public function handle(UserCreateRequest $request)
    {
        return new UserCreateResponse('test-id');
    }
}
```

先ほどの UserCreateInteractor はいうなれば本番用のオブジェクトです。  
対してこちらのオブジェクトは名前に Mock という言葉がつくとおり、擬似的な仮モジュールです。  
実装を確認してもそれが見て取れるのではないでしょうか。

なぜこのようなオブジェクトを用意しているのかというと、それはひとえにテスト用です。

たとえばフロントのためのテストを考えてみましょう。  
フロントでテストする際にバックエンドから必要なデータを返す時、本番用オブジェクト UserCreateInteractor はリポジトリを利用しているので、リポジトリを操作すれば意図するデータを受け取ることができます。  
言い換えれば、テストするために必要なデータを作るという作業が必要です。  
それは少し、あるいはとても面倒な作業に思えませんか。

また、バックエンドで、ある特定条件下における例外が発生した時のケースをテストしたいときはどうでしょう。  
テストを行うには例外が発生する、すなわち特定条件を満たしたデータを準備する必要があります。  
それは得てして難しい作業になるでしょう。  
整合性のある不正なデータというのは総じて用意するのが難しいものです。

こういったとき、この擬似オブジェクトは役に立ちます。  
もしもデータを準備するのがとても手間な例外が発生したときの動作を確認したいときは、次のように送出したい例外をそのまま発生させるだけでよいのです。

```php
class MockUserCreateInteractor implements UserCreateUseCaseInterface
{

    /**
     * @param UserCreateRequest $request
     * @return UserCreateResponse
     */
    public function handle(UserCreateRequest $request)
    {
        throw new ComplexException();
    }
}
```

もちろん、本番用オブジェクトの最初の行で同じことをすれば結果は同じなのですが……賢明なみなさんのことですから、なるべくなら本番用のロジックを頻繁に触りたいとは思わないことでしょう。

## Controllers

[![controllers.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/d7237692-3015-16e9-52b8-cfcb4c1ad518.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Fd7237692-3015-16e9-52b8-cfcb4c1ad518.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=82af9d7724256c8a80e00778c20495bc)  
Controllers は入力をアプリケーション（UseCases）が要求する形に変更して伝えるのが役目です。

たとえ話をするとゲームのコントローラなどはよい例になります。  
ゲームコントローラはボタンを押したら、その圧力をそのままゲーム機に伝えるわけではありません。コントローラはゲーム機本体が分かる信号に変換してゲーム機に送りますよ。  
言い換えると「ボタンを押す」という入力をゲーム機が要求する形に変換して伝えているのです。

たとえ話でなく、具体的な話も欲しいところですね。MVC フレームワーク上での実装の話をしましょう。  
わかりやすいケースとして想定するのは MVC フレームワークの都合でフロントとバックエンドの通信の都合上 DateTime などの日付型の値をリクエストとして送ることができないケースを想像してください。  
こういったときフロントからの入力を文字列で伝えることはあるでしょう。  
このときアプリケーション（UseCases）が要求すべきは文字列型でしょうか。  
もちろん違います。  
アプリケーションはなるべくであれば日付型が欲しいです。文字列にすると余計なチェックが必要になってしまいます。可能であれば、型の力を借りてデータの範囲を狭めたいところです。  
こういったケースの際にコントローラが一仕事します。  
つまり、コントローラはリクエストから得た文字列のデータを日付型に変換するのです。  
アプリケーションが要求する形に入力を変換し「適合」させる。コントローラがアダプターと呼ばれる所以はここにあります。

さぁ、具体的な実装を見ていきましょう。  
Controllers に含まれるモジュールは次のモジュールです。

- App\\Http\\Controllers\\UserController

### App\\Http\\Controllers\\UserController

```php
class UserController extends BaseController
{
    public function index(UserGetListUseCaseInterface $interactor)
    {
        $request = new UserGetListRequest(1, 10);
        $response = $interactor->handle($request);

        $users = array_map(
            function ($x) {
                return new UserViewModel($x->id, $x->name);
            },
            $response->users
        );
        $viewModel = new UserIndexViewModel($users);

        return view('user.index', compact('viewModel'));
    }

    public function create(UserCreateUseCaseInterface $interactor, Request $request)
    {
        $name = $request->input('name');
        $request = new UserCreateRequest($name);
        $response = $interactor->handle($request);

        $viewModel = new UserCreateViewModel($response->getCreatedUserId(), $name);
        return view('user.create', compact('viewModel'));
    }
}
```

標準的な MVC のコントローラです。

各アクションは後述のユースケースの interface を `$interactor` という変数名でメソッドインジェクションしています。  
特に `create` アクションを眺めると `$interactor` が要求する形に変換しているのがみてとれます。

```php
$name = $request->input('name');
$request = new UserCreateRequest($name);
$response = $interactor->handle($request);
```

さて、きっと気になっていたであろう改変ポイントがここにあります。（ここだけなので堪えてご覧いただけると嬉しいです。）  
この節の冒頭でクリーンアーキテクチャの Controllers は基本的に入力に関わる適合処理を行うとお話ししました。  
にもかかわらずこのコードは入力して戻ってきた結果を、あまつさえ View 用に変換を行い、あろうことか View に伝えることまでしてしまっています。これはとんでもない越権行為です。  
コントローラのコードは本来であれば次のコードになってしかるべきです。

```php
public function create(UserCreateUseCaseInterface $interactor, Request $request)
{
    $name = $request->input('name');
    $request = new UserCreateRequest($name);
    $response = $interactor->handle($request);
}
```

表示のための変換処理がすべて無くなり、戻り値がなくなりました。  
では、このときどうやって表示処理を行うのかというと、次のように UseCases の実体の Interactor で Presenter を呼び出すことにより、表示処理へ結果データを通知するのです。  
ここで問題となるのが Web の Http 通信の大前提がリクエストとレスポンスとで一対だということです。大抵の MVC フレームワークもその前提で作られているので、リクエストを受けるとレスポンスを戻すようなコードを強制されます。  
つまり、戻り値がないこの形を踏襲できるような仕組みがないのですね。  
結果、表示のためのデータを作成し、戻すという処理を Contollers が行っています。

本来、表示用のデータ変換を行うのは図で言う Presenters の役目です。  
[![presenters.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/aa85eb79-4b54-a6e8-0f83-de9a3405b8b1.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Faa85eb79-4b54-a6e8-0f83-de9a3405b8b1.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1073010484b503a508c8f86b26da148c)  
とはいえ Controllers に目をやると同じレイヤー（緑色）にいます。  
[![controllers.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/0e87fc60-56c8-8fd4-4fc7-5937539da091.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F0e87fc60-56c8-8fd4-4fc7-5937539da091.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ea336df892c95b156ca5cf078e6845de)  
この緑のレイヤーは Interface Adapters となっております。  
[![clean_layer.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/9922e388-cdc0-76e6-2a2c-dd891366202d.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F9922e388-cdc0-76e6-2a2c-dd891366202d.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6909d152f41eeb5ea5ed187a67ca35ba)  
このレイヤーにおける責務で考えると同じオブジェクトに入力の変換処理と結果データの変換処理が同居していても、ぎりぎり納得できるのではないかなと考えています。

## Presenter

[![presenters.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/901897c4-30d3-136f-e5f6-035da1a4928f.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F901897c4-30d3-136f-e5f6-035da1a4928f.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f82e8459cd8b8af8c87b5541fbb70671)  
すでに解説のあったとおり、この項目のモジュールはサンプルコードには存在していません。  
もしもコードを書くとしたら次のコードになるでしょう。

```php
interface UserCreatePresenterInterface
{
    public function output(UserCreateResponse $result);
}

class UserCreatePresenter implements UserCreatePresenterInterface
{
    public function output(UserCreateResponse $result)
    {
        $model = new UserCreateViewModel($result->getCreatedUserId(), $result->getUserName());
        NotifySystem::notify($model); // このモジュールは適当なものです。実際はどのようなモジュールになるかは仕組み次第です。
    }
}
```

これを利用する UserCreateInteractor は次のように変更されます。

```php
class UserCreateInteractor implements UserCreateUseCaseInterface
{
    /**
     * @var UserRepositoryInterface
     */
    private $userRepository;
    /**
     * @var UserCreatePresenterInterface
     */
    private $presenter;

    /**
     * UserCreateInteractor constructor.
     * @param UserRepositoryInterface $userRepository
     * @param UserCreatePresenterInterface $presenter
     */
    public function __construct(UserRepositoryInterface $userRepository, UserCreatePresenterInterface $presenter)
    {
        $this->userRepository = $userRepository;
        $this->presenter = $presenter;
    }

    /**
     * @param UserCreateRequest $request
     * @return void
     */
    public function handle(UserCreateRequest $request)
    {
        $userId = new UserId(uniqid());
        $createdUser = new User($userId, $request->getName());
        $this->userRepository->save($createdUser);

        $outputData = new UserCreateResponse($userId->getValue())
        $this->presenter->output($outputData);
    }
}
```

Presenter から先、どうやって表示データをユーザに表示するかは仕組みによります。  
Http を利用したときの MVC フレームワークでそれを実現するのは難しいでしょう。（何名かから頂いた情報で Laravel ならいけそうという情報があるので、まとまり次第加筆をしようと考えています。）

## 右下の図

同心円の図の登場人物は確認し終わりました。  
とはいえ、これでクリーンアーキテクチャは完璧！ と素直に行かないのが世の中です。  
ここでいったん、図を確認しなおしてみましょう。

[![clean.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/f9f3dee0-8ea8-8cc9-2f9b-311f238b3db5.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Ff9f3dee0-8ea8-8cc9-2f9b-311f238b3db5.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b0a6ad43f9460b3dae1ea6aeaccda7fa)  
よく見ると同心円以外に図があるのに気づきませんか。そう右下に。  
この右下の図、気になりますよね。  
図形に書かれている文字を読んでみると、チラホラここまでに登場した名前が出ています。

そう。この右下の図もまたクリーンアーキテクチャの重要要素なのです。

とはいえ、「まだあるのか」と辟易したり構える必要はありません。  
右下のこの図は、実はこれまでに出てきた登場人物で説明しきれます。  
つまり別名として紹介されているだけなので、どれがどれにあたるのかを再度確認するだけです。

そう思うと簡単そうに見えてきませんか。

簡単そうに見えてきましたよね。

それでは早速確認していきましょう。

## Controller

[![lower_right_controller.PNG](https://qiita-image-store.s3.amazonaws.com/0/293368/e5db07cb-672b-4929-e9ae-68c5da35751d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Fe5db07cb-672b-4929-e9ae-68c5da35751d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b9d41bb5c053961814f0b789a35c27d5)  
これまで紹介してきた Controller と同一です。  
つまり該当するモジュールはコントローラですね。

- App\\Http\\Controllers\\UserController

必然的に解説はまったく同じになるので省略し、ここでは Controller から伸びる矢印があるということを覚えておきましょう。  
次の節ですぐにこちらの矢印について説明をします。

## UseCaseInputPort

[![lower_right_usecase_inputport.PNG](https://qiita-image-store.s3.amazonaws.com/0/293368/78c82fab-1969-2048-5b45-24bf4c9d0d45.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F78c82fab-1969-2048-5b45-24bf4c9d0d45.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=12cba17d2461ceeccd123d5b5d4f9a3d)  
この図形に対応するのは次のモジュールです。

- packages\\UseCase\\User\\Create\\UserCreateUseCaseInterface
- packages\\UseCase\\User\\GetList\\UserGetUseCaseInterface

UseCases のときに紹介した interface ですね。  
そう interface なのです。  
この観点で改めて図形を確認すると図形の右上にある <I> という字に気づきませんか。  
これは interface を表しています。  
まさに UseCaseInputPort は interface であるというのが図に書かれていたのですね。

また、矢印を注意深く観察すると、二種類あることがわかります。  
この矢印の違いは意図したものです。

ひとつ目の黒い矢印。これは依存の矢印です。  
つまり、根本のオブジェクトは矢印の先のオブジェクトに依存している、ということを示しています。  
[![lower_right_controller_usecase_inputport.PNG](https://qiita-image-store.s3.amazonaws.com/0/293368/1222b0c9-2343-c964-cf38-8ef004bae96d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F1222b0c9-2343-c964-cf38-8ef004bae96d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8167b619669a02c283dd5ef29cc609f8)  
Controller は UseCaseInputPort に依存＝利用していたでしょうか。  
コードを確認してみましょう。

```php
class UserController extends BaseController
{
    /* 
     * 省略 
     */

    public function create(UserCreateUseCaseInterface $interactor, Request $request)
    {
        $name = $request->input('name');
        $request = new UserCreateRequest($name);
        $response = $interactor->handle($request); // UserCreateUseCaseInterface に依存しています！

        $viewModel = new UserCreateViewModel($response->getCreatedUserId(), $name);
        return view('user.create', compact('viewModel'));
    }
}
```

正しく図のとおりになっています。  
そうすると、もうひとつの白抜き矢印は何になるでしょうか。

UML で白抜き矢印は汎化を示します。つまり、抽象と実装の関係ですね。  
矢印の先が抽象で、元が実装クラスです。  
UseCaseInputPort に対して白抜き矢印を伸ばしているオブジェクトは次に解説する UseCaseInteractor です。

## UseCaseInteractor

[![lower_right_usecase_interactor.PNG](https://qiita-image-store.s3.amazonaws.com/0/293368/ed923288-14d6-bb23-f4b7-ba6a87dab539.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Fed923288-14d6-bb23-f4b7-ba6a87dab539.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=302e510ddb2537b805cd05affb84bd64)  
次の一覧が UseCaseInteractor のモジュールたちです。

- ユーザ作成処理
- packages\\Domain\\Application\\User\\UserCreateInteractor
- packages\\MockInteractor\\User\\MockUserCreateInteractor
- ユーザ取得処理
- packages\\Domain\\Application\\User\\UserGetListInteractor
- packages\\MockInteractor\\User\\MockUserGetInteractor

UseCaseInputPort の項で説明したとおり、図に記載されている UML のような矢印に従うと、このモジュールたちは UseCaseInputPort の実装クラスであるはずです。  
[![lower_right_usecase_inputport_usecase_interactor.PNG](https://qiita-image-store.s3.amazonaws.com/0/293368/81c26daa-7dce-7a13-d7de-2793a21be2e5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F81c26daa-7dce-7a13-d7de-2793a21be2e5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=db2683f80334eea9d2d1d826a1ca9960)  
それぞれのクラスの宣言部分を確認してみましょう。

```php
class UserCreateInteractor implements UserCreateUseCaseInterface
```

```php
class MockUserCreateInteractor implements UserCreateUseCaseInterface
```

```php
class UserGetListInteractor implements UserGetListUseCaseInterface
```

```php
class MockUserGetInteractor implements UserGetListUseCaseInterface
```

どのクラスを確認しても UseCaseInputPort にあたるモジュールを実装しているので、これは間違っていなかったようです。

## UseCaseOutputPort と Presenter

[![lower_right_usecase_outputport_presenter.PNG](https://qiita-image-store.s3.amazonaws.com/0/293368/4afcb8df-3ab5-09e0-d8d4-f9f208373c38.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F4afcb8df-3ab5-09e0-d8d4-f9f208373c38.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=22ae0e24535e99567b38edbc622bdf87)  
今回 Presenter を利用していないのでサンプルコードにこれらは現れてきません。

しかし、実はここまでの解説で実装コード自体はでてきています。

そう Presenter の項で説明したコードですね。  
具体的に確認したい場合は Presenter の項のコードをご覧ください。  
ここに出てくる UserCreatePresenterInterface が UseCaseOutputPort で UserCreatePresenter が Presenter にあたります。  
汎化や依存の矢印も正しくあらわされているはずです。

## Flow of control

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/f817afca-4188-bb59-0f73-ca510ac494f5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Ff817afca-4188-bb59-0f73-ca510ac494f5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=50e80b6e26d5b982750dbdff7f692c82)  
図を再度確認して矢印に着目すると、まだ解説されていない矢印があることに気づきます。  
この Flow of control と書かれた矢印は、英語の意味そのまま「処理の流れ」を表しています。

どういうことかというと、これは処理を順番に追うとわかります。

まずは Controller から見てみましょう。

```php
class UserController extends BaseController
{
    /* 
     * 省略 
     */

    public function create(UserCreateUseCaseInterface $interactor, Request $request)
    {
        $name = $request->input('name');
        $request = new UserCreateRequest($name);
        $response = $interactor->handle($request); // UserCreateUseCaseInterface に依存しています！

        $viewModel = new UserCreateViewModel($response->getCreatedUserId(), $name);
        return view('user.create', compact('viewModel'));
    }
}
```

Controller は UserCreateUseCaseInterface を経由してその実体の UserCreateInteractor に処理を引き渡します。

```php
// Presenter を利用している Version です
class UserCreateInteractor implements UserCreateUseCaseInterface
{
    /**
     * @var UserRepositoryInterface
     */
    private $userRepository;
    /**
     * @var UserCreatePresenterInterface
     */
    private $presenter;

    /**
     * UserCreateInteractor constructor.
     * @param UserRepositoryInterface $userRepository
     * @param UserCreatePresenterInterface $presenter
     */
    public function __construct(UserRepositoryInterface $userRepository, UserCreatePresenterInterface $presenter)
    {
        $this->userRepository = $userRepository;
        $this->presenter = $presenter;
    }

    /**
     * @param UserCreateRequest $request
     * @return void
     */
    public function handle(UserCreateRequest $request)
    {
        $userId = new UserId(uniqid());
        $createdUser = new User($userId, $request->getName());
        $this->userRepository->save($createdUser);

        $outputData = new UserCreateResponse($userId->getValue())
        $this->presenter->output($outputData);
    }
}
```

UserCreateInteractor は UserCreatePresenterInterface を呼び出すことで UserCreatePresenter に処理を引き渡します。

ここまでの内容を制止すると次の順序になります。

1. UserController が UserCreateUseCaseInterface を呼び出す
2. UserCreateUseCaseInterface の実体の UserCreateInteractor に処理が移る
3. UserCreateInteractor が UserCreatePresenterInterface を呼び出す
4. UserCreatePresenterInterface の実体の UserCreatePresenter に処理が移る

この流れで抽象を抜きにすると次の順序で実行されます。

1. UserController
2. UserCreateInteractor
3. UserCreatePresenter

もう一度 Flow of control を見てみましょう。  
[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/f817afca-4188-bb59-0f73-ca510ac494f5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Ff817afca-4188-bb59-0f73-ca510ac494f5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=50e80b6e26d5b982750dbdff7f692c82)  
矢印は Controller → UseCaseInteractor → Presenter という動きをしています。  
それぞれサンプルコードのクラスに当て込むと次のようになります。

1. Controller: UserController
2. USeCaseInteractor: UserCreateInteractor
3. Presenter: UserCreatePresenter

UserController → UserCreateInteractor → UserCreatePresenter という処理の実際の動きと一致していますね。  
Flow of control は interface などを加味しない実際の処理が走るモジュールを追った図になっていたのです。

## もうひとつの図

やっとクリーンアーキテクチャの象徴的な図については制覇しました。本当にここまでお疲れ様でした。  
ここでひとつお祝いをしたいところではありますが、残念なお知らせがあります。  
実はもうひとつ、図があるのです。

ケーキは嘘ではありません。でももう少し後です。  
次の図で最後ですのでもうひとふんばり頑張っていきましょう。

新しい図はこちらになります。  
[![classes.jpg](https://qiita-image-store.s3.amazonaws.com/0/293368/905ab8b4-4c6b-5fd8-0446-f1473be5cc00.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F905ab8b4-4c6b-5fd8-0446-f1473be5cc00.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9ca9858f243ff3a115c11b8e5d04d76e)  
今度の図はこれまでの図と比べると具体的なものになっていますね。  
矢印については [右下の図](https://qiita.com/nrslib/items/#%E5%8F%B3%E4%B8%8B%E3%81%AE%E5%9B%B3) のときと同じように、依存と汎化についてを表しています。

この図だけを見ると多くの用語に囲まれるので少し怖く感じるかもしれませんが、実はこの図にはこれまでの登場人物が多く再登場しています。  
問題なのはそれらの名前が変わってたりするということです。

構える必要はありません。ここまで見聞きしたものたちが大半です。  
左上から順番に解説していきましょう。

## Controller

[![classes_controller.PNG](https://qiita-image-store.s3.amazonaws.com/0/293368/7335f3d5-89d0-1186-8d45-dc331dbf89ce.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F7335f3d5-89d0-1186-8d45-dc331dbf89ce.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=263b334da8f39615bf7e52765f4301eb)  
またも Controller です。  
もちろんこれは次のモジュールです。

- App\\Http\\Controllers\\UserController

特筆すべき点はありません。このあとまだまだ用語が控えています。さっさと次へ行きましょう。

## InputData

[![classes_inputdata.PNG](https://qiita-image-store.s3.amazonaws.com/0/293368/ee99c85b-62a8-c0d3-96ec-3e0c23869a72.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Fee99c85b-62a8-c0d3-96ec-3e0c23869a72.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=df6ab85d6610af0151e89452b35ff78d)  
これはここまでに解説はされていない登場人物です。

右上の DS は DataStructure の略です。つまりデータ構造体。  
コードとしては次のモジュールがこれにあたります。

- packages\\UseCase\\User\\Create\\UserCreateRequest
- packages\\UseCase\\User\\GetList\\UserGetListRequest

コードを確認するとかなり単純な DTO（DataTransferObject, レイヤーをまたぐようにデータをやり取りする際に利用するデータ運搬専用ブジェクト）です。

```php
class UserCreateRequest
{
    /**
     * @var string
     */
    private $name;

    /**
     * UserCreateRequest constructor.
     * @param string $name
     */
    public function __construct(string $name)
    {
        $this->name = $name;
    }

    /**
     * @return string
     */
    public function getName(): string
    {
        return $this->name;
    }
}
```

InputData という名のとおり、UseCase に渡す入力データになっています。

## InputBoundary

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/a426cb24-1a04-e47c-6b6d-902ad994d70b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Fa426cb24-1a04-e47c-6b6d-902ad994d70b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d8da8aae5aa9ec32ec432c40e3a169de)  
これは UseCaseInputPort と同一です。  
どうして名前を変えたのかはわかりません。

モジュールは次のふたつですね。

- packages\\UseCase\\User\\Create\\UserCreateUseCaseInterface
- packages\\UseCase\\User\\GetList\\UserGetUseCaseInterface

## UseCaseInteractor

[![classes_usecaseinteractor.PNG](https://qiita-image-store.s3.amazonaws.com/0/293368/d2d69b50-36c8-afb5-7c64-434544f587c1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Fd2d69b50-36c8-afb5-7c64-434544f587c1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2c8a07e0e24c3ece12001fd139c8a271)  
これは同じ名前で登場していました。  
InputBoundary (UseCaseInputPort) の実装クラスです。  
つまり次のモジュールたちのことです。

- ユーザ作成処理
- packages\\Domain\\Application\\User\\UserCreateInteractor
- packages\\MockInteractor\\User\\MockUserCreateInteractor
- ユーザ取得処理
- packages\\Domain\\Application\\User\\UserGetListInteractor
- packages\\MockInteractor\\User\\MockUserGetInteractor

## OutputBoundary

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/3172b1f8-439e-4130-5718-c46e57563b87.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F3172b1f8-439e-4130-5718-c46e57563b87.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f14355a68ff2729cbb78e8f57078bc75)  
これは Presenter です。  
今回のサンプルコードには存在していないというお話でした。

## OutputData

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/d0e07b30-7f0b-6452-55d8-c39bbd19adfa.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Fd0e07b30-7f0b-6452-55d8-c39bbd19adfa.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=28a1b2c9f736e66911451fbd31e30547)  
またも DataStructure です。  
こちらは Input に対する Output なので次のコードがこれにあたります。

- packages\\UseCase\\User\\Create\\UserCreateResponse
- packages\\UseCase\\User\\GetList\\UserGetListResponse

また出力用データの一種の次のモジュールもこれに含むことは許されるでしょう。

- packages\\User\\Commons\\UserModel

## Presenter

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/fcf7b6fb-2cfc-3e1b-3f04-4747f201f720.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Ffcf7b6fb-2cfc-3e1b-3f04-4747f201f720.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a5e86c66ae6608b9f97351b570571f5c)  
もちろん Presenter は利用されていません。

## ViewModel

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/3a822728-3a40-0fd3-5ef7-bd622ca469cd.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F3a822728-3a40-0fd3-5ef7-bd622ca469cd.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=5289dd2ea89ffcbecfe2612a88c9fafd)  
いわゆる MVVM の ViewModel とは異なります。  
DataStructure になっているのでこれは DTO として扱われているようにみえます。  
サンプルコードでは次のモジュールがこちらにあたります。

- App\\Http\\Models\\User\\Create\\UserCreateViewModel
- App\\Http\\Models\\User\\Commons\\UserViewModel\\UserIndexViewModel

また ViewModel の構成要素となる次のモジュールもこれに含んで差し支えないでしょう。

- App\\Http\\Models\\User\\Commons\\UserViewModel

## View

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/42882210-4bef-8fc5-5fc0-e19b3cf10ee2.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F42882210-4bef-8fc5-5fc0-e19b3cf10ee2.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=907cbc588986fa67db2782dd9b423acb)  
いわゆるビューです。  
仮組みになっているのですが次のビューファイルがこれにあたります。

- resources\\views\\user\\create.blade.php
- resources\\views\\user\\index.blade.php

## DataAccessInterface

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/bbc198e0-3d2b-586b-cf58-ec27367ce8c6.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Fbbc198e0-3d2b-586b-cf58-ec27367ce8c6.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=cf3e8d7d621dd9935a5037fabbbb3bb7)  
<I> マークから分かるとおりこれは interface で、具体的には [Gateways](https://qiita.com/nrslib/items/Gateways) で登場したリポジトリの interface です。  
よって次のモジュールが DataAccessInterface です。

- packages\\Domain\\Domain\\User\\UserRepositoryInterface

## DataAccess

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/d94a5479-32fb-30d0-6f08-0c89ba56f465.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2Fd94a5479-32fb-30d0-6f08-0c89ba56f465.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b964a0bb122aba7da06ffec3b30b5ca4)  
interface があればそれの実装クラスがあります。  
次のリポジトリが DataAccess です。

- packages\\Infrastructure\\User\\UserRepository
- packages\\InMemoryInfrastructure\\User\\InMemoryUserRepository

## Database

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/6f87a35c-a761-6dbb-f9cb-5c7465311100.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F6f87a35c-a761-6dbb-f9cb-5c7465311100.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b452748209f205980a2c6d2aa0040cd5)  
急にここだけ特定の技術になりました。  
MySQL などを指しています。

## Entities

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/293368/90c9553f-fe35-84a3-9993-e4c2c027581e.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F293368%2F90c9553f-fe35-84a3-9993-e4c2c027581e.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ec66917932405212362585faba6ce08c)  
冒頭の Entities と同じです。

- packages\\Domain\\Domain

## 何がうれしいの？

このプロジェクトの構成は非常に大げさにみえることもあるでしょう。  
大掛かりな設計を行う理由付けとしてこれのメリットについて語らなくてはいけません。

さて、クリーンアーキテクチャの最大のメリットについては原文でも明記されていますが、どのレイヤーにおいてもテストができることではないかと考えます。

たとえばフロントの挙動を確認したいときは MockUserGetListInteractor や MockUserCreateInteractor を利用することで、お好みのデータを正常な動作として引き渡すことが簡単にできます。  
もちろん UserGetListInteractor の中身を書き換えれば同じことが可能なのですが、完全 Mock で動作する設定ファイルを用意して手軽に実行することが可能になったりします。

ついで、バックエンドのロジックもテストできることは特筆すべき点です。

ユースケースにおけるテストは次のように記述できます。

```php
class UserCreateInteractorTest extends TestCase
{
    /**
     * テスト：正常にユーザを生成する
     *
     * @return void
     */
    public function testValidCreate()
    {
        $repository = new InMemoryUserRepository();
        $interactor = new UserCreateInteractor($repository);
        $request = new UserCreateRequest('test-name');
        $response = $interactor->handle($request);

        $this->assertNotNull($response);
        $this->assertNotNull($response->getCreatedUserId());

        $userId = new UserId($response->getCreatedUserId());
        $saved = $repository->find($userId);

        $this->assertEquals($saved->getName(), 'test-name');
    }
}
```

UserCreateInteractor の動作は「引数オブジェクトからユーザを生成し、データ永続化以来を行う」というのがその動作です。  
このようにインメモリなモジュールを利用することで単体テストを行うことができます。  
結合テストは別途実施する必要がありますが、単体テストをするためにデータベースを用意し、テーブルを作成してデータを投入しなくてはいけないという手順はいかにも面倒です。  
クリーンアーキテクチャの構成はデータの永続化や外部連携部分（api など）を抽象化し、ユースケースの実行を気軽に行うことができます。  
これは間違いなくメリットです。

テストが気軽に実行でき、細かい動作確認ができることは開発者の自信につながります。  
そう、「動かしてないけどたぶん動くだろう」という希望的観測で諦めていたあのコードを「ロジックを通したから動く」という確証（データベースが死んでいたりすると動きませんが）を持ったコードに変えることができるのです。

## まとめ

以上でコードの解説は終わりです。  
最初は難しく見えたサンプルコードの構成もわかってしまえば単純なものに見えるようになったのではないでしょうか。

サンプルコードで実現しているのは単純なユースケースです。  
にもかかわらずコード量はかなり膨大です。  
ただ単純にユーザを作成したり、一覧を取得するためだけにこれだけのコードを書かなければならないのかと辟易もするでしょう。  
それでもそれをするだけの価値があると考えています。

なぜか。

ソフトウェアは必ず変化します。  
変化しないソフトウェアはもはやハードウェアです。  
変化に対応できないソフトウェアはいつか何かしらの不便さや窮屈さを利用者に押し付けるでしょう。  
それでは意味がないのです。  
不便さや窮屈さを解決するために存在するのがソフトウェアです。

変化に対応するには、関心によってモジュールを分離して影響範囲を狭めると同時に、伝播すべき箇所には変更が伝わるようにすることで、手を入れやすくすることが不可欠です。  
クリーンアーキテクチャはこれをうまくやってのけます。

冒頭にお話したとおり、理解がゴールです。  
もしもここまでの内容である程度の理解まで落とし込むことができたのであれば、それは素晴らしいではないかと考えます。

あくまでのこの記事でお出しした実装は一例にすぎません。つまり忠実に再現する必要はありません。  
この形にすることで何が達成できたのか。得られたメリットは何なのか。これらの実装パターンのどれを取り入れることがご自身のプロダクトによい影響を与えるのか。  
開発時における選択肢が増えたようであれば幸いです。

## おまけ

妥協せず Presenter を使う形にしてみました。  
[https://qiita.com/nrslib/items/eaf39be65b2ebe5ccf08](https://qiita.com/nrslib/items/eaf39be65b2ebe5ccf08)

クリーンアーキテクチャよりも軽量で無理なく導入しやすいアプリケーションアーキテクチャパターンを考案しました。  
[https://nrslib.com/adop/](https://nrslib.com/adop/)

[21](https://qiita.com/nrslib/items/#comments)

[904](https://qiita.com/nrslib/items/aa49d10dd2bcb3110f22/likers)

865

ストックを更新しました