---
layout: post
category : Wordpress
title: Wordpress用の.gitignoreを考える
tagline: 既にあるけど
author : kobayashi
tags : [Wordpress, Git]
---
{% include JB/setup %}
wordpressでサイトつくるとことが増えてきたしgitで管理しようと思って

「ついでにwordpress用の.gitignoreを考えよう」

と思ったら既にgithubにあがってた。そりゃそうか。

[https://github.com/github/gitignore](https://github.com/github/gitignore)

なのでこれを使うことにしようと思ったのですが、
何も考えずに使うのもあれなので、それぞれどういうものかを見ておきます。


### ■ .htaccess
本番環境とテスト環境がURLが違う場合ほとんどなので、.htaccessはバージョン管理しません。



### ■ wp-config.php 
データベース設定とか、環境依存の設定があるのでこれも当然外す。



### ■ wp-content/uploads/
管理画面から画像とかアップロードしたときの保存先ディレクトリ。



### ■ wp-content/blogs.dir/
v3.5より前で、マルチサイト運用するときに使うようです。
多分もういらない。



### ■ wp-content/upgrade/
アップグレード処理をするときに一時的に使う・・・のかな



### ■ wp-content/backup-db/
Wordpressをバックアップするプラグイン WP-DB Managerを使った場合のバックアップ先ディレクトリ



### ■ wp-content/wp-cache-config.php、wp-content/advanced-cache.php
プラグイン：WP Super Cacheで使う



### ■ sitemap.xml、sitemap.xml.gz
Webマスターツールに登録する用のsitemap.xml
作成しないとありません。



### ■ *.log
ログ系は必要ないので外す



### ■ wp-content/cache/
プラグイン：WP-Cacheで使う



### ■ wp-content/backups/ 
Wordpressをバックアップするプラグイン Wordpress Backup to Dropboxでのバックアップ先ディレクトリ

