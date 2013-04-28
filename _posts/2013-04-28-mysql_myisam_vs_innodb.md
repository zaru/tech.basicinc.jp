---
layout: post
category : MySQL
title: MySQLのMyISAMとInnoDBパフォーマンス比較
tagline: テーブルロック vs 行レベルロック
author : sakuraba
tags : [MySQL]
---
{% include JB/setup %}

MySQLにはいくつかのストレージエンジンが存在していますが、二大巨頭MyISAMとInnoDBのどちらかを大抵選択していると思います。攻めのMyISAM、守りのInnoDBといったイメージがあるんですが（どこに）実際の所、どんなもんなのって思ったのでベンチマークしてみました。

とはいえ、テーブル設計・アプリケーション設計によってパフォーマンスなんて変わってくるので、あくまで一例として参考にしてください。

## ベンチマーク

### 条件

- MySQL5.1.44

### 同時に9個のコネクションでinsertしてupdateしてselectする

というわけで、そこそこの人気サービスと仮定し一つのテーブルに対して、insertしまくるスクリプト、updateしまくるスクリプト、selectしまくるスクリプトの3つを用意し、同時に3個分、計9個のスクリプトを同時に動かしてみます。

### insertのベンチマーク結果

![](/assets/img/2013-04-28-myisam_innodb1.jpg)

InnoDBの勝ち。

### updateのベンチマーク結果

![](/assets/img/2013-04-28-myisam_innodb2.jpg)

InnoDBの勝ち。

### selectのベンチマーク結果

![](/assets/img/2013-04-28-myisam_innodb3.jpg)

InnoDBの勝ち。

というわけで、全てにおいてInnoDBが勝ちました。やはりMyISAMはテーブルロックになるので、InnoDBの行レベルロックに比べると平行アクセス時のパフォーマンスは結構変わってきますね。

これだけ見ると、InnoDB最強とか思っちゃったりしますが、そんなことはないのが難しい所。更新処理が発生しない（ログ系とか）テーブルの場合は、MyISAMの方が高速になったりしますし、selectの条件によってはInnoDBは遅くなることもあります。

あと、InnoDBだからといって必ず行レベルロックになるわけじゃなく、テーブルロックになる条件も存在します。

**要は、ちゃんとパフォーマンステストしろと。**

ただ、いわゆるWebアプリケーションであれば、InnoDB標準と考えてもいいような気がしてます。

まぁ、そもそもトランザクションの有無とか他にも色々とエンジンの特性はあるので、パフォーマンスだけで決めるようなものではないですが。

### 計測スクリプト

実行バッチ

```bash
#!/bin/bash

php init.php
php insert.php > result_insert1.txt &
php update.php > result_update1.txt &
php select.php > result_select1.txt &
php insert.php > result_insert2.txt &
php update.php > result_update2.txt &
php select.php > result_select2.txt &
php insert.php > result_insert3.txt &
php update.php > result_update3.txt &
php select.php > result_select3.txt &
```

insert.php

```php
<?php
	$start = microtime(true);
	
	for ($i=0; $i<=10000; $i++) {
		mysql_query(sprintf('insert myisam (name) values(%d)', rand(1,100)));
		if ($i % 100 === 0) {
			echo sprintf('%0.10f', microtime(true) - $start) . "\n";
		}
	}
```

update.php

```php
<?php
	$start = microtime(true);
	
	for ($i=0; $i<=10000; $i++) {
		mysql_query(sprintf('update myisam set name = "%d" where id = %d limit 1', rand(1,1000), rand(1,1000)));
		if ($i % 100 === 0) {
			echo sprintf('%0.10f', microtime(true) - $start) . "\n";
		}
	}
```

select.php

```php
<?php
	$start = microtime(true);
	
	for ($i=0; $i<=10000; $i++) {
		mysql_query(sprintf('select id, name from myisam where name = "%d" limit 5', rand(1,100)));
		//mysql_query('select id, name from myisam');
		if ($i % 100 === 0) {
			echo sprintf('%0.10f', microtime(true) - $start) . "\n";
		}
	}
```
