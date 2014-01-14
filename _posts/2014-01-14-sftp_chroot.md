---
layout: post
category : Linux
title: SFTPでディレクトリ制限ありなユーザを作成する
tagline: chrootなやつ
author : sakuraba
tags : [Linux]
---
{% include JB/setup %}

SFTPでユーザの行動範囲を制限したい時の設定メモ。

## ユーザの追加

```bash
#useradd sftpuser
#passwd sftpuser
#usermod -d / sftpuser
```

## ディレクトリの設定

```bash
#chmod 755 /home/sftpuser/
#chown root:root /home/sftpuser/

#mkdir /home/sftpuser/data/
#chmod 755 /home/sftpuser/data/
#chown sftpuser:sftpuser /home/sftpuser/data/
```

ユーザのホームディレクトリの権限をrootに変更（chrootのため）。実際にユーザにデータを配置できるようにするためのディレクトリを作成し、そちらはそのユーザ権限にする。

## SSHDの設定変更

### /etc/ssh/sshd_config

```bash
#Subsystem      sftp    /usr/libexec/openssh/sftp-server
Subsystem       sftp    internal-sftp

Match User sftpuser
    ChrootDirectory /home/sftpuser
    ForceCommand internal-sftp
```

Subsystemをsft-serverからinternal-sftpへ変更。それに伴い、対象ユーザの制限ディレクトリをChrootDirectoryで指定する。ここで指定した所がトップディレクトリになる。

## SSHDの再起動

```bash
#service sshd restart 
```

これで完了。

## 動作確認

```bash
$sftp sftpuser@example.com
# 接続できる

$ssh sftpuser@example.com
# 接続できない
```