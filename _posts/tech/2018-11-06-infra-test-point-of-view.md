---
title:  "Infrastracture Test Point Of View"
date:   2018-11-06 10:20:00 +0900
category: tech
tags: IasC テスト
layout: post
---

最近Infrastracture as Codeにおいて、
単体、結合、システムテストそれぞれにおいて何をテストすべきか悩んでいます。

現状は個々のプロジェクトでここさえ確認しておけばいいだろというのはあり、それぞれのプロジェクトに合わせてテストを作っています。
ただAnsibleで構築してるのに、このユーザは存在するか？みたいなテストをするのは意味ないとも感じています(実際にはこんなのテストしていないけど)。

いったいどういうときに何をテストするのかが分からないのです。

要するによくわからずIaCのテストを行っているのです。
なので一度自分なりにまとめてみたいと思います。

IaCのテストを他の人はどうやってるんだろうというのが分からなかったので、
ひとまず他の人の事例を調べてみます。

## 知りたいこと

* インフラのユニットテストでは何を確認しているのか
  * Ansibleをはじめとした構成管理ツールを用いている場合にテストする意味があるのか
* 複数のリソースが存在することで初めてテストしやすくなるようなもののテストはどうしているのか
  * 例えばnginxのproxy_passのテストとか
    * モックサーバを作成してテストしているのか(単体テストか)
    * 正規のプロキシ先を作成してテストしているのか(結合テストか)
  * 例えばserver-agent型のミドルウェアとか
    * serverに正しくデータが送られているかはserver自体を構築しないと確認できないのではないか

## 確認したものとそのエッセンス

