---
layout: post
category : Linux
title: Webサーバ起動時に、自動的にGitで最新アプリケーションをデプロイする方法
tagline: /etc/rc.local or Cron
author : sakuraba
tags : [Linux, Git]
---
{% include JB/setup %}

AWSのEC2でWebサーバを分散化していて、トラフィックが急激に上昇した場合、スケールアウトするんだけど、AMIイメージファイルのアプリケーションファイルが若干古いことって、まぁまぁよくあると思います。オートスケールだと、ちょっとマズイ。というわけで、EC2インスタンスをロンチ後、自動でGitから最新ファイルを持ってくるようにする方法。

っても、AWSにもEC2にもGitにもまったく関係ない、古くからあるLinuxの手法。

## /etc/rc.local を使う

/etc/rc.local は、他のサービスなどが起動し終わった後に独自にシェルスクリプトが実行できる。

```bash
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local
ここに独自のスクリプトを記述する
```

ただし、スーパーユーザ権限で動くので、一般ユーザとしてGitコマンドを実行する必要あり。

```bash
sudo -u webmaster git pull origin master
```

こんな感じ。

## Cron を使う

一般ユーザのCronを使って、起動時のみに実行することができる。

```bash
$ crontab -e
@reboot /home/webmaster/git_deploy.sh
```

通常は先頭に実行感覚の時間などを記述するが、@reboot と指定すると起動した時にのみ実行させることができる。もちろんCronで実行したユーザ権限なので、sudoとかしなくてもOK。ちなみに、スクリプトの中身は単純。

```bash
#!/bin/bash
cd /var/www/vhosts/www.example.com/
git pull origin master
```

ちなみに、@reboot以外に指定できるオプション。

|オプション|内容|
|---|---|
|@reboot|起動時に1度だけ実行|
|@yearly|1年に1度だけ実行（ 0 0 1 1 * と同じ）|
|@annually|@yearlyと同じ|
|@monthly|1ヶ月に1度だけ実行（ 0 0 1 * * と同じ）|
|@weekly|1週間に1度だけ実行（ 0 0 * * 0 と同じ）|
|@daily|1日に1度だけ実行（ 0 0 * * * と同じ）|
|@hourly|1時間に1度だけ実行（ 0 * * * * と同じ）|



もっと綺麗にやるならCapistranoとかでやるのが良いんだろうけど。

