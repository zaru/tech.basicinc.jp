---
layout: post
category : Git
title: Gitの忘れがちだけど絶対に使うであろうコマンド達
tagline: Git可愛いよGit
author : sakuraba
tags : [Git]
---
{% include JB/setup %}

ベーシックでは、Gitを使ったバージョン管理システムを導入しています。一部のプロジェクトでは先行して導入していたものの、全社的にはまだまだ…といったわけで、よくGitコマンドについて質問されるので、ここで軽くまとめておきたいと思います。

普段は git add / commit / push / pull しかしてない…っていう人向けです。

## addしたファイルを取り消す

```bash
$git reset HEAD ファイル名
```

更新内容自体は取り消さず、addしてインデックスに登録するのを取り消します。

## 更新したファイルの更新内容を取り消す

```bash
$git checkout ファイル名
```

commitする前限定です。

## 他ブランチの特定のコミットだけマージしたい

```bash
$git cherry-pick コミットID
```

とても便利なコマンドですが、cherry-pickを多用するような運用スタイルになっていたら問題なので、ブランチの切り方などを見なおしたほうが良いでしょう。

## コミットしたけど失敗した！コミットをやり直したい。

```bash
$git commit --amend
```

その他：ファイルをaddし忘れた・コミットメッセージをミスったなど。ただし、リモートリポジトリへpushする前に限る。

## コミットを取り消して、指定の位置まで戻りたい

```bash
$git reset コミットID
```

指定したコミットIDのコミットを取り消すのではなく、そこまで戻るという意味。ただし、リモートリポジトリへpushする前に限る。

## 全てをなかったコトにして戻りたい

```bash
$git reset --hard コミットID
```

指定したコミットまで、全ての更新をなかったコトにします。ただし、リモートリポジトリへpushする前に限る。

## コミットをハードリセットしてしまった…死にたい

上記 git reset --hard でコミットを闇に葬ってしまってから、しまった！となったオッチョコチョイなエンジニアのためにもgitは配慮してくれています。すべての行動は記録されているのです。

```bash
$git reflog
7a4b4b6 HEAD@{0}: reset: moving to 7a4b4b6e31a6f4c908ee41dfc29bd132b53bcf68
f6f79b0 HEAD@{1}: commit: aaa
$git reset --hard HEAD@{1}
```

reflogで行動履歴が出るので、戻りたい所を指定します。

## 意気揚々とマージしようとしたら、コンフリクト嵐、いったんマージをなかったコトにしたい

よっしゃ他のブランチとマージしたるぜ！とmergeなりrebaseなりしたら、コンフリクトの嵐…。いったん見なかったことにして元に戻したい。そんな時あると思います。

```bash
$git reset --hard ORIG_HEAD
```

まぁ、どっかで綺麗にマージしなきゃいけないんですけどね。

## リモートブランチからmaster以外のブランチをローカルに新しくもってきたい

リモートリポジトリ上で master / develop / hoge.. など色々なブランチで運用している場合、新規で入ってきたエンジニアはcloneしてくるだけでは、通常masterしか存在してないのです。そこで、開発用developブランチに切り替えたいという時は、こんな感じで。

```bash
$git fetch
$git branch -a
* master
  remotes/origin/develop
  remotes/origin/master
$git checkout -b develop origin/develop
```

origin/develop（リモートブランチ）をdevelop（ローカルブランチ）として作って、ついでにチェックアウトするよって意味です。

## リモートブランチを追加する

リモートブランチを後からadd（追加）/ rm（削除）するのは簡単。

```bash
$git remote add origin git@example.com:git_test
$git remote -v
```

## pushしようとしたら、リモートリポジトリが先行していて怒られた

```bash
$git push
To git@exmaple.com:git_test
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@exmaple.com:git_test'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
hint: before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

チーム開発しているとよくある光景ですね。俺の更新ファイルがアップできん！みたいなの。こんな時は２つの回避方法があります。一つは、pushする前にpullする。お手軽です。よほどのコンフリクトがなければ、自動マージしてくれます。ただし、マージコミットが発生して後から見返すと、マージコミットだらけて分かりにくくなります。

```bash
$git pull --rebase
$git push
```

まぁ、安易にpullする前に、fetchしてどんな更新が先行しているのか確認するのがベストです。



