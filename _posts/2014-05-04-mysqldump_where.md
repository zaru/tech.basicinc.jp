---
layout: post
category : MySQL
title: mysqldumpオプションまとめ。whereで条件を指定したりする
tagline: 
author : sakuraba
tags : [MySQL]
---
{% include JB/setup %}

![mysqldump](/assets/img/2014-05-04-mysql-dump.png)

mysqldumpコマンドを使ってMySQLのデータをダンプする毎日を送っている[zaru](https://twitter.com/zaru)です。ダンプしまくるのはいいんだけど、実際に必要なデータって全部じゃなかったりするわけです。というわけで、mysqldumpのよく使うパターンをまとめてみた。

## 指定テーブルだけダンプする

まぁ、まず基本。データベース丸ごとじゃなくて指定したテーブルだけダンプしたい。

```
$ mysqldump -u user DB名 テーブル名A テーブル名B > dump.sql
```

単純にデータベース名の後ろに欲しいテーブル名を記載するだけ。

## テーブル作成情報は必要ない

上記のダンプだと、drop table + create table文も同時に作成される。これらが必要ない場合は「-t」オプションを使う。

```
$ mysqldump -u user -t DB名 テーブル名A テーブル名B > dump.sql
```

これで純粋にinsert文だけ取れる。

## 逆にテーブルの構成情報・スキーマだけ欲しい

結構よく使う。create table文のみ。レコード情報は必要ない場合。「--no-data」オプションを使う。「-d」の短縮でもOKだけど、なんかdropしそうで怖いので、あえて--no-dataの方を使っている。

```
$ mysqldump -u user --no-data DB名 > dump.sql
```

## whereで指定したレコードのみダンプしたい

「--where（-w）」オプションを指定すると、whereが使える。指定テーブルだけではレコード数が多すぎて困るという場合や、予め必要なデータだけに絞ってダンプしたい場合に使う。かなり便利。

```
$ mysqldump -u user DB名 --where 'is_delete = 0' > dump.sql
```

ちなみに、複数テーブルあった場合、すべてのテーブルに対して同じwhereを適用してくれる。

## XMLフォーマットでダンプする

「--xml（-X）」オプションを指定すると、XMLフォーマットでダンプする。あまり使う場面はないけど。

```
$ mysqldump -u user DB名 --xml > dump.xml
```

## その他

--optオプションというダンプデータを高速にインポートできるように最適化するものがあるが、MySQL5.1あたりくらいからデフォルトで有効になっているので指定する必要ない。
