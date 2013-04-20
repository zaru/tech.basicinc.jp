---
layout: post
category : PHP-FPM
title: nginx + PHP-FPMで502 bad gateway...ヘッダーが大きすぎるとエラーが出た時の対処法
tagline: JSONを吐き出すAPIでエラーに…
author : sakuraba
tags : [PHP-FPM, nginx]
---
{% include JB/setup %}

nginx + PHP-FPMで運用しているWebAPIサーバがあり、レスポンスタイプとしてJSON / XML を採用しています。どちらのタイプも自由に選べるようにしています。

ある日、JSONでだけなぜか 502 bad gateway のエラーが…！XMLでは正常にレスポンスが返ってくる。

とりあえずエラーログを見たら

	2013/04/01 0:00:00 [error] 10636#0: *388556 upstream sent too big header while reading response header from upstream, client: 1.78.15.168, server: www.example.com, request: "GET /api/hoge.json HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "www.exmaple.com"

というのが。どうやら、ヘッダーが大きすぎて駄目だよってことらしい。

## nginxの設定変更

調べてみたらnginxのバッファを調整すればいけそう。というわけで nginx.conf に下記のようなオプションを加えます。

	fastcgi_buffers 8 16k;
	fastcgi_buffer_size 32k;

あとはnginxを再起動すればOK。無事JSONで正常なレスポンスを得ることに成功。XMLと違って、JSONだとヘッダーとしてコンテンツが認識されちゃうんだろうか…。