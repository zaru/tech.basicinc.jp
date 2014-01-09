---
layout: post
category : CakePHP
title: PHPUnitをComposerでインストールしてCakePHP2.2系で使う時の注意
tagline: 
author : sakuraba
tags : [CakePHP, PHP, PHPUnit]
---
{% include JB/setup %}

いまどきのPHPUnitインストールはComposer使っていると思います。まぁ、いまどきなCakePHPならそれで問題ないです。ただ、微妙に古いCakePHPを使っているとPHPUnitが入ってないよって判定食らって test.php にアクセスしてもダメだったりします。（autoload自体はされているので使えるはずだけど）

## Composer経由PHPUnitへの正式対応はCakePHP2.3.2以降から

PHPUnitがあるかどうか判定するメソッド loadTestFramework の履歴を見るとここで対応しているっぽいです。

[Be compatible with PHPUnit installed with composer. · 386be52 · cakephp/cakephp · GitHub](https://github.com/cakephp/cakephp/commit/386be52c71a34621d60373a29c19f178864f0223)

### lib/Cake/TestSuite/CakeTestSuiteDispatcher.php

```php
<?php
// snip
public function loadTestFramework() {
	if (class_exists('PHPUnit_Framework_TestCase')) {
		return true;
	}
	// snip
}
```

これでautoloadされているPHPUnitが判定されます。まぁ、CakePHP自体をアップデートできるような環境の人は素直にアップデートした方がいいと思います。

## おまけ：PHPUnit Composerインストールメモ

### composer.json

```javascript
{
	"require-dev": {
	    "phpunit/phpunit": "3.7.*"
    },
	"config": {
		"vendor-dir": "htdocs/app/Vendor/Composer"
	}
}
```

### app/Config/bootstrap.php

```php
<?php
// snip
App::import('Vendor', 'Composer'.DS.'autoload');
```

```bash
$ composer install
```

これだけ。vendor-dir でインストール場所を指定できる。
