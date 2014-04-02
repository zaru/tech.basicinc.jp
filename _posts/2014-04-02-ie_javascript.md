---
layout: post
category : JavaScript
title: IE9以下でSCRIPT438エラーするJavaScriptコード、id名と変数名がかぶると駄目？
tagline: 
author : sakuraba
tags : [JavaScript]
---
{% include JB/setup %}

社内の古（いにしえ）のWebサービスのメンテをしている過程で、InternetExplorer9以下の時、JavaScriptが正常に動いていない状況に出くわした…。未だに根本的な原因が分からず。誰か教えて…。

__SCRIPT438: Object doesn't support property or method IE__

これがF12の開発者ツールで表示されるエラー。

## 該当コード

```html
<html>
<body>
<div id="hoge"></div>
<script>
hoge = document.getElementById("hoge"); // エラーにならない
function piyo() {
	hoge = document.getElementById("hoge"); // エラーになる
	var hoge = document.getElementById("hoge"); // エラーにならない
}
piyo();
</script>
</body>
</html>
```

これをIE9以下で表示させると上記のSCRIPT438エラーになる。

### 発生条件

- DOCTYPEなしの互換表示モード（Quirksモード）
- var 宣言なし
- 関数内でのみ再現
- HTMLのid名と変数名が同じ

なぜか、id="hoge" と hoge変数の名前がかぶるとエラーになる。

### 対処方法

- DOCTYPEちゃんとする
- var 宣言ちゃんとする

まぁ、要はちゃんと書いていれば問題が起こらないようなケースなんだけど、古（いにしえ）すぎるWebサイトの場合、なんかよくありがちなケースに思える…。
