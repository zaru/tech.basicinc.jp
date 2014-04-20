---
layout: post
category : Mac
title: MacOSXの中に仮想MacOSX環境をVirtualBoxで作る
tagline: Mavericks on Mavericks
author : sakuraba
tags : [Mac]
---
{% include JB/setup %}

![Mavericks](/assets/img/2014-04-20-osx2.png)

Macの開発環境を色々と作る実験のためにMaxOSXの中に仮想環境でMacOSXが欲しい…と思っていたら、なんとそれをVirutalBoxでうまいことやってくれるスクリプトを作っている天使みたいな人がいました。

- [iESD OSX on OSX](http://ntk.me/2012/09/07/os-x-on-os-x/)

どうやら以前はShellScriptでやっていたらしいが、かなり煩雑になったのでRubyGemに移行したっぽい。一応、ShellScript版でもできた。

というわけで、MaxOSX 10.9（Mavericks）環境のVirtualBoxにMavericksを入れてみます。

## AppStoreからMavericksをダウンロード

すでにMavericksな人の場合、再度AppStoreからMavericksをダウンロードしておきます。無事、ダウンロードが終わると、下記パスにAPPファイルが配置されているはず。

```
% ls /Applications/Install\ OS\ X\ Mavericks.app
```

## Ruby版iESD

まずは、gemでiESDをさくっとインストール。

```
$ gem install iesd
```

適当なディレクトリで下記iesdコマンドを実行。Output.dmgファイルができあがる。

```
$ iesd -i /Applications/Install\ OS\ X\ Mavericks.app -o Output.dmg -t BaseSystem
```

## ShellScript版iESD

GitHubに公開されているのでcloneして持ってきて、同じようにiesdコマンドを実行してOutput.dmbファイルを生成する。

```
$ git clone https://github.com/ntkme/iesd
$ cd iesd
$ bin/iesd -t BaseSystem -i /Applications/Install\ OS\ X\ Mavericks.app/Contents/SharedSupport/InstallESD.dmg -o Output.dmg
```

## VirtualBoxで新規OS作成

- 名前：適当
- タイプ：Mac OS X
- バージョン：Mac OS X Mavericks(64bit)

### ハードウェア

- メモリ：2048MB〜
- CPU：2コア〜
- ビデオ：16MB〜（適当）
- ストレージ：60GB（適当）
- ネットワーク：Intel PRO/1000 MT Server(NAT)
- システムチップセット：PIX3

なぜかデフォルトで選ばれているチップセットICH9では起動しないので、PIX3を選択しておく。

### Mavericksインストーラーを選択

![Mavericks](/assets/img/2014-04-20-osx1.png)

CD/DVDドライブイメージで上記で作成したOutput.dmgファイルを選択しマウントするようにしておく。後は起動する。

無事、インストーラーが起動したら、OS Xインストーラー内の「ユーティリティ」→「ディスクユーティリティ」を起動し、ディスクを選択→「消去」タブ→「Mac OS 拡張（ジャーナリング）」を選択し、フォーマットする。

その後は、普通にインストール作業を進めて完了。

わりと新しめのiMac（3.2Ghz Core i5 / 16GB mem）でやったけど、軽いって言う感じじゃない…あくまで動作検証レベルで使う感じ。GUI描画がちょい荒い感じ。

### 参照URL

- [Mac上のVirtualBoxにMavericksをインストールする](http://qiita.com/hnakamur/items/fca6379213a3033cb29d)
- [OS X on OS X](http://ntk.me/2012/09/07/os-x-on-os-x/)
- [GitHub iesd](https://github.com/ntkme/iesd)