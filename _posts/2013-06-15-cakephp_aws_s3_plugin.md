---
layout: post
category : AWS, CakePHP
title: CakePHP2でAWS S3を操作するプラグインを作りました
tagline: 
author : sakuraba
tags : [AWS, S3, CakePHP]
---
{% include JB/setup %}

AmazonWebServicesのS3、とても便利ですよね。便利だし、安いし。パフォーマンスは…まぁそこそこ。

最近、AWSのS3を画像ファイルのホスト先として使う機会が多くて、Amazonから提供されているAWS SDK fro PHPを使ってガリガリコードを書いていたんだけど、初期設定が面倒になったのでCakePHPのプラグインとしてまとめました。SDK使っているので、コード量自体は最初からスリムなんだけどね。

ダウンロードはこちらから。
[CakePHP2 AWS S3 DataSource](https://github.com/zaru/Cakephp2_AWS_S3_DataSource)

使い方は非常に簡単で、プラグインファイルを設置して、database.phpにAWSのキーなどを設定して、適当なモデルから呼び出すだけです。これを使えばコントローラからファイル操作はもちろん、モデルの中からでも簡単に操作出来ます。

機能はファイルアップロード／削除／移動／コピーだけのシンプル構成。使い方は、GitHubの方を参照してください。

それにしても、CakePHP2のデータソースの作り方が全然まともに解説されていなくて死んだ。なんかバージョンによって構成が全然違うし。他のデータソースプラグインを参考に作ったのだけど、どうやっても DataSource::query() っていうquery関数が最初に呼び出されて、そこからハンドリングする方法しか分からなかった。

なんか、もっと良い記述方法がある気がするんだけどなー。元のソースを完全に追う気力がない…。

