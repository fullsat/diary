---
title:  "Raspberry Pi上のdockerの罠"
date:   2019-03-31 23:00:00 +0900
category: tech
tags: raspberrypi docker
layout: post
---

Raspberry Pi 上にdockerインストールしようとして、２箇所詰まったのでまとめ

## 最新版を入れようとするとセグメンテーション違反

```
# curl -fsSL https://get.docker.com -o get-docker.sh
# sh get-docker.sh
```

これで入れようとするとdockerdするだけで落ちます。

[ここ](https://github.com/moby/moby/issues/38175) によると、バージョンが18.09.0だと落ちるようです。

対処法も書いてあって、バージョンによるから下げろという話らしいです。

```
# apt install docker-ce=18.06.2~ce~3-0~raspbian
```

## Raspberry PiのCPUアーキテクチャはarm

dockerデーモンが動いてるので

```
# docker run -d -P --name ngx nginx:latest
# docker run -it --rm centos:centos7 bash
```

で、コンテナを作って、コンテナ内部のネットワークテストをしようとしました。
結果、docker ps に出てこない。centosの方はログインせずに終了してしまう。

？？？？？？？という感じでしたが、ログもエラー出てる気配はない。

で、表題というわけです。

```
# docker run -d -P --name ngx arm32v6/nginx
# docker run --rm -it arm32v6/bash bash
```

こっちでいけました。

よくよく考えると、だいたいのdockerイメージはx86_64上でビルドされているため、そのイメージを実行したらCPUの命令が実行できなくて落ちますよね。

## まとめ

頭の中にない事象は最初から対処は出来ませんね。

この辺が経験というところでしょうか。

時間はかなり失いましたが、いい経験がつめたような気がします。(というかそう思わなければやってられない)


