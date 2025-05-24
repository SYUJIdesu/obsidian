---
title: Laravel アンチパターンと対策まとめ
source: https://qiita.com/ucan-lab/items/bdef3bc513c779116834
author:
  - "[[ucan-lab]]"
published: 2023-12-01
created: 2025-05-22
description: Laravel Advent Calendar 2023 1日目の記事です。まだ枠が空いてるのでよければ参加して盛り上げてください🙌ぽえむ最近はQiitaを書く習慣がなくなってしまいました。…
tags:
  - web
  - Laravel
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

[Laravel Advent Calendar 2023](https://qiita.com/advent-calendar/2023/laravel) 1日目の記事です。  
まだ枠が空いてるのでよければ参加して盛り上げてください🙌

## ぽえむ

最近はQiitaを書く習慣がなくなってしまいました。  
年間100記事近くを書いていた頃は書きたいネタがどんどん出てきてしかたなかったのですが、その習慣がなくなると何書けばいいのかわからなくなって困りました。

ここ数年はLaravel案件に触れる機会もめっきり減ってしまったのもありますが、  
副業等でたまに触れるのでその時にこれってアンチパターンだよなと思うことが溜まってきたので記憶を頼りにまとめていきたいと思います。

## マイグレーション編

### 開発中にどんどん新しいマイグレーションファイルを追加してしまう

開発中は気軽にデータベースのリセットが行えるので、マイグレーションファイルは追加するのではなくガッツリ書き換えていきましょう。  
マイグレーションファイルが増えるとマイグレーションの実行がどんどん遅くなります。

```shell
$ php artisan migrate:fresh --seed
```

このコマンドでマイグレーションのやり直しとシーダーの実行をしてくれます。  
運用が始まったら、マイグレーションファイルは追加していきましょう。

### timestamps を使ってしまう

MySQLのtimestamp型の2038年問題があります。

```text
> show variables like '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | UTC    |
| time_zone        | SYSTEM |
+------------------+--------+

> CREATE TABLE test (id int, time timestamp);

> INSERT INTO test VALUES(1, '2038-01-19 03:14:07');
Query OK, 1 row affected (0.00 sec)

> INSERT INTO test VALUES(1, '2038-01-19 03:14:08');
ERROR 1292 (22007): Incorrect datetime value: '2038-01-19 03:14:08' for column 'time' at row 1
```

タイムゾーンがUTCの時、 `2038-01-19 03:14:08` 以上の値を入れるとエラーになります。  
これから15年以上続くシステムの場合、この問題が付いてくるので早い段階でdatetime型を使うことを推奨します。

```php
public function up(): void
    {
        Schema::create('{{ table }}', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }
```

デフォルトがtimestampsなのなぁぜなぁぜ？

```php
public function up(): void
    {
        Schema::create('{{ table }}', function (Blueprint $table) {
            $table->id();
            $table->datetimes();
        });
    }
```

`datetimes()` は用意されているのでこちらを使用します。

```shell
$ php artisan make:migration create_hogehoge_table
```

で生成されるファイルを直したい場合は下記のコマンドを実行します。

```shell
$ mkdir ./stubs
$ sed 's/timestamps/datetimes/' ./vendor/laravel/framework/src/Illuminate/Database/Migrations/stubs/migration.create.stub > ./stubs/migration.create.stub
```

`php artisan stub:publish` スタブファイルを生成して、中身を変更しても良いです(変更してないスタブファイルは削除した方が良いです)

## モデル編

### filterやmapで絞り込みしてしまう

[株式会社ゆめみさん](https://x.com/yumemiinc) さんから良いお題の出されていました。

```php
<?php

class UserController
{
    // 特定の国に住んでるユーザー一覧を取得
    public function searchByCountry(Request $request): Response
    {
        $users = User::all();

        $users = $users->filter(function(User $user) use ($request) {
            $userCountry = UserCountry::query()
                ->where('user_id', $user->id)
                ->where('country_id', $request->input('country_id'))
                ->get();

            return isset($userCountry);
        });

        $country = Country::find($request->input('country_id'));

        $result = [];
        foreach ($users as $user) {
            $result[] = [
                'id' => $user->id,
                'name' => $user->name,
                'country_id' => $country->id,
                'country_name' => $country->name,
            ];
        }

        return response()->json($result);
    }
}
```

`User::all();` ユーザーの件数次第ではメモリに乗らなかったり、速度低下が懸念があります。

データベースを実行した後にfilterやmapで絞り込んだりするのではなく、where句を指定して取得するデータ件数を絞り込みましょう。  
それでも対象となるデータが多くなる場合は [カーソル](https://readouble.com/laravel/10.x/ja/eloquent.html#cursors) や [ペジネーション  
](https://readouble.com/laravel/10.x/ja/pagination.html)を使いましょう。

```php
<?php

class UserController
{
    // 特定の国に住んでるユーザー一覧を取得
    public function searchByCountry(Request $request): Response
    {
        $countryId = $request->input('country_id');

        $users = User::with(['countries' => fn (Builder $query) => $query->select('countries.id', 'countries.name')])
            ->whereHas('countries', fn (Builder $query) => $query->whereKey($countryId))
            ->select('users.id', 'users.name')
            ->get();

        $result = $users->map(fn (User $user) => [
            'id' => $user->id,
            'name' => $user->name,
            'country_id' => $user->countries->first()->id,
            'country_name' => $user->countries->first()->name,
        ]);

        return response()->json($result);
    }
}
```

- with 句を使うとリレーション先のデータを取得する際にN+1問題を防ぎます
	- ここに関しては元のコードのように1つのモデルインスタンスを取得しても良いです
	- こういう書き方もできる例を載せたかっただけです
- whereHas と whereKey で多対多のリレーションのidで絞り込みできます
- select で使用するカラムのみを定して省メモリ、高速化ができます

### なんとなくSoftDeletesを使ってしまう

Laravelには [Illuminate/Database/Eloquent/SoftDeletes](https://laravel.com/api/10.x/Illuminate/Database/Eloquent/SoftDeletes.html) 論理削除機能が用意されており、モデルクラスで `use SoftDeletes;` すると気軽に使用できます。

app/models/Example.php

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use SoftDeletes;

    // ...
}
```

対策:  
エンドユーザにはデータを表示させたくないが、実際のデータは消したくない。  
「削除した」データを参照したり、誤った操作をなかったことにしたいということは「削除」ではなく「凍結」、「退会」など具体的な状態の名前を付けてステータス管理しましょう。  
本当に削除が必要な場合は物理削除しましょう。

ただし、DELETEはUPDATEより実行コストが高いことが多いです。  
高トラフィックなサイトではDELETE(物理削除)が難しい場合もあります。

その時でも論理削除ではなく具体的な名前を付けるようにしたり履歴テーブルに移すなどして欲しいです。  
複数テーブルを参照する複雑なSQLを書く時に各テーブルの論理削除を意識してSQL書くのはとてもしんどかったです。  
おすすめのスライド: [https://www.slideshare.net/t\_wada/ronsakucasual](https://www.slideshare.net/t_wada/ronsakucasual)

## 日付編

### DateTime, Carbon クラスを使ってしまう

結論: DateTimeImmutable, CarbonImmutable を使う

- [DateTime](https://www.php.net/manual/ja/class.datetime.php)
- [DateTimeImmutable](https://www.php.net/manual/ja/class.datetimeimmutable.php)
- [Carbon](https://github.com/briannesbitt/Carbon/blob/master/src/Carbon/Carbon.php)
- [CarbonImmutable](https://github.com/briannesbitt/Carbon/blob/master/src/Carbon/CarbonImmutable.php)

```text
$date = DateTime::createFromFormat('Y-m-d', '2023-12-01'); // 2023-12-01 00:00:00
$date->modify('+1 days'); // 2023-12-02 00:00:00
$date->modify('+1 days'); // 2023-12-03 00:00:00 (インスタンスの中身が変更されてしまっている)

$carbon = Carbon\Carbon::create(2023, 12, 1); // 2023-12-01 00:00:00
$carbon->addDay(); // 2023-12-02 00:00:00
$carbon->addDay(); // 2023-12-03 00:00:00 (インスタンスの中身が変更されてしまっている)
```

補足: PHP-CS-FixerのルールにDateTimeをDateTimeImmutableに変換するものがあるので有効化すると良い(既存プロジェクトだと挙動が変わるので導入時は注意)

- [https://mlocati.github.io/php-cs-fixer-configurator/#version:3.38|fixer:date\_time\_immutable](https://mlocati.github.io/php-cs-fixer-configurator/#version:3.38%7Cfixer:date_time_immutable)

### 月末の日付操作に addMonth, subMonth を使ってしまう

```text
$carbon = Carbon\CarbonImmutable::create(2023, 10, 1)->endOfMonth(); // 2023-10-31 23:59:59
$carbon->addMonth(); // 2023-12-01 23:59:59 => 2023-11-30 になって欲しい
$carbon->subMonth(); // 2023-10-01 23:59:59 => 2023-09-30 になって欲しい
```

10月は31日までありますが、9月と11月は31日がないので、 `addMonth()`, `subMonth()` を使うと繰り上がってしまいます。  
本当にきっちり30日後の日付が欲しい場合は良いのですが、月末処理が意図せず月初処理になってしまうのは大変困ります。

結論: `addMonthNoOverflow`, `subMonthNoOverflow` を使う

```text
$carbon = Carbon\CarbonImmutable::create(2023, 10, 1)->endOfMonth(); // 2023-10-31 23:59:59
$carbon->addMonthNoOverflow(); // 2023-11-30 23:59:59
$carbon->subMonthNoOverflow(); // 2023-09-30 23:59:59
```

これで意図した通り、翌月と先月の月末の日付が取得できます。

## フォームリクエスト編

Laravelのフォームリクエストでは認可と検証が行えます。

### ビジネスロジックを書いてしまう

## config編

`app/config` 配下に連想配列を返す配列を作ると簡単にconfigファイルを作れます。

app/config/account.php

```php
<?php

return [
    'status' => [
        'active' => 1,
        'inactive' => 2,
        'leave' => 3,
    ]
];
```

`config('account.status.active')` で簡単に作れます。  
ただし、これを多用してしまうとコードジャンプできないですし、何の型が入ってるかわかりづらいです。

対策:  
php8.1から追加されたEnumクラスを使いましょう。  
php8.1から使えますが、配列のキーに使えなかったり不便なのでphp8.2以降だとより使いやすいです。

account.php

```php
<?php

enum AccountStatus: int
{
    case ACTIVE = 1;
    case INACTIVE = 2;
    case LEAVE = 3;
}

final class Account
{
    private const LABEL = [
        AccountStatus::ACTIVE->value => 'アクティブ',
        AccountStatus::INACTIVE->value => '非アクティブ',
        AccountStatus::LEAVE->value => '退会',
    ];

    public function __construct(
        private readonly string $name,
        private readonly AccountStatus $status,
    ) {
    }

    public function statusLabel(): string
    {
        return self::LABEL[$this->status->value];
    }
}

$account = new Account('ucan', AccountStatus::ACTIVE);
echo $account->statusLabel() . PHP_EOL;
```

```shell
$ docker run --rm -v $(pwd):/app -w /app php:8.1-cli php account.php
=> Fatal error: Constant expression contains invalid operations in /app/account.php on line 12

$ docker run --rm -v $(pwd):/app -w /app php:8.2-cli php account.php
=>アクティブ
```

## 例外編

### すべての例外をExceptionクラスで表現してしまう

`throw new Exception(xxx);` で例外を作ってしまうと例外ハンドリングの柔軟性が下がります。

個別に例外を作っておくと例外ごとにコントローラで表示を出し分けしたり、  
例外ハンドラーでハンドリングしやすくなります。

## アーキテクチャ編

### FatController にしてしまう

全をControllerに記載するとコード量が増えてしまいます。

- 認可, 検証はFormRequestに任せる
- メソッドインジェクションを使う
- シングルアクションコントローラを使う
- DB操作はEloquentに任せる
- 新たにService層を導入しビジネスロジックを任せる
	- 小規模プロジェクトではなくても困らないかも
	- 大規模プロジェクトではクリーンアーキテクチャなどの導入を検討したい

上記のような手法で責務を分けて行くとシンプルなコントローラを作れます。

### 安易にRepositoryパターンを導入してしまう

Repositoryは永続化層の抽象するために用いられます。

- テーブルと1:1でRepositoryを作ってしまう
	- 例: orders, order\_details テーブルがあった場合は OrderRepository, OrderDetailRepository を作ってしまう
		- OrderRepository を作って Order エンティティでやりとりする
- Repositoryの受け渡しをEloquentモデルで行ってしまう
	- EloquentだとRepository外でも簡単にDBへアクセスできてしまう。
		- Repositoryが破綻してしまうので、そうなるなら導入しない方が吉

Repositoryパターンを導入する場合はドメイン層(エンティティ、値オブジェクト)もセットで必要です。  
プロジェクトの規模が小さい場合は導入しない選択肢もアリです。

### このコード、あなたならどんな風にレビューしますか？

[株式会社ゆめみ](https://x.com/yumemiinc) さんから良いお題の出されていました。

```php
<?php

namespace App\Repositories;

use App\Entities\User as Entity;
use App\Models\User;
use App\Models\UserDetail;

class UserRepository
{
    public function __construct(public User $user, public UserDetail $detail) {}

    public function update(int $id, array $request): Entity
    {
        // User Entity は users と user_details の集約
        $userRecord = $this->user->newQuery()->find($id);
        if ($userRecord === null) {
            abort(404);
        }
        if ($request['username'] !== null) {
            $userRecord->username = $request['username'];
        }
        if ($request['email'] !== null) {
            $userRecord->email = $request['email'];
        }
        $userRecord->save();

        $detailRecord = $this->detail->newQuery()->where('user_id', $id)->first();
        if ($detailRecord === null) {
            abort(404);
        }
        if ($request['address'] !== null) {
            $detailRecord->address = $request['address'];
        }
        if ($request['phone_number'] !== null) {
            $detailRecord->phone_number = $request['phone_number'];
        }
        $detailRecord->save();

        return new Entity(
            $userRecord->id, $userRecord->username, $userRecord->email, $detailRecord->address, $detailRecord->phone_number,
        );
    }
}
```

- まずはインターフェースを切りましょう
	- リポジトリでは必須とは言えませんが、インターフェースを使用することで可読性や保守性が向上します。
- コンストラクタでEloquentモデルのDIを止めよう
	- しかも public プロパティになってるので外から変更できてしまってます。
	- そもそもここでDIするメリットがないので普通に `User::find($id)` と使ってあげて良いでしょう
- リポジトリはエンティティを受け取り、エンティティを返します。
	- update で $request の配列を受け取ってるので良くないですね。
	- `update(Entity $entity): void;` で良いです。
		- `$entity` の中身は書き換える必要ないので return もしなくていいですね。
- abort ではなく例外クラスを返す
	- `throw new UserNotFoundException();`
	- コントローラ側でこの例外をキャッチしてabortするのはokです。
- DBトランザクションを張ろう
	- `$detailRecord` が null だったら `$userRecord` だけ更新されちゃいますね。

上記を踏まえてコードを直してみます。

```php
<?php

namespace App\Entities;

interface UserRepository
{
    public function update(User $entity): void;
}
```

まずはインターフェースを作成します。  
名前空間はエンティティと同じ場所に配置します。  
update メソッドは Userエンティティを受け取ります。

```php
<?php

namespace App\Entities;

final readonly class User
{
    public function __construct(
        public int $id,
        public ?string $username,
        public ?string $email,
        public ?string $address,
        public ?string $phone_number,
    ) {}
}
```

- プロパティを値オブジェクト(ValueObject)で型付けるとより良いです。
	- 例えば電話番号を文字列型(string)ではなくPhoneNumber型を作るとドメインの業務を型でわかりやすく示すことができます。

```php
<?php

namespace App\Repositories;

use App\Entities\User as Entity;
use App\Entities\UserNotFoundException;
use App\Entities\UserDetailNotFoundException;
use App\Models\User;
use App\Models\UserDetail;
use Illuminate\Support\Facades\DB;

final readonly class ConcreteUserRepository implements UserRepository
{
    public function update(Entity $entity): void
    {
        DB::transaction(function () use ($entity) {
            // User Entity は users と user_details の集約
            $userRecord = User::find($entity->id);
            if ($userRecord === null) {
                throw new UserNotFoundException("ID {$entity->id} のユーザーが見つかりませんでした。");
            }
            if ($entity->username !== null) {
                $userRecord->username = $entity->username;
            }
            if ($entity->email !== null) {
                $userRecord->email = $entity->email;
            }
            $userRecord->save();

            $detailRecord = UserDetail::where('user_id', $entity->id)->first();
            if ($detailRecord === null) {
                throw new UserDetailNotFoundException("ID {$entity->id} のユーザーの詳細情報が見つかりませんでした。");
            }
            if ($entity->address !== null) {
                $detailRecord->address = $entity->address;
            }
            if ($entity->phone_number !== null) {
                $detailRecord->phone_number = $entity->phone_number;
            }
            $detailRecord->save();
        });
    }
}
```

- ※データベーストランザクションに関してはアプリケーションサービス層で行う場合もあります。

```php
namespace App\Entities;

use Exception;

final class UserNotFoundException extends Exception
{
}
```

```php
namespace App\Entities;

use Exception;

final class UserDetailNotFoundException extends Exception
{
}
```

例外クラスもエンティティを同じ場所に配置しています。

## 開発体験

### N+1検出設定をしない

ローカル環境でN+1を検出したら例外を発生させる設定です。  
実際のSQLログを見に行ったりせずともN+1を発見できるので開発体験が良くなるはずです！

設定して損ないです。

app/Providers/AppServiceProvider.php

```php
<?php

declare(strict_types=1);

namespace App\Providers;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\ServiceProvider;

final class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Model::shouldBeStrict(! $this->app->isProduction());
    }
}
```

- [https://qiita.com/ucan-lab/items/8eab84e37421f907dea0#n1%E6%A4%9C%E5%87%BA%E8%A8%AD%E5%AE%9A](https://qiita.com/ucan-lab/items/8eab84e37421f907dea0#n1%E6%A4%9C%E5%87%BA%E8%A8%AD%E5%AE%9A)

### PhpStormを使わない

PhpStormはPHPの統合開発環境です。

コードジャンプ、コード補完機能、静的解析が特に強力だと思います。  
メインでLaravelを使用する方には有償ですがぜひ使用してもらいたいです。

個人的におすすめなスライドと動画をご紹介します。

- PhpStorm30分集中超絶技巧
	- [https://speakerdeck.com/yusuke/phpstorm30fen-ji-zhong-chao-jue-ji-qiao-number-phperkaigi-number-a](https://speakerdeck.com/yusuke/phpstorm30fen-ji-zhong-chao-jue-ji-qiao-number-phperkaigi-number-a)
	- [https://www.youtube.com/watch?v=DLs9bbySp1I](https://www.youtube.com/watch?v=DLs9bbySp1I)
- PhpStorm最新情報 AIとnew UI、便利プラグイン
	- [https://speakerdeck.com/yusuke/phpstormzui-xin-qing-bao-aitonew-ui-bian-li-puraguin-number-phpcon-okinawa](https://speakerdeck.com/yusuke/phpstormzui-xin-qing-bao-aitonew-ui-bian-li-puraguin-number-phpcon-okinawa)

### IDEヘルパーライブラリを使わない

これを導入しないとPhpStormの強力なコードジャンプが機能しません。  
必須で導入するべきライブラリです。

```shell
$ composer require --dev barryvdh/laravel-ide-helper

# Git管理したくないファイル
$ echo _ide_helper.php >> .gitignore
$ echo _ide_helper_models.php >> .gitignore
$ echo .phpstorm.meta.php >> .gitignore

# Model用のPHPDocsを生成する
$ php artisan ide-helper:models --write --reset
# Facade用のPHPDocを生成する
$ php artisan ide-helper:generate
# PhpStorm用のメタファイルを生成
$ php artisan ide-helper:meta
```

`--write` でモデルに直接PhpDocを生成した方がコードジャンプする時に2ファイル表示されずに済みます。  
`--reset` で実行するたびに再生成した方が新規カラムが追加された時も順番が整頓されて良いです。

### dump-server を使わない

```shell
$ composer require --dev beyondcode/laravel-dump-server
```

```text
$ php artisan dump-server
```

`dump();` ヘルパー関数を仕込むとdump-serverのコンソールに出力してくれます。  
便利なのがレスポンスにデバッグログを出力しないのでレスポンスを汚さず開発できます。

Bladeの場合はまだ良いですが、  
Jsonを返すルーティングの場合はJsonとデバッグログを同時に確認しながら開発を進められるので気持ちよく開発できます。

### Pintを使わない

Laravel 9系から公式に標準搭載されています。

```shell
# ルール適用
$ ./vendor/bin/pint

# 変更箇所の詳細表示
$ ./vendor/bin/pint -v

# お試し実行
$ ./vendor/bin/pint --test
```

Laravel Pintのおすすめのルールは別記事にまとめています。

## さいごに

もっとあるとはずなんですが、思い出したのがこれだけでした。  
こんなアンチパターン見かけた！という方いましたらコメントくださいませ！

[1](https://qiita.com/ucan-lab/items/#comments)

[97](https://qiita.com/ucan-lab/items/bdef3bc513c779116834/likers)

65