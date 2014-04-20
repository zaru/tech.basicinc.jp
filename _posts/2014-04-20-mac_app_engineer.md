---
layout: post
category : Mac
title: 新卒エンジニア向け：Macにインストールすべきアプリ達
tagline: 
author : sakuraba
tags : [Mac]
---
{% include JB/setup %}

2014年新卒エンジニア向けに、最初から知っておくと便利になるであろうアプリなどを紹介する機会があったので、ついでにブログ記事としてまとめておく。Boxenとかもあるけど、正直手軽さにかける印象なので、普通にHomeBrewと手作業。

下記で説明しているHomeBrew Caskを使えば、ほぼ一発で必要なアプリケーションが手に入るのでオススメ。

正直、iOS/Androidエンジニア向けのものは、あんまりない…。すまそん。

## 前提

- Mac OSX 10.9〜（多少、古くても出来ると思うけど）
- Webエンジニア（PHPメイン）／iOS,Androidエンジニア対象

## なにがなんでも最初に入れるべきアプリ

- [Xcode](#xcode)
- [HomeBrew](#homebrew)

## HomeBrewを通じてインストールすべきもの

- Brew Cask
- [zsh](#zsh)
- [vim](#vim)
- [jq](#jq)  | **JSONパーサー**
- [curl](#curl)
- wget
- [dnsmasq](#dnsmasq)  | **DNS**
- [composer](#composer)  | **PHPライブラリ管理ツール**
- [iTerm2](#iterm2)  | **ターミナル**
- [alfred](#alfred)  | **キーボードランチャ**
- [Skitch](#skitch)  | **スクリーンキャプチャ**
- Evernote
- Dropbox
- Skype
- [Dash](#dash)  | **ドキュメント検索・スニペット**
- [SourceTree](#sourcetree)  | **Gitクライアント**
- [MySQL Wordkbench](#mysqlworkbench)  | **MySQL ER図など**
- [Sequel Pro](#sequel)  | **MySQLクライアント**
- Firefox
- GoogleChrome
- YoruFukurou  | **定番Twitterクライアント**
- kobito  | **Qiita対応Markdownエディタ**
- VitualBox
- [Vagrant](#vagrant)
- SublimeText
- [PHPStorm](#phpstorm)
- Eclipse
- Unity3D
- CyberDuck  | **SFTPクライアント**
- The Unarchiver  | **圧縮解凍**
- [ImageOptim](#imageoptim)  | **画像軽量化**
- [SpeedLimit](#speedlimit)  | **ネット接続速度制限**
- [COLORS](#colors)  | **カラーピッカー**
- SiteSucker  | **指定ホストのファイル全部ダウンロード**
- Google日本語入力

## 手動でも入れておきたいもの

- Android SDK
- Android NDK
- cocos2d-x
- CocoaPods  | **iOSライブラリ管理ツール**
- [Shupapan](#shupapan)  | **リネーム**

## なくてもいいけど個人的に好きなアプリ

- [Pocket](#pocket)  | **あとで読む系**
- Reeder | **定番RSSリーダー**
- Byword   | **Markdownエディタ**
- 1Password   | **パスワード共有アプリ**

## 登録すべきサービス

まぁ、そんなにないんだけど登録しておくと良い。

- GitHub
- はてな
- Qiita
- TestFlight
- Gravatar
- ChatWork
- Skype

## 以下、各アプリの紹介

### <a name="xcode">Xcode</a>

![xcode.png](/assets/img/2014-04-20-xcode.png)

誰がなんと言おうと絶対に必要な開発環境Xcode。AppStoreからサクッとインストールしましょう。ファイルサイズはかなりのものなので忍耐強く待つ。

#### Command Line Toolsをインストール

Xcodeをインストールする理由は、iOSアプリ開発というのもあるけど、Command Line Toolsをインストールしたいというのもある。なんか新規だとOSX 10.9以降、Xcodeのプレファレンスからだと入れられないっぽい？

```
$ xcode-select --install
```

このコマンドで、Command Line Toolsのインストールが可能。このCommand Line Toolsなにがあるのかというと、コンパイラやGitなどのコマンド群が入っている。

### <a name="homebrew">HomeBrew</a>

![homebrew.png](/assets/img/2014-04-20-homebrew.png)

Mac OSXのパッケージ管理システム。ホームブルーと発音するらしい。ブリューだと思ってた…。大抵のMacアプリはこのHomeBrewを通じてインストール可能。

#### インストール方法

```
ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
```

#### HomeBrew本体のアップデート

```
$ homebrew update
```

#### パッケージを探す

```
$ homebrew search パッケージ名
```

#### パッケージをインストール

```
$ homebrew install パッケージ名
```

#### インストールされたパッケージ一覧

```
$ homebrew list
```

#### パッケージをアンインストール

```
$ homebrew remove パッケージ名
```

### Brewfileで一発インストール

HomeBrewにはBrewfileというものを用意することで一発でインストールしてくれる。適当なディレクトリに Brewfile という名前のファイルを作り、自分がHomeBrew経由でインストールしたいパッケージ名などをつらつら書く。

ちなみに、Homebrew-Caskをインストールしておくことで、HomeBrew本体では管理されていないパッケージ群なども入れられる。

```
# Add Repository
tap homebrew/versions || true
tap phinze/homebrew-cask || true
tap homebrew/binary || true

# Brew Update
update || true

# Brew Cask
install brew-cask || true

# Packages
install --disable-etcdir zsh || true
install vim || true
install jq || true
install curl || true
install wget || true
install dnsmasq || true
install composer || true

# Cask Packages
cask install iterm2 || true
cask install alfred || true
cask install dash || true
cask install skitch || true
cask install evernote || true
cask install dropbox || true
cask install skype || true
cask install sourcetree || true
cask install mysqlworkbench || true
cask install sequel-pro || true
cask install firefox || true
cask install google-chrome || true
cask install yorufukurou || true
cask install kobito || true
cask install vitualbox || true
cask install vagrant || true
cask install sublime-text || true
cask install phpstorm || true
cask install eclipse-ide || true
cask install unity3d || true
cask install cyberduck || true
cask install the-unarchiver || true
cask install imageoptim || true
cask install speedlimit || true
cask install colors || true
cask install sitesucker || true
cask install google-japanese-ime || true
```

bundleコマンドでゴニョゴニョインストールしてくれる。それなりに時間がかかるので辛抱強く待つ。エンジニアは待つことが仕事。

```
$ brew bundle
```

### <a name="zsh">zsh</a>

最強コマンドシェルと呼び声高いzshちゃん。Mac標準でも入っているけど、バージョンが古いのでHomeBrewで最新版を入れてあげる。

brew bundleで自動に入れている場合は下記は必要ない。

```
$ brew install --disable-etcdir zsh
```

HomeBrew経由のzshのパスを追加する。

```
$ sudo vi /etc/shells
```

```
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
/usr/local/bin/zsh # ←追加
```

デフォルトのログインシェルをzshに変更する

```
$ chpass -s /usr/local/bin/zsh
```

#### .zshrc

zshをもっと便利に使うために設定をカスタマイズしてみる。いろんな人が公開しているzshをパクってみたもの。

<script src="https://gist.github.com/zaru/7462719.js"></script>

一部、MAMPで入れたMySQLへのパスが入っているので必要ない人は、さくっと削除で。

### <a name="vim">vim</a>

Emacsじゃなくて、どっちかっていうとVim派なので、とりあえず.vimrc。これも誰かが公開していたのをパクった。Vimプラグインを入れてガツガツ使うほどじゃないので必要最低限レベル。

<script src="https://gist.github.com/zaru/7462752.js"></script>

### <a name="jq">jq</a>

jqはJSONを読みやすくしてくれる便利コマンド。詳しくは[JSONを超絶に読みやすくする jq コマンド](http://tech.basicinc.jp/JavaScript/2013/05/17/json_jq/)を参照。

```
$ curl 'http://example.com/api/hoge.json' | jq .
```

こんなかんじで、JSONをパースしてくれる。

### <a name="curl">cURL</a>

cURL（カール）は、たぶん色んな所で使う。ちょっとした便利コマンドを紹介しておく。

#### レスポンス計測

```
$ curl -kL 'https://example.com/' -o /dev/null -w "%{time_total}\n" 2> /dev/null 
```

curlを使って特定URLのレスポンスを計測することが出来る。お手軽パフォーマンスチェックツール的に。

#### リクエストヘッダ・レスポンスヘッダを見たい

```
$ curl -v'http://example.com/' 1> /dev/null
```

#### Cookieの送受信

```
# cookie.txtにCookieを保存
$ curl -c cookie.txt 'http://exmaple.com/'
# cookie.txtを送信
$ curl -b cookie.txt 'http://exmaple.com/'
```

#### リファラの指定

```
$ curl -e 'http://example.com/ref' 'http://exmaple.com/'
```

#### ユーザエージェントの変更

```
$ curl -A UserAgent 'http://exmaple.com/'
```

### <a name="dnsmasq"> dnsmasq </a>

dnsmasqは要はローカルにたてられるお手軽DNSサーバ。社内の開発用サーバとかに手軽にアクセスするのに使ったり、AWS EC2へのショートカットに使ったり色々。便利。

[EC2インスタンスのIPアドレスを自動でローカルDNSに登録する](http://tech.basicinc.jp/ec2/2013/09/01/ec2_private_dns/)を参照。

### <a name="composer"> composer </a>

PHPのライブラリ管理ツールcomposer。今どきこれがないとPHPなんてやってられん。というわけで、絶対必須。というか開発環境を配布する際にcomposerファイルも一緒に配布するので、ないと死ぬ。

[PHPのパッケージ管理Composerを使う](http://tech.basicinc.jp/php/2013/08/18/php_composer/)を参照。

### <a name="iterm2"> iTerm2 </a>

もうターミナルアプリはiTerm2以外ないでしょということで。

![iterm2.png](/assets/img/2014-04-20-iterm2.png)

使い方は、特にないけどショートカットを覚えておけば捗る。

- cmd + d（画面縦分割）
- cmd + shift + d（画面横分割）
- cmd + [ or ]（ペイン移動）
- cmd + shift + h（クリップボード履歴）

### <a name="alfred"> alfred </a>

定番ランチャ。以前はQuickSilverを使っていたけど、alfredの方が流行っているふうなので、乗り換え。有料版だとワークフローなどを作れるらしいけど、普通にキーボードランチャとして使っているレベル…。

![alfred.png](/assets/img/2014-04-20-alfred.png)

### <a name="skitch"> Skitch </a>

Evernoteに買収されてから著しく機能がダウングレードしたスクリーンキャプチャアプリSkitchさん。でもまぁ、とりあえず便利なので入れておく。範囲塗りつぶしツールが削除されたのがマジで痛い。

最近はMonosnapというのが流行っているっぽい。ので、こっちも良いかもね。

- [Monosnap](http://monosnap.com/welcome)

![skitch.png](/assets/img/2014-04-20-skitch.png)

### <a name="dash"> Dash </a>

ドキュメント・リファレンスの超高速検索アプリ。大抵の言語・フレームワークのドキュメントは対応している。検索だけじゃなくて、スニペットを登録して、さっとコード入力も出来るスグレモノ。ちなみに無料でも使い続けられるけど、時々有料にしませんか？とか出たりするので、頻繁に使うなら買った方がいい。

![dash.jpg](/assets/img/2014-04-20-dash.jpg)


### <a name="sourcetree"> SourceTree </a>

Git/MercurialのGUIツール。無料で使える中では一番使いやすい。

push / pull / mergeなどはコマンドで行うが、addやリセット、履歴を追っかけるときにはSourceTreeを使うほうが圧倒的に早い。ただし、あまりにも巨大なリポジトリの場合、落ちることがある…。

![sourcetree.png](/assets/img/2014-04-20-sourcetree.png)


### <a name="mysqlworkbench"> MySQL Workbench </a>

地味にアップデートし続けているMySQL Workbench。安定性には欠けるものの、手軽にER図やスキーマ設計などができる。実際にMySQLサーバに接続して、スキーマの同期を行ったり、稼働しているDBを元にER図を作ってくれたりと、便利。

Excelとかなんからのドキュメントにスキーマ一覧を出力してくれる機能があると便利なんだけど、それは…ない。

![mysqlworkbench.png](/assets/img/2014-04-20-mysqlworkbench.png)


### <a name="sequel"> Sequel Pro </a>

MySQLサーバへ接続してるSQLクライアント。個人的には使用していないが、社内エンジニアがおすすめするので入れておいた。データベースの値を見ながら開発をするのに便利らしい。

あと、接続先を保存できるので、いくつもプロジェクトを抱えている場合は、さっと接続できて良いのかもしれない。

![sequel.jpg](/assets/img/2014-04-20-sequel.jpg)

### <a name="vagrant"> Vagrant + VirtualBox </a>

ちょっと前まではMAMPでローカル開発環境を用意していたけど、今はVagrant + VirtualBoxでサクッと作るのが良い。いろんな使い方があるけど、ベーシック社内ではBerkShelf + Vagrant-Omnibusの組みあせが定番になりつつある。ただ、バージョン依存がけっこう激しいので、Macの状態によっては導入するのに苦戦するという難問が…。

ここらへんもう少しベストプラクティス的なのあるといいんだけど。

[Vagrant+Chef+BerkShelfでLAMPの環境を作る](http://tech.basicinc.jp/Vagrant/2014/02/02/vagrant_chef/)を参照。

### <a name="phpstorm"> PHPStorm </a>

PHP使っているなら、もうこれでしょっていうくらい無敵IDE PHPStorm。弱点は、個人で$99、企業だと$199の年間ライセンスがかかるということ。（ちなみに1年間過ぎてもアプリ自体は継続して使用可能。アップデートの提供がうけられなくなる）

PHPStorm使っていて良いなと思う点。

- コードジャンプ（特にフレームワーク使っていると便利）
- コード補完強力
- プロジェクト内ファイル検索が早い
- コードフォーマットを柔軟に設定可能（コード規約統一しやすい）
- Git/Mercurialの操作も可能（diffとか便利）
- Vagrantとも連携

SublimeTextなども軽量高機能エディタとして良いけどね。

![phpstorm.png](/assets/img/2014-04-20-phpstorm.png)


### <a name="imageoptim"> ImageOptim </a>

とにかく画像の余計な情報をすっ飛ばして、軽量化したいという時にはImageOptim。Webページの表示高速化や、ゲームアプリの容量を削減するときに大活躍。無料なのに凄い。

対象ファイルを適当に放り込めば完了。それなりにCPUパワーを使うので、いっぱい放り込んだらファンがブンブン回って、軽量化している感が出て良い。

![imageoptim.png](/assets/img/2014-04-20-imageoptim.png)


### <a name="speedlimit"> SpeedLimit </a>

Macのネットスピードを制限できる素晴らしいアプリ。iOS/Androidアプリ開発時に、3G回線のシミュレーションをしたりできる。実機でも行いたい場合は、Macをインターネット接続共有すればOK。

特定ホスト・ポートごとに設定可能なので、開発サーバだけに限って指定可能。

![speedlimit.png](/assets/img/2014-04-20-speedlimit.png)


### <a name="colors"> COLORS </a>

Macのカラーピッカーアプリの中で一番使いやすいと思う。

![colors.png](/assets/img/2014-04-20-colors.png)

### <a name="shupapan"> Shupapan </a>

Macで唯一と言ってもいいGUIのリネームアプリ。時々、大量ファイルのリネームをする必要があり、GUIで確認しながらやりたいビビリなので重宝している。正規表現なども使えるので、ひと通りのリネームに対応できる。

- [Shupapanダウンロード](http://sunsky3s.s41.xrea.com/shupapan/download/)

![shupapan.png](/assets/img/2014-04-20-shupapan.png)

### <a name="pocket"> Pocket </a>

あとで読むを支援してくれるサービス。各種あるあとで読む系で今一番良いサービス。MacでもPocketアプリが出ているが、どちらかというとiPhoneとかで使う。スマートニュースとかグノシーとかもPocket連携しているので、いろんなニュース元から気になる記事をどんどん追加していって、暇な時に読み返すと良い。

![pocket.png](/assets/img/2014-04-20-pocket.png)

## まとめ

これで必要最低限のエンジニアスタートが切れると思う。後は、プロジェクトや開発言語などに応じて必要なパッケージを入れて、各アプリの使い方を深堀りするとOK。