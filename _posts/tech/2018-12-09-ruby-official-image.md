---
title:  "I was supprised Ruby official image is large."
date:   2018-12-09 23:00:00 +0900
category: tech
tags: docker ruby
layout: post
---

rubyのdockerイメージを落とした時SIZEが869MBでびっくりしました。

ほとんど1GBじゃないか！という。

1GBだからといって何だというわけではないのですが、単純にびっくりしました。

びっくりついでに、PHPを落としてきてみたら367MB。

ミドルウェアが入ると結構容量でかくなるんですね。

アプリケーションが入ると更にでかくなると考えると、
イメージは予めpullできるようにしておかないと起動時間が長くなりそうだなと思いました。

都度pullになるシステムはDLの時間は注意ですね。

```
$ docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
ruby                  2.5                 bdbc506da19b        3 weeks ago         869MB
php                   7.2-cli             8473cbe51b22        3 weeks ago         367MB
```
