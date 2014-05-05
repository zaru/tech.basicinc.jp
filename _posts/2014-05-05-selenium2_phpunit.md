---
layout: post
category : Selenium
title: Selenium2とPHPUnitでMac/Winブラウザ自動テスト
tagline: 
author : sakuraba
tags : [Selenium, PHPUnit]
---
{% include JB/setup %}

## Selenium2とは

ものすごいややこしいんだけど、SeleniumってSelenium◯◯っていうのが沢山あるよね…いったいどれが何なの？っていう迷子状態になって使うに至らない。そんな人、多いと思います。いや俺だけかも。そんな疑問に答えてくれる素晴らしいエントリー。

- [Selenium何とかっていうツールがやたら色々あるのはどういうわけなのか | 品質向上ブログ](http://blog.trident-qa.com/2013/05/so-many-seleniums/)

非常に助かります。要は巷で噂のSelenium2っていうのは、Selenium WebDriverの事。これさえ分かっていれば迷子にならない。と思う。

## いきなりまとめ

Macで開発して、PHPUnitでSeleniumのテストコードを書いて、MacブラウザとVirtualBox経由仮想Windowsブラウザに対してテストを実行する。それだけ。

必要なソフトは「PHPUnit」「PHPUnit/selenium」「SeleniumServer」「ブラウザドライバ」。それだけ。

## Selenium2 + PHPUnitを使ってテストしたい

PHPをメインで使っている場合、テストにはPHPUnitを使うわけだけど、SeleniumもPHPUnitからテストケース書いて実行したいっていう事で、試してみた。

構成としては普段コードを書いているMac開発機からPHPUnitを実行。Mac検証は開発機を兼用、Windows検証はVirtualBoxを使って仮想OS。それらに対してSeleniumServerを入れてテストを実行。

![selenium phpunit](/assets/img/2014-05-05-selenium_phpunit.png)

## 開発機にPHPUnitとPHPUnitSeleniumをインストール

まず開発機でPHPUnitを実行できないとお話にならない。今どきならComposerで入れるのが流行りなので、それに習って入れる。

### composer.json

```javascript
{
    "require-dev": {
        "phpunit/phpunit": "4.1.*"
        , "phpunit/phpunit-selenium": "dev-master"
    }
}
```

```bash
$ composer install
```

これでPHPUnit本体と、PHPUnitとSeleniumを連携する拡張機能もインストールされる（各種PHPフレームワークを使っている場合は、その流儀に則ってください）。

## PHPUnitの実行

ComposerでPHPUnitをインストールするとデフォルトでは ./vendor/bin/phpunit に配置されている。

```bash
$ ./vendor/bin/phpunit --version
PHPUnit 4.1.0 by Sebastian Bergmann.
```

## VirtualBox + modern.ieでWindows検証機を作る

開発環境がMacなので、Macの検証はいいとして、Webサービスをやっている限り、Windowsの検証も必要ですよ…というわけで、Microsoftが用意してくれているWindows仮想環境をmodern.ieから用意する。

- [modern.ie](http://modern.ie/ja-jp/virtualization-tools#downloads)

![selenium phpunit](/assets/img/2014-05-05-selenium_phpunit3.png)

適当に検証したいOSやIEのバージョンを選択して、一番下にある「Grab them all with cURL」をクリック→書かれているcurlコマンドを実行する。

ダウンロード完了後、拡張子sfxのファイルに実行言言を与えて実行すればVirtualBoxのOVAファイルができあがる。これを起動すればOK。

### Windowsにネットワークアダプターを追加

ホストOS（今回はMac）からゲストOS（Windows）に対してSeleniumServerで通信を行いたいので、ネットワークアダプターを追加する。

![selenium phpunit](/assets/img/2014-05-05-selenium_phpunit4.png)

- 「アダプター2」を追加
- 割り当て「ホストオンリーアダプター」
- 後は適当

これで起動すれば2つのネットワークが構成されて、ホストからゲストへ通信ができる。コマンドプロンプトで ipconfig を実行すれば割り当てられたIPアドレスもわかる。

## 検証機にSeleniumServerをインストール

実際にブラウザでテストを実行する検証機にSeleniumServerをインストールする。下記リンクからSeleniumServerをダウンロード。単なるjarファイルなので、適当な場所に配置する。

- [Selenium Donwload](http://docs.seleniumhq.org/download/)

ちなみに、HomeBrewでも入れられる。

```bash
$ brew install selenium-server-standalone
```

## 各種ブラウザのドライバをインストール

各種ブラウザを動かすにはドライバが必要になる。SeleniumServer標準だとFirefoxとSafariには対応済み。IEとChromeを動かすにはドライバが必要。

先ほどの[Selenium Donwload](http://docs.seleniumhq.org/download/)の中にあるThe Internet Explorer Driver Serverというのと、サードパーティのChromeDriverをダウンロードして、SeleniumServerのjarファイルと同じ位置に配置する。

- Mac HomeBrewで入れた場合 /usr/local/Cellar/selenium-server-standalone/2.41.0/bin に chromedriver を配置
- それ以外はjarと同じディレクトリ

Windowsの例

![selenium phpunit](/assets/img/2014-05-05-selenium_phpunit5.png)

Macの例

![selenium phpunit](/assets/img/2014-05-05-selenium_phpunit6.png)

ちなみに、iOSやAndroidなどのドライバもある。

### SeleniumServerの実行

テストを実行する前に、SeleniumServerを起動させておく。

#### jarをダウンロードした場合

Mac

```bash
$ java -jar selenium-server-standalone-2.41.0.jar
```

Windows

```bash
> java -jar selenium-server-standalone-2.41.0.jar \
	-Dwebdriver.ie.driver=./IEDriverServer.exe \
	-Dwebdriver.chrome.driver=./chromedriver.exe
```

#### HomeBrew経由で入れた場合

```bash
$ selenium-server -p 4444
```

エラーが何も出なければOK。

## PHPUnitでテストケースを書く

ようやくPHPUnitからSeleniumを実行するテストケースを書く。継承するクラスは、PHPUnit_Extensions_SeleniumTestCase じゃなくて PHPUnit_Extensions_Selenium2TestCase というのに注意。

### WebTest.php

```php
<?php
class WebTest extends PHPUnit_Extensions_Selenium2TestCase {
    public function setUp() {
        $targetUrl = 'http://stg.example.com/form/';

        $this->setHost('192.168.56.101');    // SeleniumServerがインストールされているホスト名
        $this->setPort(4444);           // SeleniumServerの稼働しているポート
        $this->setBrowser('firefox');   // firefox, chrome, iexplorer, safari
        $this->setBrowserUrl($targetUrl);
    }

    public function testTitle() {
        $this->url('/');
        $this->assertEquals('Title Hoge', $this->title());
    }
}
```

Selenium2TestCaseで使えるテストケース例については公式のスクリプトが参考になる。

- [phpunit-selenium/Tests/Selenium2TestCaseTest.php at master · sebastianbergmann/phpunit-selenium](https://github.com/sebastianbergmann/phpunit-selenium/blob/master/Tests/Selenium2TestCaseTest.php)

### PHPUnitを実行する

```bash
$ ./vendor/bin/phpunit --colors WebTest
```

![selenium phpunit](/assets/img/2014-05-05-selenium_phpunit2.png)

Firefoxが自動起動して、しゅぱっと消えると思います。そして、タイトルタグが指定通りになっているかのテストが見事にこけてくれました。これでテストを実行できる環境の出来上あり。

## 今後のテスト戦略

ひとまずPHPUnitとSeleniumを組み合わせて自動テストな環境の下地ができた。次はCIとの組み合わせや、データベースのテストとの組み合わせとか…かな。





