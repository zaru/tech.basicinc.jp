---
layout: post
category : Unity
title: UnityからAndroid Javaのインスタンスメソッドを呼び出す
tagline: 
author : sakuraba
tags : [Unity, MySQL]
---
{% include JB/setup %}

UnityからAndroidのネイティブコードを呼び出すのは、かなり簡単…なんだけど、Staticじゃないインスタンスメソッドを呼び出すには、なんかちょっと一工夫必要だったので、メモ。

↓参考
[UnityでAndroid JARファイルを呼び出す最も簡単な方法 - Androidネイティブプラグイン](http://tech.basicinc.jp/Unity/2013/04/14/unity_andoird_jar_plugin/)

Staticなメソッドを呼び出すには、単に CallStatic() すれば良いのだけど、インスタンスメソッドの場合は、当然インスタンス化しなくてはならないので、インスタンスを返すだけのStaticメソッドを用意。それを AndroidJavaObject として受け取って、Call() するだけ。

## Unityスクリプト
```c#
AndroidJavaClass plugin = new AndroidJavaClass("jp.example.Hoge");
AndroidJavaObject jo = plugin.CallStatic<AndroidJavaObject>("instance");
jo.Call("piyo");
```

## Java
```java
class Hoge {
	public static Hoge instance() {
		return new Hoge();
	}
	
	public void piyo() {
	}
}
```

これだけで、UnityからJavaのインスタンスメソッドを呼び出せます。