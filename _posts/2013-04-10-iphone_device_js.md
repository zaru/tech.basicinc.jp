---
layout: post
category : JavaScript
title: iPhone5/4_4S/3G_3GSを判定するJavaScript
tagline: 
author : sakuraba
tags : [JavaScript, iPhone]
---
{% include JB/setup %}

```javascript
(function(){
	if (navigator.userAgent.search('iPhone') !== -1) {
		if (window.screen.height === 568) {
			alert('iPhone5');
		} else if (window.devicePixelRatio > 1) {
			alert('iPhone4-4S');
		} else {
			alert('iPhone3-3GS');
		}
	}
})();
```

UserAgentだけだと、機種の判定までは出来ないので画面解像度と画面密度を表すdevicePixelRatioで判定。ただし、iPhone4とiPhone4S・iPhone3GとiPhone3GSは完全には判定できない…。

なんか良い方法ないんだろうか。