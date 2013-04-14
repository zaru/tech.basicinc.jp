---
layout: post
category : Unity
title: UnityでAndroid JARファイルを呼び出す最も簡単な方法
tagline: Androidネイティブプラグイン
author : sakuraba
tags : [Unity, Android, C#, Java]
---
{% include JB/setup %}

最近、Unityなゲームアプリ増えてますね。この手のワンソースマルチプラットフォーム対応ゲームエンジンは、広告などネイティブで実装しなくちゃならない部分が面倒だったりします。というわけで、UnityからJavaのJARファイルを呼び出す最も簡単な方法を紹介します。

## JARファイルの配置

/Assets/Plugins/Android ディレクトリに目的のJARファイルを配置します。その他リソースファイルが必要であればそれも同じように配置します（assetsなど）。

アクティビティなどでAndroidManifest.xmlが必要な場合は、それも配置します。もし、他のプラグインでも使う場合は、必要な部分をマージして一つのマニフェストにまとめます。

こんな感じです。

![](/assets/img/2013-04-14-1.png)

## JARを呼び出すC#スクリプトを作る

次に、JARのクラスを呼び出すためのC#スクリプトを書きます。

```csharp
using UnityEngine;
using System.Collections;

public class  HogeObserver : MonoBehaviour {
	
	void Start () {
		//呼び出すJARのパッケージ名を指定
		AndroidJavaClass plugin = new AndroidJavaClass("jp.example.AppController");
		AndroidJavaClass unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
		AndroidJavaObject activity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity");
		//JARのメソッドを指定する
		plugin.CallStatic("show", activity);
	}
}
```

このスクリプトを /Assets/Plugins に作成します。その後、適当なシーンに関連付けることで完成です。ものすごく簡単ですね。iOSは…もうちょっと面倒です。

UnityからJavaネイティブコードを呼び出す方法は、JNIとかいくつかあるけれど、JARで呼び出すだけなら、こっちの方が楽です。