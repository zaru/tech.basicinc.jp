---
layout: post
category : AWS
title: ELBを使ってアクセス元のIPを取る方法
tagline: ELBに限らないよ
author : kobayashi
tags : [AWS, Apache]
---
{% include JB/setup %}
AWSを使うことで今までやりにくかったインフラの設定が簡単に試せるようになりました。
そんなAWSのサービスの中にELBという負荷分散装置、いわゆるロードバランサーがあります。

1回だけ実機のロードバランサーを見たことがあるのですが、そいつはちょっといい車が買えるぐらい値段のやつでした。
機能的には結構かなり高機能なものだったので、おそらくロードバランサーの部類でもかなり高いと思うのですが
安価なものでも、そうそう手が出せるものではありません。それに設定も面倒。（多分）

それがELBを使うことでGUIでポチポチするだけで、簡単に設定することができちゃいます。
しかもアクセス数に応じて勝手にスケールするというおまけ付き。
なんてすごいんでしょうAWS。

転送料で料金が決まるというのも、初回のコストをおさえられるので大変たすかります。



## ELBを使うとアクセス元IPアドレスがわからない？
ELBに限らないのですが、ロードバランサーを使った場合apacheに出力されるアクセス元のIPアドレスは、あくまでロードバランサーのIPアドレスになってしまいます。

このままだと何かと不便になりそうなので、ログの出力設定をいじりましょー。

### やり方

http.confに設定をします。

ELBを経由するとhttpのヘッダにアクセス元のIPアドレスしめす、

「X-Forwarded-For」

が付与されます。

これをアクセスログに出力するようにします。
ついでにhttpsでアクセスしたかどうかわかる「X-Forwarded-Proto」も追加しておきます。


%{ヘッダ名}i

でログにhttpヘッダの内容を出力することができるので、、

	LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D %{X-Forwarded-For}i %{X-Forwarded-Proto}i" elb-customlog

こんな感じ。
後はVirtualHostのCustomLog設定を

	CustomLog "|/usr/sbin/rotatelogs /var/log/httpd/mysite/access_log.%Y%m%d 86400" elb-customlog

としておけばOK


実際にアクセスしてみると・・・

	10.146.39.54 - - [28/Apr/2013:21:41:47 +0900] "GET / HTTP/1.1" 200 98430 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3) AppleWebKit/537.31 (KHTML, like Gecko) Chrome/26.0.1410.65 Safari/537.31" 64251 124.255.0.66 http

こんな感じで、項目の後ろから2つ目にIPアドレスが取得できていることが確認できましたー！




## 監視用のアクセスをログに出力しないようにする
ELBはサーバの生死確認のために定期的にアクセスをして監視しています。
この監視用のアクセスがapacheのアクセスログに出力されてしまうので、ついでに監視のためのアクセスはログに出力しないようにします。

こんな感じのログです。

	10.146.39.54 - - [28/Apr/2013:18:29:50 +0900] "GET /index.html HTTP/1.1" 200 678 "-" "ELB-HealthChecker/1.0" 2584 - -

例によってhttp.confに設定します。

	SetEnvIf User-Agent "ELB-HealthChecker/1\.0" nolog

User-Agentを見て「ELB-HealthChecker/1\.0」の場合にはnologという環境変数を設定します。

そしてCustomLogの方で

	CustomLog "|/usr/sbin/rotatelogs /var/log/httpd/mysite/access_log.%Y%m%d 86400" elb-combined env=!nolog

とするとnologが設定されているとログが出力されなくなります。
