---
title:  "Terraformの自動適用に向けてパラメータストアを試してみた"
date:   2019-04-16 16:00:00 +0900
category: tech
tags: terraform
layout: post
---

みんなterraformの手動運用にこなれてきたのでそろそろ自動化しよう

そのためにも変数をどこで定義するかはちゃんとしよう

ということでパラメータストア試してみました。

## モチベーション

terraformでCI/CD組むときにコンポーネントごとに変数が異なっていると自動化がしにくいなと想像しています。

```
terraform plan -var hogehoge
terraform plan -var-file config.tfvars
```

こんな感じでやってると、config.tfvarsはs3から落としてきて...とか、CI/CDの環境変数から与えてやって...とかやらなくちゃいけなくなりそうという予想です。

こうなるとCI/CDの構築もterraformの運用に依存してしまって、CI/CDのコードの再利用も難しくなるはず。

しかも変数が定義されていないコードをcloneしてapplyしようとして、変数指定しろと言われても、結局何使えばいいの？ってなりますしね。

要するに変数の定義はterraformのコードの中で完結させるべきじゃね？と思った次第です。

## 何を使うか

terraformのコードの中で動的に参照する変数を変えればいいので、真っ先に思い浮かんだのは以下３つ

* vault
* aws secrets manager
* aws system manager の parameter store

vaultはこのためだけにインスタンスを建てなくちゃいけなくなるので「絶対嫌でござる」のお気持ちでした。

となるとaws secrets managerとparameter store

## どっちも試してみました

githubのapikeyを定義する場合の記述量を見てみました


* secrets manager


```
+data "aws_secretsmanager_secret" "ghe_apikey" {
+  name = "ghe/apikey"
+}
+
+data "aws_secretsmanager_secret_version" "ghe_apikey" {
+  secret_id = "${data.aws_secretsmanager_secret.ghe_apikey.id}"
+}
+
+data "external" "ghe_apikey_json" {
+  program = ["echo", "${data.aws_secretsmanager_secret_version.ghe_apikey.secret_string}"]
+}


+  token = "${data.external.ghe_apikey_json.result["ghe_apikey"]}"
```

* parameter store

```
+data "aws_ssm_parameter" "ghe_apikey" {
+  name = "ghe_apikey"
+}

+  token = "${data.aws_ssm_parameter.ghe_apikey.value}"
```

圧倒的にparameter storeのほうが分かりやすい


## まとめ

軽く調べた感じだと、どっちも暗号化して保存できるので、であれば記述量が少ないparameter storeかなと。

後から見たときに意図が分かるように、という観点から考えると、依存をたどる必要があるsecrets managerは嫌ですね。

(もしかしたら、もっとやり方があるのかもしれませんが、ぐぐって一番上に出てきたやり方がこれでした。)


