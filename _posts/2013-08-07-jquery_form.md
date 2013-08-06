---
layout: post
category : jQuery
title: jQueryでのよくあるフォーム操作まとめ
tagline: 基本のキ 
author : kobayashi
tags : [jQuery, JavaScript]
---
{% include JB/setup %}

たまーーーにあれどうやるんだっけ？ということがあるので、jQueryでよくあるフォーム操作についてまとめました。

## テキストボックス

### 値を取得する

	jQuery("input['name=target']").val();

## セレクトボックス

singleとmultipleで微妙に違ってます。

### 選択している値を取得する(single)

	jQuery("input['name=target'] option:selected").val()

option:selectedはなくてもOK

### 選択しているテキストを取得する(single)
	
	jQuery("input['name=target'] option:selected").text()


こっちはoption:selectedがないと全部のテキストがとれてしまいます。

### 選択している値を取得する(multiple)

	jQuery("input['name=target'] option:selected").each(function() {
		 data.push(jQuery(this).val());
	}

もしくは

	jQuery("input['name=target']").val();

だけでも配列で取得できます。

### 選択しているテキストを取得する(multiple)

	jQuery("input['name=target'] option:selected").each(function() {
		data.push(jQuery(this).text());
	}

複数選択なのでループさせます。

### 選択させる

	jQuery("input['name=target']").val("hogehoge")  //valueで指定
	jQuery("select[name='target']").prop("selectedIndex", index)    //indexで指定


### 要素の追加

	jQuery("input['name=target']").append(jQuery('<option value="1">test1</option>'));

普通にappendでoptionタグを追加します。


## ラジオボタン

### 選択している値を取得

	jQuery("input['name=target']:checked").val()

checkedが肝

## チェックボックス

### 選択している値を取得

	jQuery("input[name='target']:checked").each(function() {
		data.push(jQuery(this).val());
	}

セレクトボックスみたいに自動で配列で取得してくれないのでループさせる必要があります。


### チェックON / OFF（単体）

	var checkflg = jQuery("#target").attr("checked");
	if(checkflg) {
		jQuery("#target").attr("checked", false);
	}else{
		jQuery("#target").attr("checked", true);
	}

attrを使ってchecked属性の値を操作します。

### チェックON / OFF（全部）

	jQuery("input[name='target']").attr("checked", true);   //全チェック
	jQuery("input[name='target']").attr("checked", false);  //全チェック外し

よくある「全てにチェック」をいれると全部のチェックをいれるやつです。


## テキストボックス

	jQuery("textarea[name='target']").text()

text()で値を操作できます。
