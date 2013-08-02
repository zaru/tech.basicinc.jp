---
layout: post
category : Linux
title: git pushでバーチャルホスト設定やデプロイを自動で行う
tagline: ブランチ切る手軽さで、開発環境を作る
author : sakuraba
tags : [Linux, Git, nginx]
---
{% include JB/setup %}

ブランチを手軽に切れるのが魅力なGitちゃん。

ただ、Webアプリケーションとして、気軽に切った先進的で先鋭的で危ない感じのその新ブランチを他の人に動作を確認してもらうのは、ちょっと面倒。

開発者ならpullってdiffって見ろよって感じなんだけど、営業さんだったりディレクターさんだったりだとブラウザで確認できなくちゃね。というわけで、開発サーバにバーチャルホスト切って、サブドメインとかで設定するわけなんだけど、それが手間。

ブランチは気軽に切れるのに、環境は気軽に作れない。

## post-receive を使おう

Gitにはpost-receiveという強力な仕組みがあるので、これを使います。

```bash
$git push origin new-branch
```
って新ブランチをリモートリポジトリにpushしたら、http://new-branch.example.com/ ってサブドメインで閲覧できるようになれば良い。

開発サーバのHTTPDは、nginxを利用し、ドキュメントルートは /var/www/vhosts/ 以下にサブドメイン名で設置するようにする。

まず、nginxのバーチャルホスト用設定ファイルのテンプレートを用意する。サブドメイン名をブランチ名として使う。

### /etc/nginx/conf.d/virtualhost.conf.template

```bash
server {
    listen   80;
    server_name __BRANCH_NAME__.example.com;
    gzip on;
    gzip_types text/css application/xml application/x-javascript application/json;

    access_log /var/www/vhosts/__BRANCH_NAME__.example.com/logs/access.log;
    error_log /var/www/vhosts/__BRANCH_NAME__.example.com/logs/error.log;

    location / {
        root   /var/www/vhosts/__BRANCH_NAME__.example.com/htdocs/app/webroot/;
        index  index.php index.html index.htm;
        if (-f $request_filename) {
            break;
        }
        if (-d $request_filename) {
            break;
        }
        rewrite ^(.+)$ /index.php?q=$1 last;
        client_max_body_size 10M;
    }


    location ~ .*\.php[345]?$ {
        fastcgi_pass    127.0.0.1:9000;
        fastcgi_index   index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/vhosts/__BRANCH_NAME__.example.com/htdocs/app/webroot$fastcgi_script_name;
        fastcgi_intercept_errors on;
        include         fastcgi_params;
        client_max_body_size 20M;
    }
}
```

次に、nginxの設定ファイル生成 + Gitをcloneして新ブランチにcheckoutするようなスクリプトを作成する。引数としてカンマ区切りのブランチ名が渡ってくる想定。

### autoload.sh

```bash
#!/bin/bash

createVirtualHost() {
    branchname=$1
    dirname=$branchname.exmaple.com
    echo $dirname

    cd /etc/nginx/conf.d/
    cp virtualhost.conf.template $dirname.conf
    sed -i "s/__BRANCH_NAME__/$branchname/g" $dirname.conf

    cd /var/www/vhosts/
    sudo -u webmaster git clone git@git.example.com:hoge $dirname
    cd /var/www/vhosts/$dirname
    sudo -u webmaster git checkout -b $branchname origin/$branchname

    /etc/rc.d/init.d/nginx restart
}

cd /etc/nginx/conf.d/

branchlist=$1
arr=$(echo $branchlist | tr "," "\n")
for x in $arr
do
    if [ ! -e ${x}.exmaple.com.conf ]; then
        echo $x
        createVirtualHost $x
    fi
done
```

あとは、post-receiveでautoload.shを叩けばいいだけなんだけど、Gitのリモートリポジトリサーバが開発サーバにない場合が多いので、SSHで開発サーバに入って叩くようにする。

### create_virtualhost.php（Gitリモートリポジトリサーバ内）

```php
<?php
exec('git branch', $branchLists);

$branchArr = array();
foreach ($branchLists as $val) {
	$val = str_replace(array(' ', '*'), '', $val);
	if ($val != 'master') {
		$branchArr[] = $val;
	}
}

exec('ssh user@exmaple.com autoload.sh ' . implode(',', $branchArr));
```

なぜかbashだとgit branchの結果がうまく取れなかったのでPHPを使ってる…。

### /home/git/repositories/exmaple.com.git/hooks/post-receive

```bash
php /home/git/repositories/exmaple.com.git/hooks/create_virtualhost.php &
```

あとはpost-receiveに書いてあげればpushを受け取った時に自動的に実行される。

DNSのレコード登録だけが手動で設定する必要があるけど、AWSを使っているならRoute53とかでそれすらも自動設定できるようになると思う。もう少しスマートにやれる方法があれば良いな…。

いずれにしても、post-receive強力で魅力的。