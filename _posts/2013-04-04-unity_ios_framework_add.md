---
layout: post
category : Unity
title: UnityのiOSアプリで、frameworkをビルド時に自動追加する
tagline: ""
author : sakuraba
tags : [Unity, iOS, Ruby]
---
{% include JB/setup %}

UnityでiOSアプリを開発していくと、iOSのframeworkをビルド時に手動で導入しなくてはならずかなり面倒です、というわけでそれを解決するスクリプトを書いてくれている偉大な人がいるので紹介＆メモ。

ちなみに、Pro版のライセンスだとBuildPipeline.BuildPlayerを使って一貫してC#で書けるみたい。でもPro版じゃないのでPostprocessBuildPlayerを使います。

ただし、Mac専用です。

## Ruby gem xcodeproj をインストールする

今回使用するスクリプトはxcodeprojというgemライブラリが必要なので

	$sudo gem install xcodeproj -v 0.3.0

とインストールしておきます。0.3.0じゃないとエラーになるのでバージョン指定しておきます。

## スクリプトを配置する

Unityのプロジェクトディレクトに /Assets/Editor というディレクトリ作って下記スクリプトを配置します。

<script src="https://gist.github.com/zaru/5308939.js"></script>

これで後はビルドするだけでOK。Xcodeに自動的にframeworkが追加されていると思います。

### 参考サイト

- [http://mkgames.me/archives/155](http://mkgames.me/archives/155)
- [http://akisute.com/2012/09/unity-postprocessbuildplayer-weak.html](http://akisute.com/2012/09/unity-postprocessbuildplayer-weak.html)