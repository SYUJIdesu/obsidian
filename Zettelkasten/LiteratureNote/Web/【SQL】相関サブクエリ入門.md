---
title: 【SQL】相関サブクエリ入門
source: https://qiita.com/aki3061/items/736abd6ea883ba647586
author:
  - "[[aki3061]]"
published: 2017-02-05
created: 2025-05-22
description: 相関サブクエリとはサブクエリの一種であり、外側のクエリの値をサブクエリ内で使用する。相関サブクエリを使用したSQLはややこしくて読みにくい場合が多いが、基本形を１つおさえておくとだいぶ理解しやすく…
tags:
  - web
  - DB
  - SQL
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から5年以上が経過しています。

相関サブクエリとはサブクエリの一種であり、外側のクエリの値をサブクエリ内で使用する。

相関サブクエリを使用したSQLはややこしくて読みにくい場合が多いが、基本形を１つおさえておくとだいぶ理解しやすくなる。

サブクエリが何かわからない場合は、こちらを先に読んで欲しい。  
[http://qiita.com/mokrai/items/6df0513ccc5aa40a075a](http://qiita.com/mokrai/items/6df0513ccc5aa40a075a)

## 動作確認環境とテーブル

PostgreSQL 9.4でクエリの動作を確認した。  
また、使用したテーブルの定義は下記。

```sql
CREATE TABLE Employees (
  id         INTEGER PRIMARY KEY,
  name       VARCHAR(10) NOT NULL,
  age        INTEGER NOT NULL,
  department VARCHAR(10) NOT NULL
);

INSERT INTO Employees VALUES(1,'Sato',23,'営業');
INSERT INTO Employees VALUES(2,'Suzuki',35,'営業');
INSERT INTO Employees VALUES(3,'Saito',38,'営業');
INSERT INTO Employees VALUES(4,'Yamada',42,'開発');
INSERT INTO Employees VALUES(5,'Tanaka',41,'開発');
INSERT INTO Employees VALUES(6,'Takahashi',35,'開発');
```

SELECTするとこんな感じ。

```sql
# SELECT * FROM Employees ;
```

```text
id |   name    | age | department 
----+-----------+-----+------------
  1 | Sato      |  23 | 営業
  2 | Suzuki    |  35 | 営業
  3 | Saito     |  38 | 営業
  4 | Yamada    |  42 | 開発
  5 | Tanaka    |  41 | 開発
  6 | Takahashi |  35 | 開発
(6 rows)
```

## 実行例

下記のSQLは、所属部署の平均年齢よりも若い社員を表示する。(ついでに年齢と所属部署も表示する)  
ここで「所属部署の」というのがポイントであり、たとえば「営業」の社員であれば営業に所属する社員の平均年齢が基準となる。（つまり、開発社員の年齢は考慮しない）

```sql
SELECT name, age, department
  FROM Employees as e1
 WHERE e1.age < (SELECT AVG(age) as avg_age
                   FROM Employees as e2
                  WHERE e1.department = e2.department);
```

```text
name    | age | department 
-----------+-----+------------
 Sato      |  23 | 営業
 Takahashi |  35 | 開発
(2 rows)
```

このSQLで注目すべきは、 `WHERE e1.department = e2.department` の部分だ。  
WHERE句で指定されたサブクエリは、外側のクエリの結果の１行ごとに実行され、その外側のクエリの結果のdepartmentの値をサブクエリ内で利用している。

「所属部署の」という制限を外す場合、つまり「社員全体の平均年齢よりも若い社員」を知りたい場合は、相関サブクエリではないサブクエリを用いる。

```sql
SELECT name, age, department
  FROM Employees
 WHERE age < (SELECT AVG(age)
               FROM Employees);
```

```text
name    | age | department 
-----------+-----+------------
 Sato      |  23 | 営業
 Suzuki    |  35 | 営業
 Takahashi |  35 | 開発
(3 rows)
```

## 所属部署の平均年齢もあわせて表示したい場合

SELECTの列の指定で、WHERE句と同じ式を書けば実現できる。  
（DRY原則に反しているので、もっとスマートなやり方があれば教えてください。）

```sql
SELECT name,
       age,
       department,
       (SELECT AVG(age) as avg_age
                   FROM Employees as e2
                  WHERE e1.department = e2.department)
  FROM Employees as e1
 WHERE e1.age < (SELECT AVG(age) as avg_age
                   FROM Employees as e2
                  WHERE e1.department = e2.department);
```

```text
name    | age | department |       avg_age       
-----------+-----+------------+---------------------
 Sato      |  23 | 営業       | 32.0000000000000000
 Takahashi |  35 | 開発       | 39.3333333333333333
(2 rows)
```

２つのサブクエリ内で `e2` という同一のテーブルの別名を指定している点が気になったかもしれないが、サブクエリ内でスコープが分かれるため問題ない。

[6](https://qiita.com/aki3061/items/#comments)

[149](https://qiita.com/aki3061/items/736abd6ea883ba647586/likers)

131