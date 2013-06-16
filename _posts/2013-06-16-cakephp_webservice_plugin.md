---
layout: post
category : CakePHP
title: CakePHP2でJSON/JSONP/XMLのフォーマットを返すプラグイン作りました
tagline: forked from josegonzalez
author : sakuraba
tags : [CakePHP]
---
{% include JB/setup %}

CakePHPでWebAPIを作る事が多くて、そんな中とても重宝して使っているプラグインがあります。

[josegonzalez / webservice_plugin GitHub](https://github.com/josegonzalez/webservice_plugin)

このプラグインをいれて、ちょちょっと設定して、URLの後ろに .json / .xml と追加してアクセスすれば $this->set() した内容がそのまま構造化されて出力されます。かなり便利。超便利。死ぬほど便利。自前で実装しようとすると意外と面倒なので。

と、これそのままでも良かったんですが、突如JSONPにも対応しなくてはならず、フォークして機能追加＆特定のバージョンで動かないバグ修正をしました。

[zaru / webservice_plugin GitHub](https://github.com/zaru/webservice_plugin)

こちらからダウンロード出来ます。josegonzalezさんに感謝。

## JSONPでの出力方法

使い方は非常に簡単で、

	http://localhost/hoge/index.json?callback=コールバック名

と、callbackキーをGETで指定してあげればJSONP形式で返ってきます。

## WebServicePluginの使い方

GitHubのREADMEにも書かれていますが、簡単に使い方を紹介します。

### 初期設定

app/Config/bootstrap.php

	CakePlugin::load('Webservice');

app/Config/routes.php

	Router::parseExtensions('json', 'xml');

### コントローラの設定

app/Controller/HogeController.php

```php
<?php
App::uses('AppController', 'Controller');
class WebController extends AppController {
	public $name = 'Web';
	public $uses = array();
	public $components = array(
		'RequestHandler',
		'Webservice.Webservice'
	);
	
	public function index() {
		$this->set('Items', array('hoge' => 1, 'piyo' => 2));
	}
```

これで、 http://example.com/web/index.json (or .xml) とアクセスをするとそれぞれのフォーマットで返ってきます。

### JSONやXMLでアクセスさせたくないアクションの設定

ブラックリスト機能があります。アクション名を指定することで、そのURLでアクセスされた場合に405ステータスエラーを返します。また、ワイルドカードも使用出来ます。…まぁ使用しないと思いますが。

```php
<?php
//...
	public $components = array(
		'RequestHandler',
		'Webservice.Webservice' => array(
			'blacklist' => array('home', 'index')
		)
	);
```

### 出力させたくない変数の設定

アクションと同様に、変数自体も出力しないように設定出来ます。CakePHPが自動でセットしてくるような変数を指定してあげましょう（モデルのバリデーション変数とか）。

```php
<?php
//...
	public $webserviceBlacklistVars = array(
		'Items'
	);
```

また、このプラグインCakePHP2.2.x以上？だと、正常に動かなかったので、ちょっとだけ修正してあります。

