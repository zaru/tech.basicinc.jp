---
layout: post
category : Android
title: Android4.2.xではaddJavascriptInterfaceが動かない＋対処法
tagline: 
author : sakuraba
tags : [Android, Java]
---
{% include JB/setup %}

AndroidのWebViewでJavaScriptからJavaのコードを呼び出すのに便利なJavascriptInterface。ただし、セキュリティ上、HTMLからAndroidアプリを操作することが可能なので、使うのは要注意ねっていうもの。

Android 4.2.x以降、このJavaScriptInterfaceを使うには@JavascriptInterfaceアノテーションを付加していないと呼び出せないようになりました。やり方は簡単。

```java
// Annotation is needed for SDK version 17 or above.
@JavascriptInterface
public void doSomething(String input) {
   . . .
}
```

これだけ。

- [参考 addJavascriptInterface - Android APIs](http://developer.android.com/reference/android/webkit/WebView.html#addJavascriptInterface(java.lang.Object, java.lang.String\))
