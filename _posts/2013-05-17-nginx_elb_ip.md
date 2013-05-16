---
layout: post
category : nginx
title: nginx + ELBを使っていてアクセスログにELBのIPが記録されるのを防ぐ
tagline: 
author : sakuraba
tags : [nginx, ELB, AWS]
---
{% include JB/setup %}

システムチームの小林さんが[ELBを使っていても本当のアクセス元を取る方法](http://tech.basicinc.jp/AWS/2013/04/28/aws_elb_customlog/)という記事を書いていましたが、Apache限定で今はnginxも必要でしょってなわけで、補足する形で僕も記事書きます。

## /etc/nginx/nginx.confの編集

	http{
		set_real_ip_from   10.0.0.0/8;
		real_ip_header     X-Forwarded-For;
	}

と、こんな感じで指定してあげるとOK。これだけでいけます。簡単ですね。