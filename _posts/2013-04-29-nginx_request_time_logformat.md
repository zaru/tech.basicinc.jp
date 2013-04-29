---
layout: post
category : nginx
title: nginxでリクエストの処理時間をログに記録する方法
tagline: 
author : sakuraba
tags : [nginx]
---
{% include JB/setup %}

nginxを使用していてリクエストにかかる処理時間をログに記録したい時ってあると思います。たぶんあります。きっとあります。そんな時は、さくっとログフォーマットを指定してあげます。

## ログフォーマットの変更

	http{
		...（略）...
	    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" $request_time';

		access_log /hoge/access.log main;
		...（略）...
	}

こんな感じで、logformatに「$request_time」を追記してあげると、処理にかかったミリ秒が記録されていきます。簡単。

ちなみに、error_logにはもちろん指定できないので気をつけて下しあ。