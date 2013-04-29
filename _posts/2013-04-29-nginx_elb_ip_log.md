---
layout: post
category : nginx
title: nginx + ELBを使っていてアクセスログにELBのIPが記録されるのを防ぐ
tagline: 
author : sakuraba
tags : [nginx, ELB, AWS]
---
{% include JB/setup %}

AWSのELBを使って複数のnginxを動かしている場合、デフォルトのままだとELBのIPアドレスがアクセスログに記録されます。当たり前ですね。でもこれだと不便なので、nginxの設定を変更します。

## /etc/nginx/nginx.confの編集

	http{
		set_real_ip_from   10.0.0.0/8;
		real_ip_header     X-Forwarded-For;
	}

と、こんな感じで指定してあげるとOK。