---
layout: post
category : Unity
title: UnityでiOSのネイティブコードを呼び出す最も簡単でシンプルな方法
tagline: C# → Objective-C
author : sakuraba
tags : [Unity, Objective-C, C#, iPhone]
---
{% include JB/setup %}

[UnityでAndroid JARファイルを呼び出す最も簡単な方法 - Androidネイティブプラグイン](http://tech.basicinc.jp/Unity/2013/04/14/unity_andoird_jar_plugin/) こちらで、AndroidJARファイルをUnityから呼び出す方法を紹介しましたが、今回はiOSのネイティブコードを呼び出す方法を紹介します。

## C#スクリプトでネイティブコードを呼ぶ

```c#
using UnityEngine;
using System.Runtime.InteropServices;

public class NativePlugin { 
	[DllImport("__Internal")]
	private static extern void hoge_();
	public static void hoge() {
		if (Application.platform != RuntimePlatform.OSXEditor) {
			hoge_();
		}
	}
}
```

ファイル名はクラス名に合わせて、NativePlugin.cs とでもして、/Assets/Plugins/ 以下に配置します。

C#としては、hoge() というメソッドを作り、その中で extern した hoge_() を実行します。もちろん引数なども使えます。

## ネイティブコードを用意する

```objective-c
extern "C" {
	void hoge_();
}
void hoge_() {
	NSLog(@"#hoge");
}
```

ファイル名は NativePlugin.cs のクラス名に合わせて NativePlugin.mm として、/Assets/Plugins/iOS/ 以下に配置します。

準備はこれだけです。

## ネイティブコードを呼び出す

NativePlugin.cs を直接使ってもいいんですが、使いまわすことが多いと思うので、NativePlugin.cs を呼ぶ Test.cs を作ります。

```c#
using UnityEngine;
using System.Collections;
using System.Runtime.InteropServices;

public class Test : MonoBehaviour {
	void Start () {
		NativePlugin.hoge();
	}
	void OnGUI() {
	}
}
```

あとはTest.csをUnityのシーンに関連付けてあげれば起動時に、NSLog() が実行されます。簡単ですね。

## Unityのシーン上にUIView的なものを置きたい

広告バナーだったり、なんかWebView的なものでもいいんですが、Objective-Cで作ったなんかしらをUnityの上に載せたいことってありますよね。それも簡単に出来ます。

Objective-C側でUnityのUIViewControllerを捕まえられます。

```objective-c
extern UIViewController* UnityGetGLViewController();

void hoge_() {
	UIViewController* parent = UnityGetGLViewController();
	
	UIView *uv = [[UIView alloc] init];
	uv.frame = CGRectMake(0, 0, 100, 100);
	uv.backgroundColor = [UIColor blueColor];
	
	[parent.view addSubview:uv];
}
```
これでUnityのルートビューを好き勝手できます。

