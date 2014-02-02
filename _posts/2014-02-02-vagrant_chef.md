---
layout: post
category : Vagrant 
title: Vagrant+Chef+BerkShelfでLAMPの環境を作る
tagline: 
author : kobayashi
tags : [PHP, Apache, PostgreSQL, Vagrant]
---
{% include JB/setup %}

いまはやりのローカルに開発環境をつくる方法です。
一度作ってしまえば環境をコピーするのは簡単そうなのですが、いろいろ覚えないといけないことがあって意外と大変でした。

Vagrant、Chef、BerkShelfがそれぞれ何ものなのかは、、、

他のサイトで確認してみてください(^^;)

タイトルでLAMPといいつつも今回はPostgreSQLをインストールしてます。

### Vagrantのインストール
ここはそんなに問題ないです。

当たり前の話なのですが、、、Boxは本番環境に合わせてちゃんと選びましょう。

例えばCentOSにしたい場合は
[http://www.vagrantbox.es/](http://www.vagrantbox.es/)

ここからCentOSのURLを選びます。

```bash
$ vagrant box add centos http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-x86_64-v20130731.box
$ vagrant init
$ vagrant up
```

これであっという間に仮想環境のできあがり！

仮想環境に入るには
```
vagrant ssh
```
を実行します。

### BerkShelfのインストール
下記コマンドを実行します。

```bash
$ vagrant plugin install vagrant-omnibus
$ vagrant plugin install vagrant-BerkShelf
$ gem i berkshelf
```
またBerksfileというファイルが必要になるので、Vagrantfileと同ディレクトリに作成しましょう。中身は
```
site :opscode
```
とします。

このファイルにインストールするアプリケーションをcookbookとして指定します。

cookbookは自分で作成することもできますが、既に公開されているcookbookがあるのでそちらを使っていく方が楽。

このcookbookを公開しているコミュニティとしてopscodeというのがあって、そこのcookbookを使いますよーという宣言になります。

[http://github.com/opscode-cookbooks/](https://github.com/opscode-cookbooks/) 

今回はphp、apache、postgresqlなので、、、

```bash
site :opscode
cookbook "postgresql"
cookbook "php"
cookbook "apache2"
cookbook "git"
cookbook "vim"
```

こんな感じのファイルになります。

postgresqlではなくてmysqlを使う場合は
```
cookbook "mysql"
```
となります。

gitとvimも必要になるので、いれておきました。apacheはapache2となるので注意。

### Vagrantfileの編集 
ここまで設定できたら次にVagrantfileを触ります。

実際にBerkshelfで指定したアプリケーションのインストールや設定をするには仮想環境上でChefが動いてごにょごにょします。
なのでChefを仮想環境に入れる必要があるので、Vagrantfileに

```ruby
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
   #...
   #...
   #...
   config.omnibus.chef_version = :latest
end
```
を追記します。

これはchefがなければインストールしてね、という設定です。

インストールするものの設定とかは

```ruby
config.vm.provision :chef_solo do |chef|
   #
   #この中に書きます
   #
end
```


最終的にVagrantfileはこちら！！(`・ω・´)

```ruby
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
 config.vm.box = "centos"
 config.vm.network :forwarded_port, guest: 80, host: 8080
 config.vm.network :private_network, ip: "192.168.33.10"

 config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
 end
 config.vm.provision :shell,path:"bootstrap.sh"
 config.vm.provision :chef_solo do |chef|
    chef.log_level = :debug
    chef.run_list = [
        "recipe[postgresql::server]",
        "recipe[postgresql::client]",
        "recipe[php]",
        "recipe[apache2]",
        "recipe[apache2::mod_ssl]",
        "recipe[apache2::mod_rewrite]",
        "recipe[git]",
        "recipe[vim]",
    ]

    chef.json = {
        :postgresql => {
            :version => "9.2",
            :enable_pgdg_yum => 'false',
            :pgdg => {
                :repo_rpm_url => {
                    "9.2" => {
                        :centos => {
                            "6" => {
                                :x86_64 => "http://yum.postgresql.org/9.2/redhat/rhel-6-x86_64/pgdg-centos92-9.2-5.noarch.rpm"
                            }
                        }
                    }
                }
            }
        },
        :php => {
            :version => "5.3",
            :directives => {
                :display_errors => 'On'
            },
            :packages => ["php-mbstring", "php-pgsql"]
        },
        :apache2 => {
            :version => "2.2"
        }
    }
  end
  config.omnibus.chef_version = :latest
  config.berkshelf.enabled = true
end

```
・・・・・・
設定内容をみれば、なんとなくやってることが掴めるかと、、、

PostgreSQLは9.2がyumからだとインストールできないようだったので、yumを使わないようにして、リポジトリを指定してます。

Vagrantfile上で仮想環境のIPアドレスが指定できるので、このIPアドレス宛にsshをすればvagrant sshをしなくても仮想環境に入れます。

この際鍵を指定しないといけないのですが、
```
vagrant ssh-config
```
をすると使用している鍵がわかるので、ssh実行時に指定してあげればOK。

あれ、そういえばポートは2222なのに特に指定しなくてもいけたな、、、

実際にはもう少し細かい設定ができるので、そこまでしておきたいなーと思ってます。
例えばapacheのドキュメントルートやpostgresqlでのデータを保存するディレクトリのパスなどの設定等です。

まだはっきり理解しきれてないところとかあるので、もう少しいろいろいじって簡単に開発環境をつくれる体制にしたい。
