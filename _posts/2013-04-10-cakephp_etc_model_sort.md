---
layout: post
category : CakePHP
title: CakePHPのPaginateでjoinしたモデルのカラムでソートする方法
tagline: 
author : sakuraba
tags : [PHP, CakePHP]
---
{% include JB/setup %}

CakePHPのPaginate便利ですよね。特に設定しなくても、ソートもページングもできる。素晴らしい。ただ、joinしているモデルのカラムをキーにソートしようとすると効かないという…。どうしよう。

答え→**バーチャルフィールドを設定する**

解決方法は簡単で、ソートしたいカラムをバーチャルフィールドとして登録するだけです。

model.php

```php
<?php
class Hoge extends AppModel {
	var $virtualFields = array(
		‘hoge’ => ‘JoinTable.hoge’
	);
```

view.ctp

```php
<?php echo $this->Hoge->sort(‘hoge’,’ソートキー’); ?>
```

ただし、ソートのためだけにバーチャルフィールドを設定するとfindした時にうざったいことになる可能性があるので、必要な時だけに設定するのが吉。

ちなみに、なぜjoinしているモデルではソートできないのかというと、URLに適当なキーを直接入力してエラーになるのを防ぐためらしい。

