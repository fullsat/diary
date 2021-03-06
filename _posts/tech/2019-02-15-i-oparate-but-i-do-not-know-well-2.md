---
title:  "俺は雰囲気でPostgreSQLの運用をやっている(実験編)"
date:   2019-02-15 19:00:00 +0900
category: tech
tags: postgresql
layout: post
---

## [前回](https://fullsat.github.io/2019/02/13/i-oparate-but-i-do-not-know-well/)までのあらすじ

「全然わからない、俺は雰囲気でPostgreSQLをやっている」

```
ERROR:  canceling statement due to conflict with recovery
```

このエラーはなんだ？

[...ここを読む...](https://qiita.com/sawada_masahiko/items/16a30b2df52331c86483)

>スレーブでの参照とスレーブが参照しているタプルを物理削除するWAL(VACUUM等で出力される)の適用がコンフリクトします。

なるほど全然分からない。実験しよう。

# 事前準備

terraformで実験環境を作成します

サブネットやセキュリティグループは各自で調整します

コンソールから作っても問題ありません

* クラスター
* Writerー用インスタンス
* Reader用インスタンス
* サブネットグループ
* クラスターパラメータグループ
* パラメータグループ

```
provider "aws" { region = "ap-northeast-1" }

resource "aws_rds_cluster_instance" "db01" {
  engine = "aurora-postgresql"
  engine_version = "9.6.8"
  cluster_identifier = "daijinadb"
  identifier         = "db01"
  instance_class     = "db.r4.large"
  db_subnet_group_name    = "${aws_db_subnet_group.daijinadb_subnet.name}"
  db_parameter_group_name = "${aws_db_parameter_group.daijinadb_parameter.name}"

  publicly_accessible = false
}

resource "aws_rds_cluster_instance" "db02" {
  engine = "aurora-postgresql"
  engine_version = "9.6.8"
  cluster_identifier = "daijinadb"
  identifier         = "db02"
  instance_class     = "db.r4.large"
  db_subnet_group_name    = "${aws_db_subnet_group.daijinadb_subnet.name}"
  db_parameter_group_name = "${aws_db_parameter_group.daijinadb_parameter.name}"
  publicly_accessible = false
}

resource "aws_rds_cluster" "daijinadb" {
  cluster_identifier = "daijinadb"
  engine = "aurora-postgresql"
  engine_version = "9.6.8"
  database_name = "totemodaijinadb"
  master_username = "postgres"
  master_password = "hogehoge"
  skip_final_snapshot = "true"
  backup_retention_period = 1
  port = 5432
  db_cluster_parameter_group_name = "${aws_rds_cluster_parameter_group.daijinadb_parameter.name}"
  vpc_security_group_ids    = "securitygroup-iikanjinihenkoushitene"
  db_subnet_group_name = "${aws_db_subnet_group.daijinadb_subnet.name}"
}

resource "aws_rds_cluster_parameter_group" "daijina_parameter" {
  name        = "daijina_parameter"
  family      = "aurora-postgresql9.6"
  description = "aurora-postgresql9.6"

  parameter {
    name = "timezone"
    value = "JST-9"
  }
}

resource "aws_db_parameter_group" "daijina_parameter" {
  name   = "daijina_parameter"
  family = "aurora-postgresql9.6"
  description = "totemo daiji"

  parameter {
    name = "log_min_duration_statement"
    value = "1000"
  }

  parameter {
    name = "log_statement"
    value = "ddl"
  }
}

resource "aws_db_subnet_group" "daijinadb_subnet" {
  name       = "daijinadb_subnet"
  description = "totemo daijina subnet"
  subnet_ids = [
    "subnet-iikanjinihenkoushitene-a",
    "subnet-iikanjinihenkoushitene-c"
  ]
}
```

## 仮説

実験用のインスタンスができたので実験前にどうすればキャンセルされるのか仮説を立ててみます。

建てた仮説が以下です。

|項番 | Writer  |   Reader |
|:---|:--------|:--------|
|1   | UPDATEで不要タプルを作成 | |
|2   | | 処理に時間のかかるSELECT |
|3   | VACUUMで更新行の物理削除を実行 | |
|4   | WALを生成 | |
|5   | | WALを適用 |
|6   | | 30秒後にSELECTがキャンセルされる |

UPDATEすると論理削除扱いになって、VACUUMしたタイミングで物理削除されるので、まずUPDATEしています。

次に、この状態で重いSELECTを実行します。これがキャンセルされれば今回の実験は成功です。

今度は論理削除扱いになっている行をVACUUMで物理削除します。

4,5は表面上確認できないので想像ですが、
Writer側でVACUUMをかければ、
Reader側で論理削除扱いになっているものも消す必要があるので、
そのためのWALを生成し、それを適用しようとするはずです。

最後にSELECTの結果を待っている状態でmax_standby_streaming_delay秒待ちます。

## 実験

* 事前準備(重いクエリを手軽に実行するための準備)

最初は空のインスタンスに大量のデータ入れなくちゃいけなくて実験無理じゃない？と思っていました。

ただ、簡単に重いSELECTを実現できる方法が紹介されていたのでこれを利用しました。

[https://qiita.com/tt2004d/items/3983234eee51dc036bb0](https://qiita.com/tt2004d/items/3983234eee51dc036bb0)

```
create table test (item1 int);
insert into test values(1);
insert into test values(2);
insert into test values(3);
insert into test values(4);
insert into test values(5);
insert into test values(6);
insert into test values(7);
insert into test values(8);
insert into test values(9);
insert into test values(10);
```

* 1 [Writer] Updateして不要タプルを作成

```
update test set item1 = '1' where item1 = '2';
SELECT relname, n_live_tup, n_dead_tup FROM pg_stat_user_tables; ※vacuumにより物理削除されるタプルがあるか確認
```

* 2 [Reader] 処理に時間のかかるSELECT

```
select count(*) from test a , test b , test c , test d , test e, test f, test g, test h, test i, test j, test k;
```

* 3 [Writer] VACUUMで更新行の物理削除を実行

```
VACUUM test;
```

* 4
* 5

これらはPostgreSQLの仕事

* 6 [Reader] 30秒後にSELECTがキャンセルされる

```
ERROR:  canceling statement due to conflict with recovery
DETAIL:  User was holding shared buffer pin for too long.
```

出た！やった！

## まとめ

今回の話「PostgreSQLを完全に理解した」

次回予告「全然わからない、俺は雰囲気でPostgreSQLをやっている」

## [おまけ]排他ロックの場合はどうなの？

参考にしたQiita記事で

> マスタでの排他ロック(AccessExclusiveLock)とスタンバイでの参照（AELを取得するとそれに対応するWALがでます）

このような興味深いことが書いてありました。

これもmax_standby_streaming_delay秒待つのかなーと思って実験してみました。

[https://www.postgresql.jp/document/9.6/html/explicit-locking.html](https://www.postgresql.jp/document/9.6/html/explicit-locking.html)

>DROP TABLE、TRUNCATE、REINDEX、CLUSTER、VACUUM FULL、（CONCURRENTLYなしの）REFRESH MATERIALIZED VIEWコマンドによって獲得されます。

AccessExclusiveLockは上記で取得されるらしいです。

ただこのAccessExclusiveLockを重い処理付きという条件で用意する方法が思いつかなかったので、今回は明示LOCKを利用しました。

## 仮説

|項番 | Master  |   Slave |
|:---|:--------|:--------|
|1   | Access Exclusive Lockを取得 |  |
|2   | WALを生成 | |
|3   | | WALを適用 |
|4   | | Lockを取得したテーブルに対してSELECT |
|5   | | 30秒後にSELECTがキャンセルされる |

まず明示的にLOCKを取得します。

LOCKをかけるようReaderに指示するWALが適用されます。

LOCKをかけたテーブルにSELECTをします。

30秒後にSELECTがキャンセルされます。

という寸法です。

## 実験

* 1 [Writer] Access Exclusive Lockを取得

```
BEGIN WORK;
LOCK TABLE test IN ACCESS EXCLUSIVE MODE; 
```

* 2
* 3

はPostgreSQLの仕事

* 4 [Reader] Lockを取得したテーブルに対してSELECT

```
select * from test;
```

* 5 [Reader] 30秒後にSELECTがキャンセルされる

されませんでした

されない模様です。奥さん。


