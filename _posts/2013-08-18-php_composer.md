---
layout: post
category : php
title: PHPのパッケージ管理Composerを使う
tagline: アプリケーションごとのライブラリ依存関係に悩まない
author : sakuraba
tags : [PHP]
---
{% include JB/setup %}

PHP界隈で静かに盛り上がっているらしいパッケージ依存管理ツール[Composer](http://getcomposer.org/)。正直、今まで本格的に使ったことはなく普通にPEARで直接インストールしてたりしたんですが（そもそもPEAR自体そんなに使ってなかった）、アプリケーション…というかフレームワークによって必要なバージョンが違うケースが多いため、試しに導入をしてみましたメモ。

あ、Composerを使うためにはPHP5.3.2以上が必要です。まぁ、今どきそれより古い環境がメインってこともない…と思いたい。

## Composerのインストール

ローカルMacに[Composer](http://getcomposer.org/)をインストールします。

```bash
% curl -sS https://getcomposer.org/installer | php
#!/usr/bin/env php
Some settings on your machine make Composer unable to work properly.
Make sure that you fix the issues listed below and run this script again:

The detect_unicode setting must be disabled.
Add the following to the end of your `php.ini`:
    detect_unicode = Off

A php.ini file does not exist. You will have to create one.
If you can not modify the ini file, you can also run `php -d option=value` to modify ini values on the fly. You can use -d multiple times.
```

公式サイトに書いてあったコマンドを実行すると、detect_unicodeの設定がねぇよって怒られるので、かといってMac標準だとphp.ini自体も置いてないので（/private/etc/php.ini.defaultにある）、アドバイス通り -d オプションで直接指定してあげます。

```bash
% curl -sS https://getcomposer.org/installer | php -d detect_unicode=Off
#!/usr/bin/env php
All settings correct for using Composer
Downloading...

Composer successfully installed to: /Users/zaru/composer.phar
Use it: php composer.phar
```

無事インストール完了。 php composer.phar と実行すると色々とオプションを表示してくれます。このままでも良いんですが、毎回 php composer.phar と打つのは（しかもパスを意識しつつ）面倒なので、aliasもしくは、バイナリファイルをパスの通った場所にコピーします。

```bash
% mv composer.phar /usr/local/bin/composer
```

これで、 composer というコマンドだけでOK。

## Composerの使い方

Composerは、簡単にいえば指定ディレクトリの中だけでパッケージ管理してくれるツールです。なので、まずはアプリケーションのルートディレクトリにでも移動します。

```bash
% cd ~/Sites/sample-app/
```

### composer.jsonでパッケージ管理

composer.jsonというパッケージ管理の設定ファイルをアプリケーションのルートディレクトリに設置します。

```javascript
{
    "require": {
        "monolog/monolog": "1.2.*"
    }
}
```

```bash
% composer install
```

これだけで、後は自動的にmonologのバージョン 1.2.* 以降をダウンロードしてくれます。完了後、/verndor/ というディレクトリが生成され、その中にある autoload.php をアプリケーションファイル上で読みこめば自動的に monolog のライブラリ読み込みがされます。

例えば、追加で PHP_Codesniffer が必要になったとした場合は、composer.json に追記をして update をかけます。（installではない）

```javascript
{
    "require": {
        "monolog/monolog": "1.2.*",
        "squizlabs/php_codesniffer": "dev-phpcs-fixer"
    }
}
```

```bash
% composer update
```

非常に簡単。

## ライブラリの探し方

[Packagist](https://packagist.org/)というサイトでライブラリを検索できます。

![title](/assets/img/2013-08-18-php_composer.jpg)

Packagistに登録されていないライブラリ（例えばPEARなど）は、composer.jsonにリポジトリを登録すれば使うことが出来ます。

```javascript
{
    "repositories":[{
        "type":"pear",
        "url":"http://pear.php.net"
    }],
    "require": {
        "monolog/monolog": "1.2.*",
        "squizlabs/php_codesniffer": "dev-phpcs-fixer",
        "pear-pear/HTTP_Request2":"2.1.*"
    }
}
```

以上です。

