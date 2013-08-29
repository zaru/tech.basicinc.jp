---
layout: post
category : munin
title: AWS RDSをMuninでリソース監視する
tagline: Muninプラグインの作り方
author : sakuraba
tags : [RDS, munin]
---
{% include JB/setup %}

AWSの花形…かどうかは知らないけど、とても使いやすいRDS。サクッとスペックに合ったチューニングでバックアップもMulti-AZもリードレプリカも簡単に作れて、とても便利ですよね。

でも、いわゆる普通のDBサーバと違って出来ないことも多いです。その一つが単純なMuninでのリソース監視。一応、AWS Consoleの方で用意されているCloud Watchがありますが、若干使いにくいし過去分が消えていってしまう…。

というわけで、RDSをCloud Watch APIを使ってMuninのプラグインを作ってみました。

## AWS RDS Muninプラグイン

[zaru/rds-munin-plugin](https://github.com/zaru/rds-munin-plugin)

GitHubにプラグインのソースを置いてあります。現バージョンだと、一つのRDSインスタンスしか監視できません。使い方はGitHubにも書いてありますが、Muninサーバに該当ソースを配置して、AWSの認証キーを設定するだけです。

```bash
cd /usr/share/munin/plugins
wget https://github.com/zaru/rds-munin-plugin/archive/master.zip
unzip master.zip
vi rds-munin-plugin-master/rds_config.php
cd /etc/munin/plugins
ln -s /etc/munin/plugins/rds_cpu /usr/share/munin/plugins/rds-munin-plugin-master/rds_cpu.php
ln -s /etc/munin/plugins/rds_mem /usr/share/munin/plugins/rds-munin-plugin-master/rds_mem.php
ln -s /etc/munin/plugins/rds_connection /usr/share/munin/plugins/rds-munin-plugin-master/rds_connection.php
```


![title](/assets/img/2013-08-29-rds-munin.png)

こんな感じの計測グラフが出力されると思います。



## Muninプラグインの作り方

今回Muninプラグインを作ってみて、思っていた以上に簡単だったので、備忘録的に作り方のメモ。

### プラグインの場所

Muninプラグインは /usr/share/munin/plugins/ に置かれています。

実際に実行されるプラグインは /etc/munin/plugins にシンボリックリンクで設置されています。

### プラグインの仕様

決まったフォーマットで出力されてさえいれば良いので、実装する言語はなんでもOKです。BashでもPerlでもPHPでもRubyでも。実際に、プラグインの挙動を見てみると理解しやすいと思います。

```bash
$/usr/share/munin/plugins/ntp_kernel_err autoconf
yes

$/usr/share/munin/plugins/ntp_kernel_err config
graph_title NTP kernel PLL estimated error (secs)
graph_vlabel est. err (secs)
graph_category time
graph_info The kernels estimated error for the phase-locked loop used by NTP.
ntp_err.label est-error
ntp_err.info Estimated error for the kernel PLL

$/usr/share/munin/plugins/ntp_kernel_err
ntp_err.value 0.001613
```

autoconf と config という2つのパラメータと、パラーメータなしに対応すれば基本動きます。

#### autoconf

プラグインが利用可能な場合 yes を返し、利用不可なら no を返します。これは munin-node-configure を使って利用可能なプラグインを自動で追加／削除するために使ったりします。

#### config

Muninの出力するHTMLやグラフを描画したりするのに使います。

|項目名|内容|
|---|---|
|graph_title|グラフのタイトル|
|graph_args|Kbyte単位の設定など|
|graph_vlabel|縦軸のラベル|
|graph_category|Muninの計測カテゴリのどこに所属するか|
|graph_info|グラフの紹介|
|フィールド名.label|グラフに描画したい値のタイトル|
|フィールド名.draw|グラフ描画スタイル（LINE1 / LINE2 ...など）|
|フィールド名.info|値の紹介|

詳しくは、[protocol-config – Munin](http://munin-monitoring.org/wiki/protocol-config) を参照して下さい。

一つのグラフに複数描画することももちろんできるので、その場合は上記表の下3つ「フィールド名.*」の部分を個数分記述をして下さい。

フィールド名というのは、ntp_kernel_errプラグインで言うと、ntp_err という名前です。

```bash
$/usr/share/munin/plugins/ntp_kernel_err
ntp_err.value 0.001613
```

### パラメータなし

実際にグラフ描画するための値を出力します。

フィールド名.value 値

というフォーマットです。1つのグラフに複数の値を描画する場合はふくす行にわたって出力してあげればOKです。


