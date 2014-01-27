---
layout: post
category : AWS
title: AWS ELBに設定したSSL証明書を削除する
tagline: 
author : sakuraba
tags : [AWS, ELB]
---
{% include JB/setup %}

AWSのELBにSSL証明書を設定するのは普通にブラウザからマネージメントコンソール画面でできますが、なぜか登録済みの証明書は削除できない…。ので、削除する方法メモ。

## IAM Command Line Toolkit インストール

IAM Command Line Toolkitというツールを使って行います。

### ダウンロード

[IAM Command Line Toolkit : Developer Tools : Amazon Web Services](http://aws.amazon.com/developertools/AWS-Identity-and-Access-Management/4143)

### 配置

```bash
$ unzip IAMCli.zip
$ mv IAMCli-1.5.0 ~/Applications/
$ cd ~/Applications/IAMCli-1.5.0
$ pwd
/Users/hoge/Applications/IAMCli-1.5.0
```

場所はどこでもいいんですが、とりあえずユーザディレクトリ以下のApplicationsに置いておきます。

### X.509証明書のダウンロード

AWSのマネージメントコンソール画面の中にあるSecurity CredentialsからX.509証明書を発行＆ダウンロードします。

[IAM Management Console](https://console.aws.amazon.com/iam/home)

ダウンロードした証明書＆秘密鍵を、下記で設定する環境変数の EC2_CERT / EC2_PRIVATE_KEY の場所に配置します。

### 環境変数の設定

使用するために、環境変数をセットします（例はzsh）。

```bash
$ vi ~/.zshrc
```

```bash
export AWS_IAM_HOME=/Users/hoge/Applications/IAMCli-1.5.0
export PATH=$PATH:/Users/hoge/Applications/IAMCli-1.5.0/bin
export EC2_CERT=/Users/hoge/Applications/IAMCli-1.5.0/cert/cert/cert_hoge.pem
export EC2_PRIVATE_KEY=/Users/hoge/Applications/IAMCli-1.5.0/cert/cert/pk_hoge.pem
```

```bash
$ source ~/.zshrc
```

### 証明書の一覧

```bash
$ iam-servercertlistbypath
arn:aws:iam::123866398456:server-certificate/SSL2012
arn:aws:iam::123866398456:server-certificate/SSL2013
```

### 証明書の削除

```bash
$ iam-servercertdel -s SSL2012
```

ちなみに、デフォルトだと証明書は10個までしか登録できないらしい…（申請して拡張可能）。

