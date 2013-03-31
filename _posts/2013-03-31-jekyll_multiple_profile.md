---
layout: post
category : jekyll
title: Jekyllで複数ユーザ対応のブログにする
tagline: ""
author : sakuraba
tags : [jekyll]
---
{% include JB/setup %}

Jekyllでエンジニアブログを作ったのはいいけど、ポストするのは僕だけじゃない…ということは、複数人で更新できるようにしなければ。

ってもコンテンツ自身はGitで管理しているし、更新作業自体は誰でもできる（Gitリポジトリにアクセスできる権限があれば）。なので、今回はJekyllにライター名を指定して、それに合ったプロフィールをサイドバーなどに表示するようにしてみる。

## _config.ymlにライター情報を追加する

デフォルトでは

	author :
		name : HOGE
		email : hoge@example.com

っていう感じでauthorが書いてあるけど、これとは別のように追記する。

	author :
		name : HOGE
		email : hoge@example.com
	authors:
		sakuraba:
			display_name: 桜庭@zaru
		tanaka:
			display_name: 田中

あとは、ポスト記事のメタに
	---
	layout: post
	category : jekyll
	title: Jekyllで複数ユーザ対応のブログにする
	tagline: ""
	author : sakuraba
	tags : [jekyll]
	---
って書いてあげればOK。

## レイアウトファイルの編集

各テーマディレクトリの post.html を開いて

```html
<h4>Author</h4>
<span>{{ "{{ author.display_name" }} }}</span>
```

としてあげればライター名が表示される。