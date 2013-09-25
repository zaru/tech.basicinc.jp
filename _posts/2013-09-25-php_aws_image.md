---
layout: post
category : php
title: AWS SDK for PHP2を使ってEC2インスタンスのイメージを定期的に作成
tagline: バックアップ大事
author : kobayashi
tags : [PHP, AWS]
---
{% include JB/setup %}

EC2でサイトを運営することが多くなってきました。
EC2の便利な所として、AMIを使って簡単に環境を複製できる所がありますね。
そこでせっかくなのでEC2のイメージを定期的に自動作成してバックアップの変わりにできるようにしてみます。（｀・ω・´）キリッ！


## 準備
今回の作成は「AWS SDK for PHP2」を使います。
[http://aws.amazon.com/jp/sdkforphp2/](http://aws.amazon.com/jp/sdkforphp2/)

上記URLにアクセスして「Download AWS PHP SDK」ボタンをクリックでaws.pharファイルが取得できるので、これを読み込んでコードを書いてきます。ちなみにpharはphp archiveの略のようです。

AWS SDK for PHP2はComposerを使ってコード管理してます。
Composerについては[こちら](http://tech.basicinc.jp/php/2013/08/18/php_composer/)で解説してますので、参考にどぞ

## プログラムを書いてみる
```php
<?php
	require("aws.phar");
	
	use Aws\Ec2\Ec2Client;
	use Aws\Common\Enum\Region;
	use Aws\Common\Aws;	
	
	define("AWS_KEY", '<AWSキー>');
	define("AWS_SECRET_KEY", '<AWSシークレットキー>');
	define("INSTANCE_ID", "<バックアップ対象のインスタンスID>");

	define("IMAGE_DELETE_TARGET", '削除対象となるイメージの作成日'); //-3 day みたいな感じで指定
	define("SITE_NAME", '<サイト名>');
	define("ADMIN_MAIL", '<エラー時のメール宛先>');
	define("ERROR_MAIL_TITLE", '<エラーメールタイトル>');

	$client = Ec2Client::factory(array(
		'key'    => AWS_KEY,
		'secret' => AWS_SECRET_KEY,
		'region' => Region::TOKYO
		)
	);
	
	//イメージ作成
	//ここで指定するNameはコンソール上でAMI Nameにあたる
	$args = array(
		'InstanceId'  => INSTANCE_ID,
		'Name'        => SITE_NAME . '_' . date('Y-m-d'),
		'Description' => SITE_NAME . 'backup create ' . date('Y-m-d'), 
	);
	try {
		$result = $client->createImage($args);
	}catch(Exception $e) {
		errorHandle($e);
	}
	
	$create_image_id = $result['ImageId'];
	
	//タグの作成
	$args = array(
		'Resources' => array($create_image_id),
		'Tags' => array(
				array(
					'Key'   => 'Created',
					'Value' => date('Y-m-d')
				),
				array(
					'Key'   => 'Name',
					'Value' => SITE_NAME
				)
			)
	);
	try {
		$result = $client->createTags($args);	
	}catch(Exception $e) {
		errorHandle($e);
	}

	//イメージの検索
	$delete_date = date('Y-m-d', strtotime(IMAGE_DELETE_TARGET));
	$search_ami_name = SITE_NAME . '_' . $delete_date;
	$filters = array(
		array(
			'Name'   => 'tag:Created',
			'Values' => array($delete_date)
		),
		array(
			'Name'   => 'name',
			'Values' => array($search_ami_name)
		),
	);
	$args = array(
		'Owners'  => array('self'),
		'Filters' => $filters 
	);
	try {
		$images = $client->describeImages($args);
	}catch(Exception $e) {
		errorHandle($e);
	}

	//イメージ削除
	//ヒットしたイメージは1件だけのはず
	if(count($images['Images']) == 0) {
		//error!
		echo 'No result  AMI Name: ' . $search_ami_name;
		mail(ADMIN_MAIL, ERROR_MAIL_TITLE, 'No result  AMI Name: ' . $search_ami_name);
		exit;
	}
	$delete_image_id = $images['Images'][0]['ImageId'];
	$args = array(
		'ImageId' => $delete_image_id
	);
	try {
		$result = $client->deregisterImage($args);
	}catch(Exception $e) {
		errorHandle($e);
	}
}

//エラー時の処理
//標準出力してメール飛ばしてるだけだけど
function errorHandle($e) {
	echo $e->getMessage();
	mail(ADMIN_MAIL, ERROR_MAIL_TITLE, $e->getMessage());
	exit;
}

?>
```

### 最初に宣言

最初に宣言部分でaws.pharを読みこんでおきます。
後はEC2インスタンスを操作用とリージョン指定の定数とAWS全般の共通部品用に読み込んでます。
この辺りはドキュメントを参考

[http://docs.aws.amazon.com/aws-sdk-php-2/latest/](http://docs.aws.amazon.com/aws-sdk-php-2/latest/)

AWSキーとかAWSシークレットキーは

[https://console.aws.amazon.com/iam/home?#users](https://console.aws.amazon.com/iam/home?#users)

上記URLでユーザーを作ると確認できるはず。後ユーザーを作った後はユーザーに対してEC2へのFullAccess権限の付与を忘れずに！

権限付与しないとPermissionエラーになっちゃいます。

### イメージとタグの作成
createImageメソッドでイメージを生成、createTagsメソッドでタグを生成します。

イメージを作成しても作成日は分からないので、作成日をタグとして生成しておきます。

後普通にイメージを作成するとawsのコンソール上でNameの欄が空になってしまいます。(createImageで指定するNameはAMI Nameです)コンソール上のNameの値はタグのNameキーの値を見ているので、Nameキータグを作って、適当な値をいれときます。

この辺たいした問題ではないんだけどちょっとややこしいね。。。(；一_一)

### イメージの削除
一定の日数がたったイメージは削除するようにします。

describeImagesメソッドでイメージを検索して、ヒットしたイメージをderegisterImageメソッドで消してます。

describeImagesで検索するときに上のプログラムではAMI Nameで検索してるんですが、キーの指定は「name」と小文字でなければならないです。タグの方のNameで検索するときは、tag:NameとやればOK

あとはこのphpをcronで定期的に実行してあげればいいです。
