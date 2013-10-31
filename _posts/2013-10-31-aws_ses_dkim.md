---
layout: post
category : AWS
title: AWS SESでGmailのFromにamazonses.com経由と表示されるのを防ぐ
tagline: 電子署名DKIM
author : sakuraba
tags : [AWS, SES]
---
{% include JB/setup %}

AWSのSESを使ってメール配信を行っていると、時々Gmailのヘッダーに amazonses.com経由 と表示されます。

> メールが別のメール サービス経由で送信されたことを示しています。つまり、送信者がサードパーティのメール サービスを使ってこのメールを作成した可能性があることを示しています。たとえば、ソーシャル ネットワーキング サイトのメール機能を使って送信されたメールや、登録しているメーリング リストから送信されたメールなどが考えられます。
> 
> Gmail でこの情報を表示する理由は、代理でメールを送信するサービスの多くで、送信者が指定した名前がメール アドレスと一致するかどうかの確認が行われないためです。Google では、知り合いを装ったメールからユーザーを保護しようと努力しています。

[送信者名の横に詳細情報が表示される理由 - Gmail ヘルプ](https://support.google.com/mail/answer/1311182?hl=ja&ctx=mail)

とまぁ、理由はこんな感じなんですが、なるべく消したいですよと。

## 電子署名DKIMを設定する

AWS SESのコントロールパネルにいき、認証済みのメールアドレス一覧から、目的のメールアドレス詳細を開きます。DKIMという項目があるので、ここを開いて有効にします。すると、CNAMEレコードが3つ表示されるので、それをDNSレコードに設定します。

Route53を使っている場合は、そのまま登録できますし、自前のDNSでもCNAMEとして登録すればOKです。登録完了後、Amazonから確認メールが届くはずです。

その後、再びコントロールパネルへ行き、「enable」をクリックし、DKIMを有効化して完了です。これを忘れると、いつまでたっても反映されないので要注意。

![title](/assets/img/2013-10-31-aws_ses_dkim.png)

### アンダーバー(_)があって、DNSレコードに正常に登録できない

おそらくDKIMで提示されているホスト名は hoge._domainkey.exmaple.com というアンダーバー込みになっていると思います。DNSによってはエラーになって登録できないと思います。BINDの場合は、named.confの該当ゾーンに

```bash
check-names ignore;
```

と追加すればスルーで登録されます。

### CNAMEレコード例

```bash
hoge._domainkey.example.com IN CANME piyo.dkim.amazonses.com.
```

dkim.amazonses.com. の最後のピリオドを追加するのに注意。
