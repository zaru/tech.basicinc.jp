---
layout: post
category : fluentd
title: fluentdで複数のnginxサーバログを収集しmongoDBに格納する
tagline: 
author : sakuraba
tags : [fluentd, mongoDB]
---
{% include JB/setup %}

![fluentd](/assets/img/2013-10-02-fluentd.png)

一つのサービスで、負荷分散のために複数のWebサーバ（nginx）を運用している場合、アクセスログなどがバラバラに散らばっていて、ログからなにか調べようと思ったり、統計をとったりするのが面倒だったりします。

そこで、ここ最近はやっているfluentdを導入してみました。うわさ通り簡単。

![title](/assets/img/2013-10-02-fluentd2.png)

イメージはこんな感じ。

## fluentdのインストール

ログを送信する側と、受信する側、両方にfluentdが入っている必要があります。

今回はAWSのディストリだったので、yum経由でインストール。公式サイトに記載されている下記コマンド一発で完了です。他にもapt-get / gem / brewでも簡単に入れられます。

```bash
curl -L http://toolbelt.treasure-data.com/sh/install-redhat.sh | sh
```

[Installing Fluentd Using rpm Package | Fluentd](http://docs.fluentd.org/articles/install-by-rpm)

起動や停止は自動的にサービス化されています。

```bash
service td-agent start | stop | restart | status
```

## nginxのログを送信する

### /etc/td-agent/td-agent.conf

```bash
<source>
  type tail
  path /var/log/nginx/access.log
  format /^(?<remote>[^ ]*) (?<host>[^ ]*) (?<user>[^ ]*) \[(?<time>[^\]]*)\] "(?<method>\S+)(?: +(?<path>[^ ]*) +\S*)?" (?<code>[^ ]*) (?<size>[^ ]*)(?: "(?<referer>[^\"]*)" "(?<agent>[^\"]*)" "(?<forwarder>[^\"]*)") (?<requesttime>[^ ]*)?/
  time_format %d/%b/%Y:%H:%M:%S %z
  tag nginx.access
  pos_file /var/log/td-agent/nginx.pos
</source>
<match nginx.access>
  type forward
  buffer_type memory
  buffer_chunk_limit 256m
  buffer_queue_limit 128
  flush_interval 5s
  <server>
    # ログの送信先IPアドレス
    host 192.168.1.100
    port 24224
  </server>
</match>
```

ログファイルのフォーマットに関しては、apacheなどのプリセットされているものもありますが、正規表現を使ってオリジナルフォーマットに対応させることも出来ます。

今回は、nginxのログフォーマットを改造しているので、独自のフォーマットとして登録します。

## nginxのログを受信する

上記サーバで type forward と設定したので、収集したログは外部サーバへ投げるようになっています。投げる先は例だと 192.168.1.100 になります。

ポートはデフォルトだと24224のTCP/UDPを使います。データ送信はTCP、サーバの死活監視用にUDPを使うみたいです。

### /etc/td-agent/td-agent.conf

```bash
<source>
  type forward
  port 24224
</source>
<match nginx.access>
  type file
  path /var/log/td-agent/access_log
</match>
```

まず、物理ファイルに出力する設定です。これで基本的には設定完了。あとはtd-agentを起動すれば動き出すはず。/var/log/td-agent/access_log に対して、どんどんログが追記されてくると思います。

ログの出力タイミングは flush_interval で調整できます。デフォルトだと60秒だった…かな。今は5秒くらいでやっていますが、特に問題なく動いています。

## mongoDBのインストール

物理ファイルじゃなくて、mongoDBに突っ込んで色々やりたいので、インストールしてみます。yumでインストールするために、リポジトリを登録します。

### /etc/yum.repos.d/mongodb.repo

```bash
[mongodb]
name=MongoDB Repository
baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/
gpgcheck=0
enabled=1
```

そしてyumでインストール。

```bash
sudo yum -y install mongo-10gen mongo-10gen-server
```

fluentdのプラグインも必要なので下記コマンドでインストール。

```bash
/usr/lib64/fluent/ruby/bin/fluent-gem install fluent-plugin-mongo
```

mongoDBの起動。

```
sudo service mongod start
```

## fluentdの出力先をmongoDBに変更する

ログ受信サーバ側の fluentd の設定を下記のように変更します。

```bash
<source>
  type forward
  port 24224
</source>
<match nginx.access>
  type mongo
  database nginx
  collection access
  host 127.0.0.1
  port 27017
</match>
```

これで td-agent を再起動すれば、mongoDBにデータが溜まるはず。

```bash
$ mongo
> use nginx
> db.access.find()
```

こんな感じでデータが見られると思います。今回はじめてmongoDBを使いましたが、すごいサクサク。ココらへんももっと勉強しないとなー。
