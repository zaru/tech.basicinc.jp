---
layout: post
category : MySQL
title: MySQLで処理に長時間かかっている複数クエリをまとめて殺す方法
tagline: 
author : sakuraba
tags : [MySQL]
---
{% include JB/setup %}

あまりにも処理に時間がかかるようなSQLを実行してしまい、MySQLがうんともすんとも言わなくなってしまうような状況、よくありますよね。っていうか、まぁそんな状況あってはならないんですが、時たまあります。そんな時、問題となっているクエリの処理を止めたいわけです。

## 特定のクエリを止める方法

[MySQLで実行中のクエリ一覧を見て、SQLを強制終了する方法](http://tech.basicinc.jp/MySQL/2013/04/20/mysql_kill_process/)

こちらを見てもらえればやり方は分かります。単純にMySQLに入って、show processlist;で問題のあるクエリを発見し、プロセスIDを kill するだけ。とても簡単。

## 複数のクエリを一括で止める方法

今回は問題のあるクエリが100個あったらどうする…？的なのを解決するエントリーです。まぁ、問題あるクエリ100個ある状況は、アプリ的に問題あるんじゃね？っていうレベルですが。

1個ずつプロセスIDをコピペして…なんてやってられないですよね。まとめてkillしちゃいたい。実は、show full processlistは以下のSQLと同じ。

```sql
SELECT * FROM information_schema.PROCESSLIST;
```

ということは、普通にwhere句などで条件が指定できる。

### 処理時間に60秒以上かかっているのを表示

```sql
SELECT * FROM information_schema.PROCESSLIST WHERE TIME > 59;
```

ということは、IDも1行にまとめられる。

```sql
SELECT GROUP_CONCAT(ID) FROM information_schema.PROCESSLIST WHERE TIME > 59;
+------------------+
| group_concat(ID) |
+------------------+
| 1,2,3,4,5             |
+------------------+
```

GROUP_CONCAT()関数を使うことで、複数レコードをカンマ区切りの1行にまとめられます。こいつはいいぜ。

### 複数のプロセスIDをkillする

あとは、こいつをkillすれば良いだけなんですが、MySQL内のkillコマンドでは複数IDを受け付けてくれません。

```sql
kill 1,2,3; # これは駄目
```

というわけで、いったんMySQLから抜けて、普通のコンソールからmysqladminコマンドを使用します。mysqladminのkillはカンマ区切りで複数のIDを受け付けてくれます。心が広い。

```
$ mysqladmin kill 1,2,3 -h localhost -u hoge
```

汚物は消毒だ～っ！！ばりのノリで殺してくれます。これでいっときの安寧が訪れますが、そもそもの原因となっているSQLやアプリケーションを修正しないと駄目ですね。はい。