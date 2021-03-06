---
title:  "Pragmatic Terraform on AWSを読んだ感想"
date:   2019-04-15 11:00:00 +0900
category: tech
tags: terraform
layout: post
---

技術書典に行ってきました。

行ってきましたが混みすぎててつらかったので、目的の本だけ買いました。

目的の本も１つだけで、Pragmatic Terraform on AWSだけです。

Terraform大好きなのでTerraformの書籍を出すよという人がいて買わなきゃと思って買いました。

その感想です。

## 新しく得られた知識

* AMIの最新版追従

以下でAmazonLinux2の最新版のAMIを参照できるらしいです。

```
data "aws_ami" "recent_amazon_linux_2" {
most_recent = true
owners = ["amazon"]

filter {
  name = "name"
  values = ["amzn2-ami-hvm-2.0.????????-x86_64-gp2"]
}

filter {
  name = "state"
  values = ["available"]
}
```

?は任意の１文字です(おそらく)。
正規表現なのか何なのか分からなかったですが、正しく動いていました。
?を１つ除外すると動きません。

AWS CLIで以下の検索をかけ、そこからタイムスタンプを見て最新のものを取ってくるということですね。

```
aws ec2 describe-images --filters "Name=name,Values=amzn2-ami-hvm-2.0.????????-x86_64-gp2" "Name=state,Values=available" --owners amazon
```

これを利用するなら、たぶんamiのlifecycleは設定しておいたほうが良いはずです。

dataの挙動がどうだったか記憶ありませんが、AMIのバージョンが変わってもリソースの再作成にならないようにするためです。

AMIのバージョンが変わっても再作成の挙動にならずとも、明示的にamiの変更を無視する設定が入っていたほうが意図を残せます。

* HashiCorpがmodulesのディレクトリレイアウトを定めている

[ここのことです](https://www.terraform.io/docs/modules/index.html)

正直なところ内部で利用するだけの環境でのmoduleはここまでファイルを分割する必要ないかなという印象です。

LICENSEとかも不要ですね。

とはいえ、こう作ってね、という指標があるのであれば、機会があった時に参考にしない手はありませんので、参考にします。

機会があれば・・・ね！

## 今後どうするか

まずAMIの最新版追従はやるべきと思いました。(こう書いている間にやりました)

この本は入門編な印象を受けたので、もう少し運用面でのterraformのまとめを自分で書いてみたいなと思いました。

## まとめ

正直なところPragmaticと名がついているのでもう少し運用面に言及していることを期待していました。

特にInfrastructure as Codeとしてのterraformに関して書かれてあると嬉しいなという印象です。

とはいえ、利用していて疑問に思ったことなどがまとめられており、人におすすめしたい書籍です。



