---
layout: post
category : PHP
title: AWS SQSをSDK ver1を使ってPHPから実行するサンプル
tagline: 
author : sakuraba
tags : [PHP, AWS, SQS]
---
{% include JB/setup %}

PHPでキューイングしたいなぁと思って、色々と調べていたらいくつか定番的なのがあった。まぁ、PHP関係ないのも含め。

- [PHP+Kestrel+Supervisorでお手軽タスクキューイング : アシアルブログ](http://blog.asial.co.jp/875)
- [PHP - ZendQueueとKestrelでメッセージキューサーバーを体験 - Qiita [キータ]](http://qiita.com/NAKANO_Akihito/items/241869eaa857ac843a65)
- [GearmanをPHPから使ってみた。 - 個人事業主のつぶやき](http://d.hatena.ne.jp/toshiyuki_saito/20110128)
- [Cake Resque | CakePHP plugin for creating background jobs that can be processed offline later](http://cakeresque.kamisama.me/)

けっこう手軽に導入できる。ただ今回は開発環境やコストから考えて、AWSのSQSを使ってキューイングすることにした。AWS安すぎるだろ…。

## AWS SDK ver1からSQSを使う

諸事情有り、SDKはver1を使用している。

### request.php

```php
<?php
require_once ('Vendor/aws_sdk/sdk.class.php');
 
$sqs = new AmazonSQS();
$region = AmazonSQS::REGION_APAC_NE1;
$sqs->set_region($region);
$queueName = 'キュー名';
$queueURL = $sqs->create_queue($queueName)->body->CreateQueueResult->QueueUrl;
$data = $sqs->send_message($queueURL, '値' . time());
if ($data->isOK()) {
    echo 'OK';
}
```

### receive.php

```php
<?php
require_once('/Vendor/aws_sdk/sdk.class.php');
 
$sqs = new AmazonSQS();
$region = AmazonSQS::REGION_APAC_NE1;
$sqs->set_region($region);
$queueName = 'キュー名';
$queueURL = $sqs->create_queue($queueName)->body->CreateQueueResult->QueueUrl;
 
while (true) {
    $data = $sqs->receive_message($queueURL);
    if ($data->isOK()) {
        if (isset($data->body->ReceiveMessageResult->Message)) {
            $msgData = $data->body->ReceiveMessageResult->Message;
            $msg = $msgData->Body;
            echo $msg . "\n";
            
            // 削除
            $handle = (string)$msgData->ReceiptHandle;
            $sqs->delete_message($queueURL, $handle);
        } else {
            sleep(1);
        }
    } else {
        echo $data->body->Error->Message . "\n";
    }
}
```

receive.php を無限ループ状態にして、キューがあるか確認し、あればメッセージを受け取って、何らかの処理をするという感じ。なければ、1秒間寝かせる。

実際に運用をするには、プロセス管理ツールを使った方がいい。ここらへんはまた今度検証する。