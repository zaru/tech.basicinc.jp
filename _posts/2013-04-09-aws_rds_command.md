---
layout: post
category : AWS
title: AWS RDS MySQLでクエリログを出力する方法
tagline: “普通のMySQLよりは面倒…”
author : sakuraba
tags : [AWS, RDS, MySQL]
---
{% include JB/setup %}

ここ最近、AmazonWebServiceばかり使ってる桜庭@zaruです。フレームワークやORMを使ってデータベースを利用することが殆どで、生のSQLを書くことなんて少なくなってきた昨今ですが、開発中にはどんなクエリが生成されているのか気になったりするよね。パフォーマンスとか。パフォーマンスとか。

というわけで、RDSのMySQLでもクエリログを出力できるようにする方法を紹介。

## Amazon Management Consoleからの操作

まずは手軽に変更する方法。

[Amazon Management ConsoleのRDSページ](https://console.aws.amazon.com/rds/home)から、DB Parameter Groups というメニューを選択。

なんらかのRDSインスタンスを立ちあげていれば、default.mysql5みたいなグループが作られているので、そちらを選択。下のフレームに、パラメータのリストが表示されます。

general_log という名前で検索をして、値に「1」を入力。その後、インスタンスを再起動すれば準備はOK。

## クエリログを確認する

対象のDBインスタンスに入って、下記クエリを実行。

```sql
select event_time, argument from mysql.general_log order by event_time;
```

今までクエリログを tail -f でリアルタイム参照していたのとは違うのが微妙だけど…。

## コマンドラインでRDSを操作する

いちいちAWSの管理画面に入らなくても、コマンドラインで操作することもできます。便利ですね。

### Amazon RDS Command Line Toolkitのダウンロード

[Amazon RDS Command Line Toolkit](http://aws.amazon.com/developertools/Amazon-RDS/2928)からダウンロード。ファイルを展開。

配置場所
/Users/hoge/Documents/AWS/RDSCli
(*) 場所は適宜

credential-file-path.template を credential-file-path.txt とコピー。ファイルを開いて、AWSAccessKeyIdとAWSSecretKeyを入力して保存。キーはAWSのSecurityCredentialsページで確認できます。

### 環境変数の登録

bashの場合は、~/.bash_profileだったかな？僕はzsh派なので、~/.zshrcを編集。

```bash
# Java Setting
export JAVA_HOME=/System/Library/Frameworks/JavaVM.framework/Versions/1.6/Home

# EC2 Setting TOKYO
export EC2_REGION=ap-northeast-1

# RDS Setting
export AWS_RDS_HOME=/Users/hoge/Documents/AWS/RDSCli
export AWS_CREDENTIAL_FILE=$AWS_RDS_HOME/credential-file-path.txt
export PATH=$PATH:$AWS_RDS_HOME/bin
```

東京の場合は、ap-northeast-1を指定してください。編集後、source ~/.zshrc などとして変更を反映します。（もしくはログアウト）

### コマンド操作

	$ rds-describe-db-parameter-groups
	DBPARAMETERGROUP  default.mysql5.5  mysql5.5  Default parameter group for mysql5.5
	DBPARAMETERGROUP  stg               mysql5.5  stg parameter

こんな感じで、登録されているパラメータグループが表示されます。パラメータの変更は

	$ rds-modify-db-parameter-group パラメータグループ名 -p "name=パラメータ名, value=値, method=immediate”

です。最後の、methodには immediate（即時） の他に pending-reboot（再起動後） を指定出来ます。
