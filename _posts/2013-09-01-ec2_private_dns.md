---
layout: post
category : ec2
title: EC2インスタンスのIPアドレスを自動でローカルDNSに登録する
tagline: dnsmasqかhostsファイルを使うよ
author : sakuraba
tags : [EC2, dnsmasq]
---
{% include JB/setup %}

なんか最近AWS関連のスクリプトばっかり書いている気がしますが…今回はEIPを割り当てていないEC2インスタンスへの接続を簡単にするための仕組みを作りました。EC2は、EIPで固定IPを割り当てていない限り、START / STOPをするとIPアドレスが変わってしまうので、SSHなどで接続しに行くのにちょっと面倒だったりします。それをローカルで簡単に名前解決してあげようという趣旨です。

ローカルのhostsファイルを直接変更するモードと、Mac版 dnsmasqを利用するモードがあります。必要に応じて使い分けてくださいませ。ちなみに、ローカルPC直接じゃなくても踏み台サーバとかでやっても良いと思います。というか、そっちのがチーム開発時には便利と思う。

## EC2HOSTの使い方

[zaru/EC2HOST](https://github.com/zaru/EC2HOST)

スクリプトはGitHubに置いてあります。AWS SDKを使用しているので認証用のキーを用意してください。

```bash
git clone https://github.com/zaru/EC2HOST.git
cd EC2HOST
cp config.php.sample config.php
vi config.php
```

### config.php の中身

```php
<?php
$config = array(
  'key' => '認証キー',
  'secret' => 'シークレットキー',
  'region' => 'ap-northeast-1', // Tokyo
);
```

### AWS SDK for PHP のインストール

AWS SDKをインストールするにはComposerを使います。インストールしていない人は[PHPのパッケージ管理Composerを使う - アプリケーションごとのライブラリ依存関係に悩まない](http://tech.basicinc.jp/php/2013/08/18/php_composer/)を参考にしてみてください。また、Composerとか入れたくないっていう人は、本家AWS SDKページにpharファイルがあるので、そちらを落としてきても構いません。

```bash
git submodule init
git submodule update
cd aws-sdk-php
composer install
```

### EC2HOST の実行

/etc/hosts用の出力モードと、dnsmasq用の出力モードがあります。標準はdnsmasqです。

```bash
php ec2host.php [hosts]
```

これで、それぞれに応じたHOST - IPの文字列が表示されると思います。

### ホスト名のルール

出力されるホスト名は、各EC2インスタンスのタグを見ています。ホスト名 = Group.Name です。Groupキーが存在しない場合は、Nameだけになります。

### /etc/hosts へ書き出す

```bash
sudo sh -c 'php ec2host.php host > /etc/hosts'
```

こんな感じで出力できるはず。

### dnsmasq へ書き出す

Mac用のシェルスクリプトを用意してあります。このスクリプトで、レコードファイル書き出し＆dnsmasqの再起動まで行ってくれます。

```bash
sudo ./load_dnsmasq.sh
```

使い方はこれで以上です。あとは、AWSの使い方に応じて毎時自動実行するもよし、PC起動時に実行するもよし、手動にするもよし。

## dnsmasq のインストール

Mac（10.8.x）にdnsmasqをインストールする備忘録。brew使って入れます。

```bash
brew update
brew install dnsmasq
cp -a /usr/local/Cellar/dnsmasq/2.66/dnsmasq.conf.example /usr/local/etc/dnsmasq.conf
vi /usr/local/etc/dnsmasq.conf
```

### /usr/local/etc/dnsmasq.conf の変更点

```bash
resolv-file=/etc/resolv.dnsmasq.conf
listen-address=127.0.0.1
conf-dir=/usr/local/etc/dnsmasq.d
```

レコードファイル用のディレクトリを用意し、Macで起動するためのplistファイルも設定してあげます。最後に、参照するDNSサーバリストを /etc/resolv.dnsmasq.conf にコピーします。

```bash
mkdir /usr/local/etc/dnsmasq.d
sudo cp /usr/local/Cellar/dnsmasq/2.66/homebrew.mxcl.dnsmasq.plist /Library/LaunchDaemons
sudo sh -c 'cat /etc/resolv.conf > /etc/resolv.dnsmasq.conf
```

これで設定は以上。

### dnsmasq の起動

```bash
sudo launchctl load -w /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
```

### DNSサーバを dnsmasq に変更

```bash
sudo networksetup -listallnetworkservices
sudo networksetup -setdnsservers Wi-Fi 127.0.0.1
```

sudo networksetup -listallnetworkservices で自身が使っているネットワーク名を確認して下さい。今回はWi-Fiで設定。

### 動作確認

```bash
nslookup yahoo.co.jp
```

### dnsmasq のレコード書式

```bash
address=/ホスト名/IPアドレス
```

/usr/local/etc/dnsmasq.d/ 以下にテキストファイルを置けばOK。あとはdnsmasqを再起動すると反映されます。

### dnsmasqの再起動

```bash
sudo launchctl unload -w /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
sudo launchctl load -w /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
sudo dscacheutil -flushcache
```