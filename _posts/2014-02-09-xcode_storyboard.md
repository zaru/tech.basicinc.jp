---
layout: post
category : Objective-C 
title: Xcode5 + StoryBoardでiOSアプリを作る
tagline: 
author : sakuraba
tags : [Xcode, Objective-C, StoryBoard]
---
{% include JB/setup %}

![StoryBoard](/assets/img/2014-02-09-xcode1.jpg)

今回、個人的にXcode5 + StoryBoardな環境でiOSアプリを作ってみたので、その時、つまずいたポイントやメリット・デメリットなどをまとめてみました。結論から言うと、StoryBoardは使える！代わりに、慣れるまでは苦行。（今までUIKitをコードで実装していた人は特に）

正直、GUIツールなどを使ってのアプリ構築には苦手意識があって（プログラマなら、コードで全部書けよ的な変な意識）、StoryBoardはおろか、InterfaceBuilderは完全に避けてました。

ただそうは言っても、一度は本気で取り組まないと文句も言いにくいし。UnityみたいにGUI実装前提も出てきているので、やっておこうかなと。

## StoryBoardつまずきポイント

### それどこで設定するの？

例えば、UINavigationBarの色ってどこで変えるの？っていう超初歩的なもので、つまずいたりしました。コードで変更しようと思っても、Navigation Controllerに対応するクラスファイルもないしなぁとかボケたこと考えてました（作れよ）。

コントロールキーを押しながら、ソース画面上にドロップすると連携できるようになるとか、ぜんぜん分からんかったし、どこにドロップすればいいのかも、いまいち分からんかったし、連携されているかどうかはStoryBoardのInspectorのところを見ないとわからないし、イライラは募る。

でも、じょじょにどこに何があるかを把握し始めてからは、いちいちクラスファイル作ってコードを書いてビルドしてっていう手順を踏んでいたのが、画面上で画面をどんどん作って、全体の遷移図が完成した後、レイアウトを調整するだけで、とりあえずのモックレベルは完成できるという手軽さ。

### Viewに対応するクラスファイルを作成＆指定する

まずStoryBoard上で、適当にViewを作ります（ViewContorollerやTableViewControllerなど）。その後、コマンド + Nでサブクラスファイルを新規作成、継承クラスはStoryBoardで作ったものを指定。

後は、StoryBoardで対象のViewを選択し、右側のインスペクタからCustomClassを指定すれば連携完了。

後はサブクラスファイルにゴリゴリ、ビジネスロジック書いていけばOK。

![StoryBoard](/assets/img/2014-02-09-xcode2.png)

### 画面遷移を設定する

画面遷移もStoryBoardだと簡単で、タップイベントを設定したいViewを選択し、コントロールキーを押しながら遷移先のViewContorllerへドラッグするだけ。後は、pushなのかmodalなのかcustomなのかを選択。

![StoryBoard](/assets/img/2014-02-09-xcode3.png)

ちなみに、このViewController同士をつないだあとにできる青い線はSegue（セグエ）というらしい。Segue自体に名前をつけて、遷移時にデータ受け渡しとか処理が色々できる。

```objective-c
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{
    if ([[segue identifier] isEqualToString:@"detailData"]) {
        DetailViewController *viewController = (DetailViewController*)[segue destinationViewController];
        viewController.detailData = [self.hogeData objectForKey:@"Piyo"];
    }
}
```

### UILableとかのオブジェクトをクラスファイルと関連付けたい

![StoryBoard](/assets/img/2014-02-09-xcode4.png)

これも簡単で、対象のUILabelなどを選択後、クラスファイルの@interfaceへコントロールキーを押しながらドラッグするだけ。勝手に@propertyを作ってくれます。

```objective-c
@interface TableViewController ()
@property (weak, nonatomic) IBOutlet UILabel *label;
@end
```

### UIButtonとかのイベントを設定したい

![StoryBoard](/assets/img/2014-02-09-xcode5.png)

イベント設定も、UILabelとかと同じように対象のクラスファイルへコントロールキーを押しながらドラッグ。勝手にイベントメソッドを作成してくれる。

### StaticCellsはTableViewControllerでないと使えない

StoryBoardとは関係ないけど…普通のUITableViewでStaticCellsを使おうと思ったら、なんか使えない…なぜ？と思ったら、TableViewControllerじゃないと使用不可…。

StoryBoardの設定時には、なんのエラーも出ないので、原因探るのに時間かかった。なぜかStoryBoardって、エラーとか選択不可とかそういう事、してくれない。

## StoryBoardのメリット・デメリット

とにかく簡単にアプリ全体の画面設計図を作れ、コード記述量が減り、よりMVCっぽい作り方が出来るのが最大のメリットだと思いました。特にクラスファイルにViewなコードで溢れかえっていたのが、ほとんどなくなったのは気持ちよかったです。

ただ、複雑なレイアウト・デザインを作ろうとすると結構苦労します。StoryBoardのインターフェイス上、細かい作業はあまり想定しないっぽい…。画面遷移と大きなパーツ配置はStoryBoardで、その後はコードで実装っていうハイブリッドな感じが良いかも。

あとは全部のViewをStoryBoardで実装できるわけでもなく、インジケータなどは普通にコードで実装したりしました。StoryBoardで出来る出来ないの判断がつきにくく、ググりまくって時間取られるとか、よくありました…。

とはいえ、慣れれば今まで以上に早くアプリを作ることが出来そうだという実感はつかめたので、今後は積極的に活用していきたいところ。

ものによっては、プロトタイプと割りきってStoryBoardを使用するっていうのもアリだと思います。

あ、チーム開発している場合、ひとつのStoryBoardファイルをさわることになるので、分割するか、事前に調整しておかないと大変なことになると思います…。ココらへん、みんなどうしているんだろう。