[読んだ記事](https://qiita.com/yoshiya64/items/22792cbcf6e8eb0010a5)

Unitテストとは

> インフラエンジニアがソフトウェアに対して行った設定が正しく設定されているか、正しく動作しているかを確かめるテストです。

としていて、例としてpostfixのmain.cfに対してmydomain がgmail.comに設定されているかgrepで確認しています。

> ちなみにデフォルト値の確認などは行いません。

設定変更した箇所のみテストをするという話ですね。

Unitテストの観点を挙げています。

```
* パラメータ確認
* プロセス確認
* サービス起動確認
* ポート確認
* 起動停止確認、二重停止二重起動確認
* ログ確認
* エラーログ確認
* PP機能確認(Postfixならばメールが送れることを確認)
```

---

[読んだ記事](https://thinkit.co.jp/article/12056)

```
* 作業ログの取得
* 構築資材の確認
* 設定差分の確認
```

インフラエンジニアが確認するものとして、共通認識の部分とのこと。
構築資材の確認は、チェックサムで正しいファイルが利用されているかを確認しようということらしいです。
要するに作業ログ、正しいファイルを利用しているか、設定差分が意図したものか、をテストでは確認しようという話です。

```
* 単体テスト
OSやミドルウェアの設定、動作を単体でテストします。例えば、”スマートフォンのロック解除パスワードがxxxに設定されている””電源ボタンを押したら起動する”といった動作を評価すると言えばイメージしやすいかもしれません。
* 結合テスト
複数の機能を組み合わせた連携が可能かをテストします。再度スマートフォンの例で言うと“パソコンに入っている音楽アプリからスマートフォンへ楽曲を転送し、スマートフォンの音楽アプリでその楽曲が再生できるか”という一連の流れを評価します。実際のインフラの結合試験はプロジェクトによって内容が大きく異なりますが、非機能要件に従って設計された基本設計内容を確認するのが大きな目的です。
* 総合テスト
別名システムテストとも呼ばれ、本番環境で正常に動作するかを確認します。インフラエンジニアが関連する試験項目としてはシステムの性能を計る性能試験、実運用時に想定される負荷に耐えられるか検証する負荷試験などが挙げられます。
```

各フェーズで何をするのか書いていますが、
例がアプリケーションの話なので、インフラのテストとしては具体的なイメージはつきませんでした。

結合テストは、各プロジェクトごとに大きく異なり、非機能要件に従って設計された基本設計の内容を確認する、とあります。
これはV字モデルの対応の話ですね。
基本設計は各プロジェクトごとに異なる、という主張かと思います。

---


[読んだ記事](https://hiroakis.com/blog/2013/12/24/serverspec-%E3%82%A4%E3%83%B3%E3%83%95%E3%83%A9%E5%B1%A4%E3%81%AE%E3%83%86%E3%82%B9%E3%83%88%E9%A0%85%E7%9B%AE%E3%82%92%E8%80%83%E3%81%88%E3%82%8B/)

> 2. テスト項目

2に作ったテスト項目一覧が出ています。
粒度はまちまちなようですが、実際に動かすことでわかることと、動かさずともわかること、が書かれていそうです。
動かすことでわかることとは、ポートがLISTENの状態になるかということ。
動かさずともわかることは、インストールされているかどうかということ。
確認したいミドルウェアのプロセスがあるかないかの違いです。

```
4. やってみた結果

おもしろいミスがたくさん見つかった
```

この話から、設定変更の自動化やパイプライン化は行っていなさそうなことが分かります。
ミスが出る = 人手を介しているという認識からです。

---

[読んだ記事](http://labs.opentone.co.jp/wp-content/uploads/2010/02/8a7a1b428edc1ed170f8556838542f41.pdf)

google検索したらテスト観点表が出てきました。

インフラに関わりそうなことは、

* これらのテストができるようなインフラ設備を用意すること
* 性能観点

あたりでしょうか。
これを汎化すると、前者が機能要件、後者が非機能要件になると思います。

---

[読んだ記事](https://blog.fieldnotes.jp/entry/infrastructure-test-by-geb)

```
* 設定ファイルの項目の確認
* アプリケーションのプロセスが起動しているかの確認
* ヘルスチェックのURLにアクセスして行う疎通の確認
* サーバーおよびネットワーク機器間の疎通の確認
という観点まではServerspec等を用いたテストが可能ですが、それより上位レベルの観点はデプロイしたアプリケーション上の動作で確認することになります。
```

これは要するに単体テストでは、ここまでのテストが可能で、それ以外は他のサーバと組み合わせないと確認できないということですね。
[こちら](https://hiroakis.com/blog/2013/12/24/serverspec-%E3%82%A4%E3%83%B3%E3%83%95%E3%83%A9%E5%B1%A4%E3%81%AE%E3%83%86%E3%82%B9%E3%83%88%E9%A0%85%E7%9B%AE%E3%82%92%E8%80%83%E3%81%88%E3%82%8B/)で確認していることと一致しています。

自動構築とテストとの関係は書いてありませんが、おそらく話の内容やDevOpsという話から自動化されているものと考えます。

---

[読んだ記事1](https://qiita.com/bbrfkr/items/192bb245c597a78c2547)
[読んだ記事2](https://qiita.com/bbrfkr/items/26fd160541e9729ec0cf)

> なぜならば、勘違いしがちなのですが、テスト駆動開発で行われるテストは品質保証のためのテストではないからです。

ISOで言われている品質特性の意味においては非機能要件も機能要件も品質に分類されると思っていましたが、文脈上品質とは非機能要件のことを指しているようです。
意図としては、非機能要件のテストのためのテスト駆動開発ではないということと推察できます。
要するに、開発フローにおけるテストでは、非機能要件のテストは本来の目的ではないということです。

> テスト駆動開発のやり方

Serverspecでテストコードを書いてから、Ansibleのコードを書くというフローが示されています。

> Ansible・Serverspecを用いたテスト駆動開発の実践

例ではServerspecでapacheをインストールしているかテストコードを書き、Ansibleでapacheをインストールするコードを書いています。

---

[読んだ記事](https://opencredo.com/self-testing-infrastructure-as-code/)

> From a DevOps perspective, unit testing could be applied to testing a single machine image, custom scripts or whether a Docker image has been constructed correctly.

ユニットテストは１つのマシンイメージ,カスタムスクリプト、Dockerイメージに適用される、としています。

> Unit
> Example

単体テストの例では以下のように動いているようです。

```
1: packerを起動する
2: ansibleのコードを動かしvaultをインストールする
3: インストールが完了したらAMIを作成する
4: terraform applyすると、インスタンスが作成される
5: インスタンス作成後にserverspecのコードを実行する
```

https://github.com/opencredo/self-testing-vault/blob/master/packer/tests/spec/vault_spec.rb

このserverspecのコードではvaultがrunningの状態か、vaultがインストールされているかなどを見ています

> Integration
> Example

https://github.com/opencredo/self-testing-vault/blob/master/tests/spec/terraform_helper_spec.rb
結合テストの例では以下のように動いているようです。

```
1: serverspecで以下のテストケースを実行する
1-1: テストケース1
1-1-1: terraformで単体テストのところで作成したイメージからインスタンスを作成する
1-1-2: 構築したインスタンスのパラメータを元にvaultヘルパーを初期化し値を挿入
1-1-3: vaultから挿入した値を引けるか確認する
1-2: テストケース2
1-2-1: 本番環境ライクなインスタンスをterraformで作成する
1-2-2: 構築したインスタンスのパラメータを元にvaultヘルパーを初期化し値を挿入
1-2-3: vaultから挿入した値を引けるか確認する
1-2-4: 5秒毎に挿入した値が引けるか確認するスレッドを起動
1-2-5: 本番環境ライクなインスタンスからchangedの状態になるterraformのコードを実行する
1-2-6: この間にvaultの値が引けてない場合があったらテスト失敗
1-2-7: ELBに紐づいているインスタンスを１台１台落としていく
1-2-8: AutoScaleで新規に立ち上がっていく
1-2-9: このインスタンスを入れ替えている最中にvaultの値が引けていない場合があったらテスト失敗
```

テストケース2では入れ替え最中にダウンタイムが発生しないことを確認するとありますが、
本来の目的は運用時にインスタンスを入れ替えたりすることの想定だと思います。

---

[読んだ記事](https://thinkit.co.jp/article/10347)

Ansibleでどのような場合にテストをするべきかが記載されています。

> 一連のtask単位をテストする

例えばjenkinsが正常に起動することを一つのタスクとすると、jenkinsプロセスが動いていることを指しているようです

> 不安なtaskをテストする

何かしらこういう場合どうなんだろうというところをテストしろという話らしいです。
一般化はできなさそうです。

> シェルスクリプトをテストする

これはおそらく冪等性がないものを指していると思われます。

> ユーザの期待する結果をテストする

結合テストやシステムテストで行うことを指しています。

---

[読んだ記事](https://dev.classmethod.jp/server-side/ansible/testing/)

> Ansible自体が宣言的に状態を定義し、その状態となっていることを保証しているので、あらためてテストを実施しなくても良い

宣言的に状態を定義しているものであればテストは実施しなくてよいとしています。
ただし、それでも実施すべきものは以下の通りと言っています。

> 不安をテストする 

としていて、具体的な不安の例として以下の項目を挙げています

```
* 納品する環境が期待通りの環境か？
* 共有Roleが正しく動作するか？
* 複数の環境で動作するか？
* OSアップデート時に動作するか？
* 複雑な手順の結果、期待通りの状態となるか？
```

納品する環境が期待通りの環境か、というのは動いているプロセスから、システムテストを含めての話のように感じます。
共有Roleのテストで何をテストしているのかがよくわかりませんでした。
複雑な手順の結果、というのは冪統性のないもののことを指していると思います。

---

[読んだ記事](https://qiita.com/katsuhisa__/items/15236253e72a8de74b00)

ansibleのroleに対してserverspecでテストするという流れを、一つのリポジトリで行うためのツールの使い方が書かれています。

> そもそもAnsible 使ってるんだったらServerspec いらないんじゃね？派

開発プロセスに幅が持たせられて便利という意見は、具体例がないためどういう意味か分かりませんでした。


## 分かったこと

* インフラのユニットテストでは何を確認しているのか
  * Ansibleをはじめとした構成管理ツールを用いている場合にテストする意味があるのか

不安に思うことをテストすることは共通しているようです。

具体例は以下のようなものがありました。

```
* 設定ファイルの項目の確認
* アプリケーションのプロセスが起動しているかの確認
* ヘルスチェックのURLにアクセスして行う疎通の確認
* サーバーおよびネットワーク機器間の疎通の確認
```

vaultの例を見てみてもアプリケーションのプロセスが起動しているか、という点に該当しそうです。

Ansibleを使った場合は、Ansibleで保障されているものは特に検査しなくても良いが、不安はテストするというところに落ち着くようです。

vaultの例のユニットテストでは主に、vaultが指定のバージョンでインストールされているか、vaultプロセスが動いているかを確認しています。
vaultが指定のバージョンでインストールされているかは、別のバージョンでは動かない可能性があるということが読み取れます。
vaultプロセスが動いているかは、イメージからインスタンスを作成したときにvaultプロセスが起動している状態になっているかをテストしていると取れます。
インストールされているかどうか、といったことはAnsibleでインストールしていることは間違いないので、確認はしていないようです。

* 複数のリソースが存在することで初めてテストしやすくなるようなもののテストはどうしているのか
  * 例えばnginxのproxy_passのテストとか
    * モックサーバを作成してテストしているのか(単体テストか)
    * 正規のプロキシ先を作成してテストしているのか(結合テストか)
  * 例えばserver-agent型のミドルウェアとか
    * serverに正しくデータが送られているかはserver自体を構築しないと確認できないのではないか

これはvaultの例を見ると結合テスト時にテストしているように感じます。
また、ユニットテストで確認していることの具体例を見る限り、単体テストで行うのは難しそうで、実際やるとなると高コストになると思われます。

## 不足しているもの

これらを調べていて、さらに調べたいことが出てきました。

* 具体例を調べられていないためあまりイメージがつかめていない
* 個々の事例はわかるが全体的に何を観点にしていけばいいのか一覧できていない

次はGitHubで具体例を追ってみようと思います。


---

その他見たもの
https://www.ospn.jp/osc2017-do/pdf/OSC2017_do_TIS.pdf
http://staycreative.jp/2015/09/infrastructure-as-code-demerit/
https://www.mhlw.go.jp/sinsei/chotatu/chotatu/kankeibunsho/20130325-1/dl/02.pdf



