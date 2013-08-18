---
layout: post
category : php
title: PHPでエラトステネスの篩で素数判定をやってみる
tagline: 『素数』を数えて落ち着くんだ…
author : sakuraba
tags : [PHP]
---
{% include JB/setup %}

> 落ち着け…心を平静にして考えるんだ…こんな時どうするか…
> 
> ２…　３　５…　７…　落ち着くんだ…『素数』を数えて落ち着くんだ…
> 
> 『素数』は１と自分の数でしか割ることのできない孤独な数字……
> 
> わたしに勇気を与えてくれる

ジョジョファンじゃなくても神父じゃなくても時には素数を数えたくなる夜ってありますよね。なんかそんな気分になったのでPHPで、2からnまでの数字の中から、素数を判定するプログラムを書いてみました。こんな感じで。素数判定なんてとても簡単ですね。

```php
<?php
function normal($max) {
	$primeLists = array();
	for ($i=2; $i<=$max; $i++) {
		$isPrime = true;
		for ($k=2; $k<$i; $k++) {
			// 割り切れた数が存在したらアウト
			if ($i % $k === 0) {
				$isPrime = false;
				break;
			}
		}
		if ($isPrime) {
			$primeLists[] = $i;
		}
	}
	return $primeLists;
}
print_r(normal(10000));
# 実行時間 : 0.37249秒
```

10000までの素数を半指定してみた。なんという力技…。素数判定するコード書けって言われてこのコード出してきたら偉い人から殴られそう。実際、殴られる。

## エラトステネスの篩

世の中には頭のいい人がいるもので、エラトステネスの篩という素数判定アルゴリズムがあります。とても単純明快パワフル。アルゴリズムをWikipediaから引用してみます。

> ステップ 1
> 
> 探索リストに2からxまでの整数を昇順で入れる。
> 
> ステップ 2
> 
> 探索リストの先頭の数を素数リストに移動し、その倍数を探索リストから篩い落とす。
> 
> ステップ 3
> 
> 上記の篩い落とし操作を探索リストの先頭値がxの平方根に達するまで行う。
> 
> ステップ 4
> 
> 探索リストに残った数を素数リストに移動して処理終了。

[エラトステネスの篩 - Wikipedia](http://ja.wikipedia.org/wiki/%E3%82%A8%E3%83%A9%E3%83%88%E3%82%B9%E3%83%86%E3%83%8D%E3%82%B9%E3%81%AE%E7%AF%A9)

というわけで、さっそくコードにしてみる。

## なんにも考えず実装してみる

```php
<?php
function eratosthenes($max) {
	$lists = array();
	$primeLists = array();
	
	// maxの平方根
	$sqrt = floor(sqrt($max));
	
	for ($i=2; $i<=$max; $i++) {
		$lists[$i] = $i;
	}
	
	while ($val = array_shift($lists)) {
		$primeLists[] = $val;
		// maxの平方根に達したら終了
		if ($val > $sqrt) {
			break;
		}
		foreach ($lists as $key2 => $val2) {
			if ($val2 % $val === 0) {
				unset($lists[$key2]);
			}
		}
	}
	$primeLists = array_merge($primeLists, $lists);
	return $primeLists;
}
print_r(eratosthenes(10000));
# 実行時間 : 0.01017秒
```

Wikipediaにある通り、そのまんま実装してみました。うーん、最初の実装よりは大分まし（約36倍）になったけど、まだ無駄があるっぽい。そもそも起点となる数字の倍数しか取り除かないんだから、倍数のみでループすればいいんじゃね、というわけでちょっと改善。

## 倍数のみでループしてみる

```php
<?php
function eratosthenes2($max) {
	$lists = array();
	$primeLists = array();
	
	// maxの平方根
	$sqrt = floor(sqrt($max));
	
	for ($i=2; $i<=$max; $i++) {
		$lists['k' . $i] = $i;
	}
	
	while ($val = array_shift($lists)) {
		$primeLists[] = $val;
		// maxの平方根に達したら終了
		if ($val > $sqrt) {
			break;
		}
		// 倍数だけ対象にしてみる
		for ($j=$val*2; $j<=$max; $j+=$val) {
			unset($lists['k' . $j]);
		}
	}
	$primeLists = array_merge($primeLists, $lists);
	return $primeLists;
}
print_r(eratosthenes2(10000));
# 実行時間 : 0.00721秒
```

約30％ほど高速化に成功。でも、もっとシンプルに書けるな。

## よけいなコードを削ってみる

```php
<?php
function eratosthenes3($max) {
	
	// maxの平方根
	$sqrt = floor(sqrt($max));
	
	for ($i=2; $i<=$max; $i++) {
		$lists[$i] = $i;
	}
	
	// もっと単純に倍数を取り除くだけ
	for ($i=2; $i<=$sqrt; $i++) {
		if (isset($lists[$i])) {
			for ($j=$i*2; $j<=$max; $j+=$i) {
				unset($lists[$j]);
			}
		}
	}
	
	return $lists;
}
print_r(eratosthenes3(10000));
# 実行時間 : 0.00284秒
```

大幅に改善。

最初のリスト（2〜$max）を作る部分をもっと効率化出来るはず。というわけで、range() と array_fill() を使ってみる。

## range() を使ってみる

```php
<?php
function eratosthenes4($max) {
	
	// maxの平方根
	$sqrt = floor(sqrt($max));
	
	// rangeで一気に作っていらない数を捨てる
	$lists = range(0, $max);
	unset($lists[0]);
	unset($lists[1]);
	
	// もっと単純に倍数を取り除くだけ
	for ($i=2; $i<=$sqrt; $i++) {
		if (isset($lists[$i])) {
			for ($j=$i*2; $j<=$max; $j+=$i) {
				unset($lists[$j]);
			}
		}
	}
	
	return $lists;
}
print_r(eratosthenes4(10000));
# 実行時間 : 0.00221秒
```

range() は指定範囲の配列を作ってくれるが、2から作るとkey-valの組み合わせが崩れてしまうので、敢えて0から指定していらない部分を削除するようにしてみた。ちょっとださい。

## array_fill()を使ってみる

```php
<?php
function eratosthenes5($max) {
	
	// maxの平方根
	$sqrt = floor(sqrt($max));
	
	// keyで配列を一気に作る
	$lists = array_fill(2, $max-1, true);
	
	// もっと単純に倍数を取り除くだけ
	for ($i=2; $i<=$sqrt; $i++) {
		if (isset($lists[$i])) {
			for ($j=$i*2; $j<=$max; $j+=$i) {
				unset($lists[$j]);
			}
		}
	}
	
	return array_keys($lists);
}
print_r(eratosthenes5(10000));
# 実行時間 : 0.00167秒
```

array_fillを使ってkeyに指定範囲の配列を作ってみた。今まではvalの方を使っていたけど逆バージョン。こっちの方が早かった。というわけで、僕がなんとなく書いてみたPHPエラトステネスの篩でした。後半はなんかアルゴリズムというか、PHP関数によるパフォーマンス差比べになってしまった…。

もっとスマートに書く方法がある気がする。

ともかくこれで安心して素数を数えて落ち着くことが出来る。