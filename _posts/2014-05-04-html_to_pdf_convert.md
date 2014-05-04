---
layout: post
category : Linux
title: HTMLをPDFに上手いこと変換してくれるwkhtmltopdfを日本語で使う
tagline: 
author : sakuraba
tags : [Linux]
---
{% include JB/setup %}

**結論：サーバサイドでHTMLをPDFに変換したい。[wkhtmltopdf](http://wkhtmltopdf.org/)を使おう。**

サーバサイドでのHTMLからPDFへの変換。今どき流行らない気もしたんだけど、業務上どうしても必要だったので調べてみた。サーバサイドでPHPを使ってHTMLを出力していたので、そのままPHPからPDFを生成するライブラリを調べていたんだけど、どうもレイアウトが弱い…。

## PHPからPDFを出力するライブラリ

- [FPDF](http://www.fpdf.org/)
- [mPDF](http://www.mpdf1.com/mpdf/)
- [TCPDF](http://www.tcpdf.org/)

そもそも、PHPを通す選択肢が悪手な感じ。ひと通り試したけど、微妙なレイアウトずれや、変換パフォーマンスが悪い。

## その他に…

- [Docverter](http://www.docverter.com/)
- [Flying Saucer](https://code.google.com/p/flying-saucer/)
- [Sphinx](http://sphinx-users.jp/cookbook/pdf/)

他になんかあるかなーと思って調べてたら、ここらへんが。けっこう良さそう。特にDocverter（内部的にはFlyingSaucerを使ってる）とかは対応フォーマットもかなりあって、良い感じ。

## もっとシンプルに

![wkhtmltopdf](/assets/img/2014-05-04-html_to_pdf_convert.jpg)

単純にコマンドベースで変換できるようなのないかな？って思って調べてたらありました。

- [wkhtmltopdf](http://wkhtmltopdf.org/)

まさに。

使い方は簡単で、ビルド済みバイナリ、もしくは自前ビルドしてコマンド実行するだけ。一応Linux / Windowsに対応。Macに関してはまだだけど、そのうち公開されるっぽい雰囲気。

### コマンド例

```
$ wkhtmltopdf http://google.com google.pdf
```

素晴らしい。レイアウトも大抵のHTML+CSSに対応しているっぽい。試しにYahoo!のトップページサンプル。

![wkhtmltopdf](/assets/img/2014-05-04-html_to_pdf_convert2.jpg)

多少、背景画像などがずれたりしてるけど、厳密な変換が求められないなら、この程度であれば大丈夫でしょう。

## wkhtmltopdfを日本語対応に

素のLinuxとかだと、wkhtmltopdfは日本語を変換できず。というよりは、単純に日本語フォントが入っていないだけ。なので、フォントをインストールしてあげればOK。

ここはひとつ、IPAフォントを使う。

### IPAフォントインストール

- [IPAexフォントダウンロード](http://ipafont.ipa.go.jp/ipaexfont/download.html)

```
# ユーザホームディレクトリにフォントを配置
$ mkdir ~/.fonts
$ cp IPAfont00302.zip ~/.fonts
$ cd ~/.fonts 
$ unzip IPAfont00302.zip 

# フォントキャッシュクリア
$ fc-cache -fv

# インストールされているフォントの確認
$ fc-list
```

これだけで日本語が入ったページも正常にPDFにしてくれる。
