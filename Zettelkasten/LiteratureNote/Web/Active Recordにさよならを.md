---
title: "Active Recordにさよならを"
source: "https://zenn.dev/naitsu/articles/ed53b91004dce9#active-record%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3"
author:
  - "[[Zenn]]"
published: 2024-02-27
created: 2025-05-15
description:
tags:
  - "web"
  - "Active Record"
---
18

1[idea](https://zenn.dev/tech-or-idea)

## まえがき

この記事は以下の記事を先に読むことで話の流れが想像しやすくなるかと思います。  
お時間ある方は先に読んでいただければ嬉しく思います。

## 概要

この記事は筆者の以下の考えを記載している記事です。

- APPとDBは性質が異なるため、DBには変更が発生しない「行為の履歴」を保存するのが良い
	- APPの性質：ロジックを保持する。変更容易
	- DBの性質：データを保持する。変更困難
- Active RecordパターンはAPPとDBが密結合となるため、将来的な負債を生みやすい
- RepositoryパターンはAPPとDBが疎結合となるため、APPとDBの特性を活かし変化に柔軟な設計ができる

## Active Recordとは

Active Recordとは「エンタープライズアプリケーションアーキテクチャパターン」（通称EAAP）の本で示されているデータアクセス周りの設計パターンです。  
Ruby on RailsのORMがActiveRecordという名前ですが、ActiveRecordはActive Recordパターンに則って作られたORMです。  
PHPのLaravel標準のORMのEloquantもActive Record思想のORMです。  
（以下、単にActive Recordと記載した場合はActive Recordパターンを指すものとします。）

Active Recordの特徴は以下です。

1. DBテーブル1つに対して対応するクラスを1つ作成する
2. データの取得処理とそれに関するビジネスロジックを同一クラスが持つ

LaravelでいうとModelクラスがActive Recordに則ったクラスです。  
Active Recordパターンと異なるパターンとしてRepositoryパターンがあります。Repositoryパターンに則った設計としてはクリーンアーキテクチャやDDDがあります。

パターンについて分かりやすくまとまっていると感じた記事がありましたので以下にリンクを設置します。

## 具体例を用いたコード例

ここで具体的にするため以下の支払機能を実装するものとして考えていきましょう。

\[支払関連の業務\]

1. 支払義務が発生した場合に支払情報を記録する
2. 記録した支払に対して振込をする

\[実装画面への要求\]

1. 支払データを一覧で確認できる
2. 一覧で振込済か未振込かが判断できる
3. 支払データを1件選択して振込処理が可能

比較対象としてRepository層を設けたコードも書いてみます。

## Active Recordパターン

LaravelらしきWEB FWとEloquantらしきORMを用いたとして実装します。  
ajaxで処理をしているとして返却はjson形式で、エラー処理は考えないものとします。

### DBテーブル設計

支払テーブル

| 支払ID | 金額 | 状態区分 | 振込手数料区分 |
| --- | --- | --- | --- |
| 1 | 100 | 2 | 1 |
| 2 | 300 | 2 | 2 |
| 3 | 500 | 1 | 1 |

### APPコード

Model

```php
public class Payment extends Model {
    public int $payment_id;
    public int $amount;
    public int $state_kbn;
    public int $transfer_charge_kbn;

    public transfer() {
        if ($this->transfer_charge_kbn == 1) {
            // transfer amount with charge
        } elseif ($this->transfer_charge_kbn == 2) {
            // transfer amount without charge
        } 
    }
}
```

Cotroller

```php
public class PaymentController extends Controller {
    
    public get(int $id) {
        $payment = Payment::find($id);
        return Response::json($payment, Response::OK);
    }

    public transfer(int $id, Request $request) {
        $payment = Payment::find($id);
        $payment->transfer();
        return Response::json($payment, Response::OK);
    }
}
```

とてもシンプルなコードですね。

## Repositoryパターン

RepositoryパターンはJavaで、なんらかのWEB FWとDIコンテナを用いて実装しているとします。  
今回は以下のクラス分けで実装します。

- ビジネスロジックを持つEntity
- DBからEntityを組み立てるRepository
- それ以外の処理を担当するController

### DBテーブル設計

支払書テーブル

| 支払書No | 金額 | 振込手数料区分 |
| --- | --- | --- |
| 1 | 100 | 1 |
| 2 | 300 | 2 |
| 3 | 500 | 1 |

振込テーブル

| 振込No | 支払書No | 振込日 | 振込者名 |
| --- | --- | --- | --- |
| 1 | 1 | 2024-02-01 | 田中 |
| 2 | 2 | 2024-02-02 | 鈴木 |

振込テーブルが寂しかったので、要件としては不要ですが振込日も保持するようにしています。

### APPコード

Entity

```java
public interface IPaymentEntity {
    public void transfer();
}

public class PaymentEntityWithCharge implements IPaymentEntity {
    private int paymentNo;
    private int amount;
    private boolean isTransfered;
    private Date transferDate;
    private String transferExecutorName;

    public PaymentEntityWithCharge(int paymentNo, int amount, boolean isTrasfered, Date transferDate, String transferExecutorName) {
        this.paymentNo = paymentNo;
        this.amount = amount;
        this.isTransfered = isTrasfered;
        this.transferDate = transferDate;
        this.transferExecutorName = transferExecutorName;
    }
    
    @override
    public void transfer() {
        // transfer amount with charge
    }
}

public class PaymentEntityWithoutCharge implements IPaymentEntity {
    private int paymentNo;
    private int amount;
    private boolean isTransfered;
    private String transgerExecutorName;

    public PaymentEntityWithoutCharge(int paymentNo, int amount, boolean isTrasfered, Date transferDate, String transferExecutorName) {
        this.paymentNo = paymentNo;
        this.amount = amount;
        this.isTransfered = isTrasfered;
        this.transferDate = transferDate;
        this.transferExecutorName = transferExecutorName;
    }

    @override
    public void transfer() {
        // transfer amount without charge
    }
}
```

Repository

```java
public class PaymentRepository {
    public IPaymentEntity find(int id, DBManager db) {
        IPaymentEntity payment = null; 
        
        // RDBからのデータの取得
        String sql = """
            SELECT 
                 payment.payment_id
                 , payment.amount
                 , payment.transfer_charge_kbn
                 , transfer.transfer_date
                 , transfer.transferExecutorName
            FROM payment 
            LEFT JOIN transfer 
                ON payment.id = transfer.payment_id 
            WHERE payment.id = ?
            ;
        """;
        ResultSet resultSet = db.query(sql, [id]);

        // 取得結果が存在しない場合、null返却
        if (!resultSet.next()) {
            return null;
        }

        // Entitiyの作成
        if (resultSet.trasfer_charge_kbn == 1) {
            // 支払振込手数料あり
            payment = new PaymentEntityWithCharge(
                resultSet.getInt("payment_id"), 
                resultSet.getInt("amount"), 
                resultSet.getDate("transfer_date") != null ? true : false,
                resultSet.getDate("transfer_date"),
                resultSet.getString("transfer_executor_name")
            );
        } else if (resultSet.trasfer_charge_kbn == 2) {
            // 支払振込手数料なし
            payment = new PaymentEntityWithoutCharge(
                resultSet.getInt("payment_id"), 
                resultSet.getInt("amount"), 
                resultSet.getDate("transfer_date") != null ? true : false, 
                resultSet.getDate("transfer_date"),
                resultSet.getString("transfer_executor_name")
            );
        }
        return payment;
    }
}
```

Cotroller

```java
public class PaymentController extends Controller {
    
    public Response get(int id, PaymentRepository repo) {
        IPaymentEntity payment = repo.find(id);
        return Response.json(payment, Response.OK);
    }

    public Response transfer(int id, PaymentRepository repo) {
        IPaymentEntity payment = repo.find(id);
        payment.transfer();
        return Response.json(payment, Response.OK);
    }
}
```

長い。。。  
書くだけで疲れました。

## Active RecordパターンとRepositoryパターンの比較

違いは色々あるかと思いますが、ここでは「DBのテーブル設計が異なる」ことについて記載します。

Active RecordはDBテーブルと1対1で対応するクラスをAPPに持ちます。  
そのため、モデルクラスのメンバ変数はDBのカラムと一致します。つまり「APPクラス設計=DBテーブル設計」になります。  
当然ですが、画面に表示したいものはクラスに保持しなければ表示できません。少し強引ですが「画面項目=APPクラス設計」と言えるでしょう。  
Active Recordでは、「画面項目=APPクラス設計=DBテーブル設計」となると筆者は考えています。

Repositoryパターンはどうでしょうか？  
RepositoryがDBテーブルからEntityを作成しているため、APPでビジネスロジックを責務とするEntityがDBのテーブル構成と一致していません。  
Repositoryパターンでは「画面項目=APPクラス設計!=DBテーブル設計」となります。

Active RecordはAPPとDBが密結合となり、RepositoryパターンはAPPとDBが疎結合となるようです。

## APPとDBが疎結合である利点

APPとDBが疎結合となることに利点はあるのでしょうか？

筆者はDBの正規化が守られることが利点と感じています。APPとDBの背景にはオブジェクト指向とリレーショナルモデルがあり、両者の考え方は根本から異なっています。その違いはインピーダンスミスマッチと呼ばれます。

Repositoryパターンは、このインピーダンスミスマッチの解消をRepositoryが担当するため、DBはリレーショナルモデルの考え方、つまり正規化を守ったテーブル設計を保つことが可能です。  
正規化を守ることは保守、改修のしやすさに直結すると筆者は考えています。

## 仕様変更への対応

保守、改修のしやすさを確認するため、試しに支払について仕様変更を発生させてみましょう。

\[支払関連の業務\]

1. 支払義務が発生した場合、それを記録する
2. 記録した支払に対して振込する
3. 振込後に支払情報を紙に印刷して保管する ←NEW!

\[実装画面への要求\]

1. 支払データを一覧で確認できる
2. 支払データを1件選択して振込処理が可能
3. 一覧で未振込か振込済か印刷済かを判断できる　←UPD!
4. 誰がいつ、振込したかを確認できる　←NEW!
5. 誰がいつ、印刷したかを確認できる　←NEW!

システム化しましたが、支払情報は紙でも保存する運用となったようです。  
また業務証跡も表示したいようです。

## Active Recordパターン

追加仕様に対応するためAcive Recordパターンに修正を加えます。  
APPのコードは今回は省略します。

支払テーブル

| 支払ID | 金額 | 状態区分 | 振込日 | 振込者名 | 振込手数料区分 | 印刷日 | 印刷者名 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 100 | 3 | 2024-02-01 | 田中 | 1 | 2024-02-03 | 鈴木 |
| 2 | 300 | 2 | 2024-02-02 | 鈴木 | 2 | NULL | NULL |
| 3 | 500 | 1 | NULL | NULL | 1 | NULL | NULL |

※状態区分：１=未振込、２=振込済、３=印刷済

「状態区分」に３=印刷済という状態を追加することで対応しました。テーブルがちょっと見づらくなってきました。  
正規化も出来ていないようです。

## Repositoryパターン

追加仕様に対応するためRepositoryパターンに修正を加えます。  
APPのコードは今回は省略します。

支払書テーブル

| 支払書No | 金額 | 振込手数料区分 |
| --- | --- | --- |
| 1 | 100 | 1 |
| 2 | 300 | 2 |
| 3 | 500 | 1 |

振込テーブル

| 振込No | 支払書No | 振込日 | 振込者名 |
| --- | --- | --- | --- |
| 1 | 1 | 2024-02-01 | 田中 |
| 2 | 2 | 2024-02-02 | 鈴木 |

印刷テーブル

| 印刷No | 支払書No | 印刷日 | 印刷者名 |
| --- | --- | --- | --- |
| 1 | 1 | 2024-02-03 | 鈴木 |

こちらはテーブル構成を見ただけ大体どんな業務なのか想像がつくと思います。  
正規化も保たれているようです。

## Active RecordパターンとRepositoryパターンの比較

正規化の観点で見ると、Active Recordは正規化をすることができません。  
「画面項目=APPクラス設計=DBテーブル設計」であるため画面項目の変更がそのままDBまで波及するからです。  
画面は「印刷者名」を要求するためAPPも「印刷者名」を要求し、APPとDBが密結合しているためDBの支払テーブルも同様に「印刷者名」を保持することなり正規化が崩れます。

Repositoryパターンでは、印刷を行為として捉え、印刷テーブルに行為の履歴として溜め込む設計をしています。  
これが可能なのは「APPクラス設計!=DBテーブル設計」なため、APPとDBを異なる設計とすることが可能だからです。結果として正規化を保つことが可能です。

Active Recordではさらに改修が発生した場合、例えば入金確認業務が追加された場合を考えると支払テーブルのカラム数がどうなるのか恐ろしく感じます。

## バックエンドの設計で大事なこと

ここまで例を挙げて考えてきましたが、ここから筆者がバックエンド開発に求められていると感じることを記載しようと思います。

## APPとDBの特徴

バックエンド開発に求められることと書きましたが、まずはその要素であるAPPとDBについてその特徴を考えたいと思います。  
それぞれの特徴から抑えるべきポイントが見えるはずです。

APPの主な目的はロジックの組み込みです。APPはロジックをそのままコードとして組み込みます。  
またロジックはコードを変更すれば全てのデータは変更後のロジックに沿って処理されます。  
APP（ロジック）の変更は比較的容易です。

DBの主な目的はデータの保持です。テーブル構成を変更することは収集するデータを変更することになります。過去のデータに対して新規にデータを収集することはできないため、大きなテーブル構造の変更は困難でしょう。  
DB（データ）の変更は比較的困難と言えるでしょう。

## 筆者が大事にしていること

DBの変更が困難で、APPの変更は容易であることが分かりました。  
このことから筆者は「DBは変更が発生しないようにデータを保存する必要がある」と考えています。

筆者が「行為の履歴」をDBに保存しようとするのは、行為を行った事実は変更されることがないからです。  
対して「状態」は「行為の結果」をどう解釈したかを表します。例では「印刷行為」を完了したことを「支払は印刷済状態となった」と解釈しています。

しかし、状態は解釈であるためにビジネスロジックの変更によりその定義や前提は変わります。  
例えば、印刷は振込後だけでなく振込前にも実施する場合が増えた、印刷は複数回行うようになったなどです。

印刷は振込後だけでなく振込前にも実施するようになったを考えると、  
Active Recordの場合、「振込前」かつ「印刷済」を表すことが出来ないため、支払テーブルに「印刷済フラグ」カラムを増やすことになります。  
このようにビジネスロジックの変更がDBに波及します。その度にAPPのコード変更のみならずDBのデータの移行をしなければならなくなります。

Repositoryの場合、「行為の履歴」を保存しているためテーブル構造の大きな変更は発生しません。「状態」はDBの「行為の履歴」をAPPのRepositoryが解釈してEntityとして組み立てるからです。変更の影響は変更容易なAPPのRepositoryとEntityに収まります。  
（ちなみに「行為の履歴」を保存することは正規化することとほぼイコールとなります。正規化はNULLを排除することであるため、正規化に沿うとデータの発生タイミングでテーブルは分割されるからです。）

Repositoryパターンを採用することで、変更困難なDBには不変な行為の履歴を保持する責務を、変更容易なAPPには変化が激しいロジックの責務を担当させることができ、変化に柔軟に対応できるシステムが実現できると考えています。

## まとめ

長々書きましたが、この記事をまとめると以下です。

- APPとDBは性質が異なるため、DBには変更が発生しない「行為の履歴」を保存するのが良い
	- APP：ロジックを保持する。変更容易。
	- DB：データを保持する。変更困難。
- Active RecordパターンはAPP=DBとして密結合となるため、将来的には負債となりやすい。
- RepositoryパターンはAPPとDBを疎結合とするため、APPとDBの特性を活かして変化に柔軟に対応できる。

システムを作成することを考えた場合、変更容易であることは大事であると筆者は考えています。

## あとがき

ここまで記載しておいてですが、この記事は偏った意見を記載しています。  
タイトルはセンセーショナルなタイトルにしたかったので、このようなタイトルにしていますが、Active Recordが常に悪い設計とは思っていせん。コード量は圧倒的にActive Recordに軍配が上がります。  
Active Recordに向かないモノがあり、それをActive Recordで実装すると困難があるという事だと思います。

次に書きたい記事としては、Active RecordパターンとRepositoryパターンの使い分けについて筆者の意見を記述しようと思っています。次の記事が書けましたらリンクを配置します。  
（筆者はサービスはActive Recordに向いているが、システムはActive Recordに向いていないと思っています。そもそもサービスとシステムの違いは何なのか？という話になりますが。。。）

## 気になったこと

この記事を書くにあたり、ネット検索して気になったことを記載します。「ORM不要論」です。

筆者の経歴として、元々SI業界で従事し主にデータ（DB）を重視した開発をしていました。その後にWEB業界に転職した経歴を持ちます。  
WEB業界で働いて感じた事は（執筆時に勤めている職場では）データをあまり重視していないと感じました。

想像ですが、WEB業界ではActive RecordなWEB FWを使う方が多いためAPPに注意が行き、SI業界のSEはDBに注意が行きがちなのではないかと思います。  
「ORM不要論」などはデータに注意が行くエンジニアとAPPに注意が行くエンジニアの意見の食い違いのように感じてます。  
この記事は「データに注意が行くエンジニア」目線の偏見記事となります。

筆者はまだ経験の浅いエンジニアなので、他のエンジニアの方がActive Recordやこの記事にどのような印象を受けているか知りたいと思っています。  
ぜひコメントにてご意見をいただけると嬉しく思います。

18

1