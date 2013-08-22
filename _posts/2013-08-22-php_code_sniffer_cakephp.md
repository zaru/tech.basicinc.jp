---
layout: post
category : CakePHP
title: PHP Code Snifferを使ってCakePHPのコーディング規約をチェックする
tagline: Composerで簡単インストール
author : sakuraba
tags : [CakePHP, PHP]
---
{% include JB/setup %}

コーディング規約…とても大事だけど、いつの間にか守られず数カ月後には、ぐちゃぐちゃなコードだらけになって頭を抱えるなんてことはザラにあったら困るけど、けっこう現実問題ありえるし、そんなの人的にチェックしてる暇ないし、コンピュータのほうが得意なので、そっちに任せるよ的なツール PHP Code Sniffer を使って CakePHP のコーディング規約をチェックしてみます。

ちなみに、CakePHPのコーディング規約はこちら。

[コーディング規約 — CakePHP Cookbook v2.x documentation](http://book.cakephp.org/2.0/ja/contributing/cakephp-coding-conventions.html)

## PHP Code Sniffer のインストール

PHP Code SnifferはPEARにあるので、PEARコマンドでインストール可能。さらに、CakePHPのコーディング規約ファイルはデフォルトだと入っていないので、CakePHPのリポジトリから取り込む。

```bash
% pear install PHP_CodeSniffer
% pear channel-discover pear.cakephp.org
% pear install cakephp/CakePHP_CodeSniffer
```

これだけでOK。

Composerを使っている人は、Composerで。Composerの導入はこちらを参照してくださいませ。[PHPのパッケージ管理Composerを使う - アプリケーションごとのライブラリ依存関係に悩まない](http://tech.basicinc.jp/php/2013/08/18/php_composer/)

### composer.json

```javascript
{
    "repositories":[{
        "type":"pear",
		"vendor-alias":"cakephp",
        "url":"http://pear.cakephp.org"
    }],
	"require": {
		"squizlabs/php_codesniffer": "dev-phpcs-fixer",
		"cakephp/CakePHP_CodeSniffer":"*"
	}
}
```
```bash
% composer install
Loading composer repositories with package information
Initializing PEAR repository http://pear.cakephp.org
Installing dependencies (including require-dev)
  - Installing squizlabs/php_codesniffer (dev-phpcs-fixer 85ac68f)
    Cloning 85ac68f990ecdd9465bd02d18d9951b10447693e

  - Installing pear-pear.cakephp.org/cakephp_codesniffer (0.1.2)
    Loading from cache
squizlabs/php_codesniffer suggests installing phpunit/php-timer (dev-master)
Writing lock file
Generating autoload files
```

これでプロジェクトディレクトリにPHP Code SnifferとCakePHP規約ファイルが入ります。ただし、CakePHP規約ファイルが所定の位置にないので、シンボリックリンクを張ります。

```bash
% cd ./vendor/squizlabs/php_codesniffer/CodeSniffer/Standards/
% ln -s ~/Desktop/hoge/vendor/pear-pear.cakephp.org/CakePHP_CodeSniffer/PHP/CodeSniffer/Standards/CakePHP/ CakePHP
```

### PHP Code Snifferの動作確認

```bash
% ./vendor/bin/phpcs -i
The installed coding standards are CakePHP, MySource, PEAR, PHPCS, PSR1, PSR2, Squiz and Zend
```

phpcs -i と打つと対応するコーディング規約が表示されます。その中に CakePHP があればOKです。

## PHP Code Snifferで実際にCakePHPのコーディング規約チェック

phpcs コマンドで実行。ソースコード量によっては、かなり時間がかかります。

./app/ ディレクトリ下のみを対象に実行してみます。--ignoreオプションでチェックしない名前も指定できます（ワイルドカード対応/ただし使ってるシェルによる）。

```bash
% vendor/bin/phpcs --report=summary --report-checkstyle=CakePHP --standard=CakePHP --extensions=php ./app

PHP CODE SNIFFER REPORT SUMMARY
--------------------------------------------------------------------------------
FILE                                                            ERRORS  WARNINGS
--------------------------------------------------------------------------------
/Users/zaru/Desktop/cakephp-2.3.9/app/Config/bootstrap.php      0       2
/Users/zaru/Desktop/cakephp-2.3.9/app/Config/core.php           0       10
/Users/zaru/Desktop/cakephp-2.3.9/app/Config/Schema/db_acl.php  1       0
/Users/zaru/Desktop/cakephp-2.3.9/app/Config/Schema/i18n.php    0       1
...s/zaru/Desktop/cakephp-2.3.9/app/Config/Schema/sessions.php  1       0
/Users/zaru/Desktop/cakephp-2.3.9/app/Console/cake.php          1       0
...ru/Desktop/cakephp-2.3.9/app/Controller/PagesController.php  2       0
/Users/zaru/Desktop/cakephp-2.3.9/app/webroot/index.php         0       1
/Users/zaru/Desktop/cakephp-2.3.9/app/webroot/test.php          0       1
--------------------------------------------------------------------------------
A TOTAL OF 5 ERRORS AND 15 WARNINGS WERE FOUND IN 9 FILES
--------------------------------------------------------------------------------
```

なんにもこっちでコード書いてないけど、エラーが出ている。というわけで、さらに詳しく見てみましょう。

```bash
% vendor/bin/phpcs --report=full --report-checkstyle=CakePHP --standard=CakePHP --extensions=php ./app/Controller/PagesController.php

FILE: /Users/zaru/Desktop/cakephp-2.3.9/app/Controller/PagesController.php
--------------------------------------------------------------------------------
FOUND 2 ERRORS AFFECTING 2 LINES
--------------------------------------------------------------------------------
 61 | ERROR | Variable "title_for_layout" is not in valid camel caps format
 70 | ERROR | Variable "title_for_layout" is not in valid camel caps format
--------------------------------------------------------------------------------
```

変数名がキャメルバックになってないよって事で怒られてますね。こんな感じで規約違反を見つけることが出来ます。簡単。出力フォーマットは、こういった full 表示の他にXMLやJSON、ファイル出力など細かく設定できます。

これ自体も手動で行うのではなくJenkinsなどのCIツールやGitのコミットフックで実行したり自動的にチェックするようにするのが良いでしょう。



