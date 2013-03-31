---
layout: post
category : jekyll
title: Jekyll + Jekyll Bootstrap + Gitでブログを構築する
tagline: "デプロイは普通のWebサーバ"
author : sakuraba
tags : [jekyll, twitter bootstrap, GitHub]
---
{% include JB/setup %}

株式会社ベーシック エンジニア 桜庭（[@zaru](https://twitter.com/zaru)）です。最近ベーシックで新卒・中途のエンジニア採用活動が活発になってきたこともありベーシックのエンジニアって、どんな事やってるんだろう…というのを外部にPRするために、エンジニアブログを作ってみることにしましたよと。

で、今回はWordPressとかのブログエンジンを使うのではなく、勉強もかねてJekyll + Jekyll Bootstrap + Gitな構成で作ってみます。

デプロイ先はGitHubPageではなく、いわゆる普通のWebサーバに配置します。なので、gitのHook（post-recieve）を使って、ローカルから更新をpushして、デプロイ先のサーバに自動でjekyllを使って生成してもらう感じです。

## Jekyllのインストール

Jekyllというのは、静的なサイトを作るためのRubyで動くジェネレータです。
なにはともあれJekyllをインストール。Macの場合。Windowsは知りません。

	% sudo gem install jekyll
	ERROR:  Error installing jekyll:
	liquid requires RubyGems version >= 1.3.7. Try 'gem update --system' to update RubyGems itself.

gemでサクッとインストールしようとしたらバージョンで怒られた。ので、gem自体のバージョンアップを。

	% gem update --system
	% gem -v
	2.0.3
	% sudo gem install jekyll

これで、Jekyllのインストールは完了。

## Jekyll Bootstrapのインストール

このまま素のJekyllを使って作ってもいいんだけど、ちょっと面倒なのでJekyll Bootstrapという便利なフレームワークを使います。

	% git clone https://github.com/plusjade/jekyll-bootstrap.git tech.basicinc.jp
	% cd tech.basicinc.jp
	% jekyll --server --auto

これで、[http://localhost:4000/](http://localhost:4000/) でサイトが確認できはず。あ、tech.basicinc.jp は適宜書き換えてください。

## シンタックスハイライトをするために

エンジニアブログなので、コードのシンタックスハイライトはやって欲しいというわけで、Pygmentsを入れてみる。

	% sudo easy_install Pygments

あとは、_config.ymlを編集。

	markdown: redcarpet
	pygments: true

これで、

	```php
	<?php
		echo 'hoge';
	```

という書き方でシンタックスハイライトされます。

## デプロイ先のサーバ設定

ローカルでのJekyll準備は整ったので、次はデプロイ先のサーバ設定。

	$ cd
	$ mkdir tech.basicinc.jp.git
	$ cd tech.basicinc.jp.git
	$ git --bare init

とりあえずgitのリポジトリ作ります。このリポジトリに対してローカルからpushしてデプロイします。そんで次は、hookでjekyll起動するようにスクリプトを書く。

	$ vi hooks/post-receiveo
	$ chmod 755 hooks/post-receive

```bash
#!/bin/sh

GIT_REPO=$HOME/tech.basicinc.jp.git
TMP_GIT_CLONE=$HOME/tmp/tech.basicinc.jp
PUBLIC_WWW=/var/www/tech.basicinc.jp/htdocs

git clone $GIT_REPO $TMP_GIT_CLONE
cd $TMP_GIT_CLONE
jekyll --no-auto $TMP_GIT_CLONE $PUBLIC_WWW
cd $HOME
rm -Rf $TMP_GIT_CLONE
exit
```

これであとはjekyllをインストールすれば、ほぼほぼ完成なんだけど、今回使用するサーバにはRubyGemsどころかRubyも入ってなかったので、おとなしくソースからインストールすることに。

## CentOSにRuby + RubyGems + Jekyllをインストール

	# wget ftp://ftp.ruby-lang.org/pub/ruby/1.8/ruby-1.8.7-p371.tar.bz2
	# tar jxvf ruby-1.8.7-p371.tar.bz2
	# cd ruby-1.8.7-p371.tar.bz2
	# ./configure
	# make
	# make install


	# wget http://rubyforge.org/frs/download.php/70696/rubygems-1.3.7.tgz
	# tar zxvf rubygems-1.3.7.tgz
	# cd rubygems-1.3.7
	# ruby setup.rb


	# gem install jekyll
	ERROR:  Loading command: install (LoadError)
	    no such file to load -- zlib
	ERROR:  While executing gem ... (NameError)
	    uninitialized constant Gem::Commands::InstallCommand

zlibがないと怒られたので…

	# yum install zlib-devel

make clean して再度Rubyをビルドしなおして（ついでにredcarpetも）インストール。

	# gem install jekyll
	# gem install redcarpet

シンタックスハイライトする場合、こっちにもPygmentsが必要なので、

	# yum install python-setuptools
	# easy_install Pygments

として入れておく。これでサーバの設定は完了。

## CentOSにPython2.7をインストールする

完了…とか思いきや、CentOSの場合はPythonのバージョン問題でエラーになるので、ソースから2.7系を入れる必要がある。（3.0系・2.4系はNG）

	Configuration from /home/hoge/tmp/tech.basicinc.jp/_config.yml
	Building site: /home/hoge/tmp/tech.basicinc.jp -> /var/www/tech.basicinc.jp/htdocs
	/usr/local/lib/ruby/gems/1.8/gems/pygments.rb-0.3.7/lib/pygments/popen.rb:357:in `get_header': Failed to get header. (MentosError)

こんな感じのエラーが出る。

ちなみに、yumなどはpythonで動いているので、最初から入っているPython2.4を消すと色々動かなくなるため注意。共存するようにインストールする。

	# yum install zlib zlib-devel tk-devel tcl-devel sqlite-devel ncurses-devel gdbm-devel readline-devel bzip2-devel db4-devel openssl-devel
	# wget http://python.org/ftp/python/2.7.2/Python-2.7.2.tgz
	# tar zxvf Python-2.7.2.tgz
	# cd Python-2.7.2
	# ./configure --with-threads --enable-shared --prefix=/usr/local
	# make
	# make install
	# ln -s /usr/local/lib/libpython2.7.so.1.0 /lib64/

## ローカルからpushでデプロイするために

ローカルのgitのリモートリポジトリに、デプロイ先を登録する。

	$ git remote add deploy hoge@example.com:piyo

あとは、pushでOK。

	$git push deploy master

これで、デプロイ先にjekyllで生成されたHTMLが配置されているはず。おつかれさまでした。