---
layout: post
category : Objective-C
title: NSNotificationでNSObject同士だと通知されない問題
tagline: 
author : sakuraba
tags : [Objective-C]
---
{% include JB/setup %}

NSNotification便利ですよね。デリゲートが実装しづらいようなパターンの場合に大活躍。そういえば、みんな通知とデリゲートの使い分けって、どうしてるんだろ。僕がNSNotificationを使うのは、通知先が複数ある／オブジェクトがものすごく遠い…っていう感じで使い分けているけど。まぁ、デリゲートよりも書くの楽なのでついつい使ってしまって、後からコードが追いにくくなるという諸刃の刃。

それはともかく、NSNotificationを使って通史を受信できないという事態に遭遇したのでメモ。

## NSObjectとUIViewControllerの通知／受信の場合

### 通知側

```objective-c
// 通知の送信
NSNotification *n = [NSNotification notificationWithName:@"HogeComplete" object:self];
[[NSNotificationCenter defaultCenter] postNotification:n];
```

### 受信側

```objective-c
// 通知受取の登録（通常は、 addObserver こちら）
NSNotificationCenter *nc = [NSNotificationCenter defaultCenter];
[nc addObserver:self selector:@selector(HogeComplete) name:@"HogeComplete" object:nil];

// 受け取り時の実行メソッド
-(void)HogeComplete{
    NSLog(@"通知受信完了");
}
```

まぁ、普通はこう書きますよね。でも、NSObject同士でやると受信できない！しかもエラーにもならない。そりゃそうだ。

## NSObject同士の通知／受信の場合

### 受信側

```objective-c
// NSObject同士の場合は、 addObserverForName こちらを使う
NSNotificationCenter *nc = [NSNotificationCenter defaultCenter];
[nc addObserverForName:@"HogeComplete"
                object:nil
                 queue:nil
            usingBlock:^(NSNotification *note) {
                [self HogeComplete];
            }];
```

NSObjectの場合は、受信側を addObserverForName こちらで登録する必要がある…と。

