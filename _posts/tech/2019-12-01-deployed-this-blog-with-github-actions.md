---
title: jekyll を github actionsでdeployする
date: '2019-12-01 00:00:00'
category: tech
tags: jekyll
layout: post
---


タイトルの通り

## 何がやりたかったのか

* ipadやスマホからjekyllを更新したい
* その場合jekyll adminは適切ではない
* githubでdeployできればgithubのcreate newファイルからブログを更新できるという寸法
* そうじゃない場合でもpushすれば良い

## ワークフロー

firebase hostingにて公開している。
ブログの作成にはjekyllを利用している。
したがって、単純に以下ができればよい

* git push
* jekyll build
* firebase deploy

これまでcircle ciを使っていたけど良い機会なのでgithub actionsを利用してみる

### github actions

初めて使ってみた。

だいたいは [GitHubのヘルプ](https://help.github.com/ja/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)
で使い方が理解できるはず。

やってみた系のブログだと正直イマイチ分からない

ということでひとまずdeploy出来たやつはこちらの通り

```yaml
name: Deploy to blog.imasdiary.com

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: jekyll/builder:4.0
      volumes:
        - $PWD:/srv/jekyll
    steps:
    - uses: actions/checkout@v1
    - name: build
      run: |
        chown -R jekyll:jekyll .
        jekyll build --trace
    - name: Deploy to Firebase
      uses: w9jds/firebase-action@master
      with:
        args: deploy --only hosting
      env:
        FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }}
```

### jekyll build

jekyll/builderのイメージを利用しようとした。

ただここがかなり詰まった。

なぜかmkdir .jekyll-cacheがpermission deniedで失敗する。

yamlの `chown -R jekyll:jekyll` がミソ

https://github.com/envygeeks/jekyll-docker/blob/master/repos/jekyll/Dockerfile#L133

jekyllはどうやらjekyllユーザで各種実行されるらしいことがなんとなくわかった。

なので事前にchownしているわけです。

README.mdに描いてあるとおりでJEKYLL_UIDを指定しようとしたけどうまく行かなかったので微妙なやり方になってしまっている


### firebase deploy

github actionsには事前にワークフローを定義するライブラリのようなものがある。

それでfirebaseのdeployをするものがないか探したらあった。

jekyllも探したがgithub pagesに対して行うものしか無かったためjekyllに関してはコンテナを利用している。

## まとめ

cacheを利用していないためデプロイに少々時間がかかるのが微妙。

というかキャッシュが利用できるか分からない。

そして相変わらず画像問題が解決しない。

どうしたものか...


