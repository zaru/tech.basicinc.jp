---
layout: post
category : Objective-C
title: Objective-Cで外部ホスト画像の読み込みを非同期で行う
tagline: 
author : sakuraba
tags : [Objective-C]
---
{% include JB/setup %}

UIImageViewで外部ホストの画像を読み込むことってかなり頻繁にあると思いますが、何も考えずに書きみたいなコードを書くと、画像読み込み時にアプリケーションがフリーズしたかのような挙動になります。要は同期的に読み込み処理が走って、描画や操作の処理がストップするわけです。APIとか通信の非同期化はちゃんとしてても、ここはしてないっていうケースがあるのでメモ。

## 何も考えずにUIImageViewで画像読み込み

```objective-c
NSString *imageURL = @"http://www.example.com/hoge.jpg";
UIImageView *iv = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, 100, 100)];
UIImage *img = [UIImage imageWithData:[NSData dataWithContentsOfURL: [NSURL URLWithString: imageURL]]];
iv.image = img;
[self addSubview:iv];
```

表示はされます。だけど、重い画像や、こんなUIImageViewが何個もあると確実に表示＆操作がもたつきます。

## dispatch_async使って非同期で画像を読み込む

```objective-c
NSString *imageURL = @"http://www.example.com/hoge.jpg";
UIImageView *iv = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, 100, 100)];

dispatch_queue_t q_global = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_queue_t q_main = dispatch_get_main_queue();
dispatch_async(q_global, ^{
    UIImage *img = [UIImage imageWithData:[NSData dataWithContentsOfURL: [NSURL URLWithString: imageURL]]];
    
    // UI操作はメインスレッドで行う
    dispatch_async(q_main, ^{
        iv.image = image;
        [self addSubview:iv];
    });
});
```

画像の通信部分は非同期で、描画する部分はメインスレッドで行う感じです。これだけでUIImageViewが何個あっても操作が邪魔されるようなことはなくなります。