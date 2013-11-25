---
layout: post
category : Unity
title: UnityからJavaネイティブコードに使う引数はプリミティブbooleanじゃなきゃダメ
tagline: 
author : sakuraba
tags : [Unity, Android]
---
{% include JB/setup %}

またまたUnity + Javaな話題。

UnityからJavaネイティブコードを呼び出す際に、引数を渡すのはよくあるシーン。そこで、Booleanな値を渡すのもよくあるシーン。何も考えずにBoolean型で設定していたら、死んだ。

## ダメな例

### Unityスクリプト

```c#
plugin.CallStatic("hoge", true);
````

### Java

```java
public static hoge(Boolean isHoge) {
}
```

## 大丈夫な例

### Unityスクリプト

```c#
plugin.CallStatic("hoge", true);
````

### Java

```java
public static hoge(boolean isHoge) {
}
```

要は、ラッパークラスのBooleanだとUnityからちゃんと渡せずエラーになってしまいます。ちゃんとプリミティブなbooleanを使用しましょう…という。むむむ。