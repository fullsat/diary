---
title: jekyllのdistをserveとbuildで同じにすべきじゃなかった
date: '2019-11-28 14:56:31'
category: tech
tags: jekyll
layout: post
---

## 何があったのか

```
firebase deploy
```

したときdeployに失敗した。
deploy時を観察してるとなぜか1ファイルだけアップロードできない。

```
grep [task id] .firebase/xxx.cache
```

すると `feed.xml` がアップロードできていない模様

## 原因

[これ](https://github.com/firebase/firebase-tools/issues/1117#issuecomment-468876073)

どうやらfeed.xmlにlocalhostが含まれてしまっているためにエラーになっている模様。

でも、そもそもそうならないために

```
jekyll build
```

してるのになぜなのか？

## 根本原因

```
jekyll serve
```

と

```
jekyll build
```

を同時に実行していた。

これが同じ場所にファイルを作るので、buildしたら定期的にserveのほうで再度buildしてしまう。

serveでビルドされたものをデプロイしようとしてしまっていた。

serveはもちろんローカルホストのリンクになるためエラーとなっていたわけです。

## どうすればよかったか

github actionsやcircle ciでデプロイする。

ナウいやり方ならこれだろうけど、まぁこれくらいパイプライン組まなくてもいいだろと思ってた。

他の案としては、buildのとき -dオプションをつけてやればいいんだけど、そういう工夫するくらいならパイプライン作ってもいいかなという感覚

ひとまず原因が分かってよかったというお話でした。