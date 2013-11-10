---
layout: post
category : munin
title: MySQLをmuninでリソース監視する
tagline: 
author : sakuraba
tags : [munin, MySQL]
---
{% include JB/setup %}

[AWS RDSをMuninでリソース監視する - Muninプラグインの作り方](http://tech.basicinc.jp/munin/2013/08/29/munin_aws_rds/)

RDS特有のCPUリソースとかコネクション数とかをCloudWatchで監視する方法は以前まとめたのだけど、そういえばMySQL標準で用意してくれているmuninプラグインの導入については書いてなかったと思い、忘れないようにまとめ。

### MySQL Develをインストール

```bash
yum install mysql-devel
```

### プラグインの配置

```bash
ln -s /usr/share/munin/plugins/mysql_ /etc/munin/plugins/mysql_commands
ln -s /usr/share/munin/plugins/mysql_ /etc/munin/plugins/mysql_innodb_bpool
ln -s /usr/share/munin/plugins/mysql_ /etc/munin/plugins/mysql_innodb_io
ln -s /usr/share/munin/plugins/mysql_ /etc/munin/plugins/mysql_innodb_log
ln -s /usr/share/munin/plugins/mysql_ /etc/munin/plugins/mysql_innodb_tnx
ln -s /usr/share/munin/plugins/mysql_ /etc/munin/plugins/mysql_select_types
ln -s /usr/share/munin/plugins/mysql_ /etc/munin/plugins/mysql_table_locks
```

### MySQLへの接続情報を設定

```bash
vi /etc/munin/plugin-conf.d/munin-node
```

```bash
[mysql*]
env.mysqlconnection DBI:mysql:mysql;host=www.example.com;port=3306
env.mysqluser DBUSER
env.mysqlpassword PASSWORD 
```

### 動作確認

```bash
cd /etc/munin/plugins
/usr/sbin/munin-run mysql_commands
```

これで正常に値が返ってくればOK。