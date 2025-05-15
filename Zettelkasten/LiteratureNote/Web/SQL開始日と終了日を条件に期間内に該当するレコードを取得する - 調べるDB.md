---
title: SQL/開始日と終了日を条件に期間内に該当するレコードを取得する - 調べるDB
source: https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#jaa6425f
author: 
published: 
created: 2025-05-15
description: 
tags:
  - web
  - SQL
---
---

- [Prev](https://db.just4fun.biz/?SQL/%E4%B8%80%E3%81%A4%E3%81%AEINSERT%E3%81%A7%E8%A4%87%E6%95%B0%E8%A1%8C%E3%82%92INSERT%E3%81%99%E3%82%8BSQL%E6%A7%8B%E6%96%87 "SQL/一つのINSERTで複数行をINSERTするSQL構文 (64d)")
- [SQL](https://db.just4fun.biz/?SQL "SQL (64d)")

---

## 開始日と終了日を条件に期間内に該当するレコードを取得するSQL

WHERE句の条件としての開始日(時)と終了日(時)がパラメータとして渡され、 この期間と重なるレコードを抽出するSQLになります。  
テーブルも開始日(時) の start\_date, 終了日(時)の end\_date カラムが存在します。

で、いつも「どう書くんだっけ？」となるので記事にしました(苦笑)。

- [開始日と終了日を条件に期間内に該当するレコードを取得するSQL](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#jaa6425f)
- [条件に指定する日が範囲内にあるかどうかを確認するSQL](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#x9bd2060)
	- [答え](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#kc25d5b8)
	- [実際に試してみる](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#c628a5ed)
	- [開始日と終了日を条件に実行してみる](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#t5136271)

[↑](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#navigator)

## 条件に指定する日が範囲内にあるかどうかを確認するSQL

範囲内にあるかどうか確認する日付を check\_start\_date と check\_end\_date とし、  
レコードの開始日は start\_date、終了日は end\_date とします。  

[↑](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#navigator)

### 答え

先に答えを記します。

```
WHERE
    start_date <= check_end_date
AND end_date   >= checK_start_date
```

となります。

[↑](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#navigator)

### 実際に試してみる

以下のテーブルを作成しデータを投入し試してみます。  
データベースは PostgreSQL を使って動作確認しました。

- 作成したテーブル
	```
	CREATE TABLE t1 (id int, start_date date, end_date date);
	```
- INSERTしたデータ
	```
	INSERT INTO t1 VALUES
	( 1,'2023-01-01','2023-01-31'),
	( 2,'2023-02-01','2023-02-28'),
	( 3,'2023-03-01','2023-03-31'),
	( 4,'2023-04-01','2023-04-30'),
	( 5,'2023-05-01','2023-05-31'),
	( 6,'2023-06-01','2023-06-30'),
	( 7,'2023-07-01','2023-07-31'),
	( 8,'2023-08-01','2023-08-31'),
	( 9,'2023-09-01','2023-09-30'),
	(10,'2023-10-01','2023-10-31'),
	(11,'2023-11-01','2023-11-30'),
	(12,'2023-12-01','2023-12-31');
	```
- 投入データの確認
	```
	sakuradb=> SELECT * FROM T1;
	 id | start_date |  end_date  
	----+------------+------------
	  1 | 2023-01-01 | 2023-01-31
	  2 | 2023-02-01 | 2023-02-28
	  3 | 2023-03-01 | 2023-03-31
	  4 | 2023-04-01 | 2023-04-30
	  5 | 2023-05-01 | 2023-05-31
	  6 | 2023-06-01 | 2023-06-30
	  7 | 2023-07-01 | 2023-07-31
	  8 | 2023-08-01 | 2023-08-31
	  9 | 2023-09-01 | 2023-09-30
	 10 | 2023-10-01 | 2023-10-31
	 11 | 2023-11-01 | 2023-11-30
	 12 | 2023-12-01 | 2023-12-31
	(12 行)
	```

[↑](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#navigator)

### 開始日と終了日を条件に実行してみる

以下のように範囲となる開始日と終了日を指定してSQLを実行した結果です。

```
SELECT * FROM t1 WHERE start_date <= '2023-01-01' AND end_date >= '2022-01-01';
 id | start_date |  end_date  
----+------------+------------
  1 | 2023-01-01 | 2023-01-31
(1 行)
```
```
SELECT * FROM t1 WHERE start_date <= '2023-08-01' AND end_date >= '2023-02-02';
 id | start_date |  end_date  
----+------------+------------
  2 | 2023-02-01 | 2023-02-28
  3 | 2023-03-01 | 2023-03-31
  4 | 2023-04-01 | 2023-04-30
  5 | 2023-05-01 | 2023-05-31
  6 | 2023-06-01 | 2023-06-30
  7 | 2023-07-01 | 2023-07-31
  8 | 2023-08-01 | 2023-08-31
(7 行)
```
```
SELECT * FROM t1 WHERE start_date <= '2024-01-31' AND end_date >= '2023-11-11';
 id | start_date |  end_date  
----+------------+------------
 11 | 2023-11-01 | 2023-11-30
 12 | 2023-12-01 | 2023-12-31
(2 行)
```
```
SELECT * FROM t1 WHERE start_date <= '2024-01-31' AND end_date >= '2024-01-01';
 id | start_date | end_date 
----+------------+----------
(0 行)
```

以上、指定した範囲に該当するレコードを取得するWHER句の条件式でした。

#### オンライン数: 4

---

[↑](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#navigator)

#### ご利用にあたり

- [プライバシーポリシー](https://db.just4fun.biz/?%E3%83%97%E3%83%A9%E3%82%A4%E3%83%90%E3%82%B7%E3%83%BC%E3%83%9D%E3%83%AA%E3%82%B7%E3%83%BC "プライバシーポリシー (3708d)")
- [お約束](https://db.just4fun.biz/?%E3%81%8A%E7%B4%84%E6%9D%9F "お約束 (3708d)")

---

[↑](https://db.just4fun.biz/?SQL/%E9%96%8B%E5%A7%8B%E6%97%A5%E3%81%A8%E7%B5%82%E4%BA%86%E6%97%A5%E3%82%92%E6%9D%A1%E4%BB%B6%E3%81%AB%E6%9C%9F%E9%96%93%E5%86%85%E3%81%AB%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B#navigator)

#### 公開サイト

- [Windowsと暮らす](https://win.just4fun.biz/)
- [Linuxと過ごす](https://linux.just4fun.biz/)
- [Web関連技術調査](https://web.just4fun.biz/)
- [Lightweight Languageと暮らす](https://ll.just4fun.biz/)
- [C言語のお勉強](https://c.just4fun.biz/)
- [Javaコードを書いてみる](https://java.just4fun.biz/)
- [仮想通貨(暗号通貨)メモ](https://cryptocurrency.just4fun.biz/)
- [ミニPC&シングルボードコンピュータ活用術](https://minipc.just4fun.biz/)

##### 最新の20件

**2024-07-01**
- [PostgreSQL/日付部分のみ取得しYYYYMMDDで取り出す方法](https://db.just4fun.biz/?PostgreSQL/%E6%97%A5%E4%BB%98%E9%83%A8%E5%88%86%E3%81%AE%E3%81%BF%E5%8F%96%E5%BE%97%E3%81%97YYYYMMDD%E3%81%A7%E5%8F%96%E3%82%8A%E5%87%BA%E3%81%99%E6%96%B9%E6%B3%95 "PostgreSQL/日付部分のみ取得しYYYYMMDDで取り出す方法 (64d)")
**2023-12-16**
- SQL/開始日と終了日を条件に期間内に該当するレコードを取得する
- [SQL](https://db.just4fun.biz/?SQL "SQL (64d)")
**2023-11-18**
- [MSSQL/テーブルのレコード内容をINSERT SQLとしてダンプする方法](https://db.just4fun.biz/?MSSQL/%E3%83%86%E3%83%BC%E3%83%96%E3%83%AB%E3%81%AE%E3%83%AC%E3%82%B3%E3%83%BC%E3%83%89%E5%86%85%E5%AE%B9%E3%82%92INSERT+SQL%E3%81%A8%E3%81%97%E3%81%A6%E3%83%80%E3%83%B3%E3%83%97%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95 "MSSQL/テーブルのレコード内容をINSERT SQLとしてダンプする方法 (544d)")
- [MSSQL](https://db.just4fun.biz/?MSSQL "MSSQL (544d)")
**2023-11-05**
- [MSSQL/ジョブの動作状況確認方法](https://db.just4fun.biz/?MSSQL/%E3%82%B8%E3%83%A7%E3%83%96%E3%81%AE%E5%8B%95%E4%BD%9C%E7%8A%B6%E6%B3%81%E7%A2%BA%E8%AA%8D%E6%96%B9%E6%B3%95 "MSSQL/ジョブの動作状況確認方法 (557d)")
**2023-10-20**
- [SQL/SQL Serverでテーブルをコピーする SELECT \* INTO](https://db.just4fun.biz/?SQL/SQL+Server%E3%81%A7%E3%83%86%E3%83%BC%E3%83%96%E3%83%AB%E3%82%92%E3%82%B3%E3%83%94%E3%83%BC%E3%81%99%E3%82%8B+SELECT+%2A+INTO "SQL/SQL Serverでテーブルをコピーする SELECT * INTO (573d)")
**2023-08-29**
- [MSSQL/SQL Serverが稼働しているサーバー名を確認する方法](https://db.just4fun.biz/?MSSQL/SQL+Server%E3%81%8C%E7%A8%BC%E5%83%8D%E3%81%97%E3%81%A6%E3%81%84%E3%82%8B%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E5%90%8D%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95 "MSSQL/SQL Serverが稼働しているサーバー名を確認する方法 (625d)")
**2023-05-17**
- [MSSQL/SSMSのクエリ実行でWITH句に誤りがないのにエラーになる場合の原因](https://db.just4fun.biz/?MSSQL/SSMS%E3%81%AE%E3%82%AF%E3%82%A8%E3%83%AA%E5%AE%9F%E8%A1%8C%E3%81%A7WITH%E5%8F%A5%E3%81%AB%E8%AA%A4%E3%82%8A%E3%81%8C%E3%81%AA%E3%81%84%E3%81%AE%E3%81%AB%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%AB%E3%81%AA%E3%82%8B%E5%A0%B4%E5%90%88%E3%81%AE%E5%8E%9F%E5%9B%A0 "MSSQL/SSMSのクエリ実行でWITH句に誤りがないのにエラーになる場合の原因 (729d)")
**2023-05-02**
- [SQL/SQLServerでサブクエリ同士をLEFT JOINする](https://db.just4fun.biz/?SQL/SQLServer%E3%81%A7%E3%82%B5%E3%83%96%E3%82%AF%E3%82%A8%E3%83%AA%E5%90%8C%E5%A3%AB%E3%82%92LEFT+JOIN%E3%81%99%E3%82%8B "SQL/SQLServerでサブクエリ同士をLEFT JOINする (744d)")
**2023-05-01**
- [SQL/SQLServerでPIVOTを使ったサンプル](https://db.just4fun.biz/?SQL/SQLServer%E3%81%A7PIVOT%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%82%B5%E3%83%B3%E3%83%97%E3%83%AB "SQL/SQLServerでPIVOTを使ったサンプル (745d)")
**2023-03-07**
- [MSSQL/クエリー作成時等に変更内容等が反映されない場合の対処方法](https://db.just4fun.biz/?MSSQL/%E3%82%AF%E3%82%A8%E3%83%AA%E3%83%BC%E4%BD%9C%E6%88%90%E6%99%82%E7%AD%89%E3%81%AB%E5%A4%89%E6%9B%B4%E5%86%85%E5%AE%B9%E7%AD%89%E3%81%8C%E5%8F%8D%E6%98%A0%E3%81%95%E3%82%8C%E3%81%AA%E3%81%84%E5%A0%B4%E5%90%88%E3%81%AE%E5%AF%BE%E5%87%A6%E6%96%B9%E6%B3%95 "MSSQL/クエリー作成時等に変更内容等が反映されない場合の対処方法 (800d)")
**2023-01-22**
- [SQLite/PHPのPDOを使ってSQLiteを操作する](https://db.just4fun.biz/?SQLite/PHP%E3%81%AEPDO%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6SQLite%E3%82%92%E6%93%8D%E4%BD%9C%E3%81%99%E3%82%8B "SQLite/PHPのPDOを使ってSQLiteを操作する (844d)")
- [FrontPage](https://db.just4fun.biz/ "FrontPage (64d)")
- [SQLite](https://db.just4fun.biz/?SQLite "SQLite (64d)")
**2022-11-09**
- [MSSQL/SSMSで実行したクエリー結果をコピー時改行も保持する方法](https://db.just4fun.biz/?MSSQL/SSMS%E3%81%A7%E5%AE%9F%E8%A1%8C%E3%81%97%E3%81%9F%E3%82%AF%E3%82%A8%E3%83%AA%E3%83%BC%E7%B5%90%E6%9E%9C%E3%82%92%E3%82%B3%E3%83%94%E3%83%BC%E6%99%82%E6%94%B9%E8%A1%8C%E3%82%82%E4%BF%9D%E6%8C%81%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95 "MSSQL/SSMSで実行したクエリー結果をコピー時改行も保持する方法 (918d)")
**2022-08-27**
- [MSSQL/VS2019にSSISをインストールする手順](https://db.just4fun.biz/?MSSQL/VS2019%E3%81%ABSSIS%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B%E6%89%8B%E9%A0%86 "MSSQL/VS2019にSSISをインストールする手順 (992d)")
**2022-08-18**
- [MSSQL/SQL Server エージェントを自動起動にする](https://db.just4fun.biz/?MSSQL/SQL+Server+%E3%82%A8%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%B3%E3%83%88%E3%82%92%E8%87%AA%E5%8B%95%E8%B5%B7%E5%8B%95%E3%81%AB%E3%81%99%E3%82%8B "MSSQL/SQL Server エージェントを自動起動にする (1001d)")
**2022-08-09**
- [SQLite/UbuntuにDB Browser for SQLiteをインストール](https://db.just4fun.biz/?SQLite/Ubuntu%E3%81%ABDB+Browser+for+SQLite%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB "SQLite/UbuntuにDB Browser for SQLiteをインストール (1010d)")
- [RecentDeleted](https://db.just4fun.biz/?RecentDeleted "RecentDeleted (1010d)")

##### 今日の15件

##### 人気の15件

[edit](https://db.just4fun.biz/?cmd=edit&page=MenuBar "Edit MenuBar")

Counter: 5891, today: 20, yesterday: 10

---

Last-modified: 2023-12-16 (土) 20:07:22 (516d)