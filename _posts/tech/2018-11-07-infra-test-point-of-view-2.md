---
title:  "Infrastracture Test Point Of View 2"
date:   2018-11-07 17:00:00 +0900
category: tech
tags: IasC テスト
layout: post
---

[Infrastracture Test Point Of View]({{ site.baseurl }}/posts/infra-test-point-of-view/)

引き続き、IaCのテストを考えていきたいと思います。

今回は

* IaCのテストでは何をするのか
* どのタイミングでそれらのテストをすべきなのか

これらの全体像の把握をしたいと思います。

全体像とは、言い換えるとシステム構築時に確認することリストです。

全体像が見えないと、結合テスト、単体テストで何を確認すれば十分と言えるのか、が見えてこないからです。
そもそも何を確認したいんだっけ?ということになってしまっているので、自分の頭の中をダンプしておこうというモチベーションです。

## 全体像の把握

そもそもシステム構築とは何なのかを演繹的にまとめると

```
1. "システム構築"は"要求"を実現すること
2. "要求"を100%実現しているものが、100%の"品質"を保っている
3. "テスト"とは"要求"がその通り実現できているかを確認する作業
4. よって"テスト"とは"品質"を証明する手段
```

飛躍して話すと、テストで品質を証明することができれば、システム構築が出来たと言えます。

ここから分かることは、品質が何なのかが分かれば、何をテストすべきなのかが分かるということです。

品質に関しては、頭の良い方たちがソフトウェア品質をISOでまとめてくれています。

これがおそらく自分の知りたいIaCのテストの全体像になります。

### 品質特性

以下が頭のいい人たちが考えた品質のリストになります。

* ソフトウェアの品質
  * 機能性
    * 合目的性
    * 正確性
    * 接続性
    * 整合性
    * セキュリティ
  * 信頼性
    * 成熟性
    * 障害許容性
    * 回復性
  * 使用性
    * 理解性
    * 修得性
    * 操作性
  * 効率
    * 実行効率性
    * 資源効率性
  * 保守性
    * 解析性
    * 変更作業性
    * 安定性
    * 試験性
  * 可搬性
    * 環境適応性
    * 移植作業性

これらを見て分かるとおり、ISOで定義してくれていることは、こういう品質がありますよ、
各プロジェクトごとにそれぞれ考えてくださいね、という観点しか与えてくれていません。

プロジェクトごとに品質が変わるので観点を示すのが限界ということは理解に容易いです。
ただ、今回はIaCのテスト(IaCの品質)という制約があるので、もう少し具体的にできないかなと思っています。

ISOの特性を具体的にすると言うよりは、自分が普段注意している観点(テストしていること = 証明している品質)が、
この品質特性にどう合致しているかを確認し、
それが全体像として私のユースケースにおいて妥当か確認します。

それを以て「IaCのテスト完全に理解したわー」と言いたいと思います。

※
私はISOのように高度に汎化できるほどのユースケースを経験しているわけではありません。
そのため具体的にしたところで、他の人のユースケースに当てはまるわけではないし、
場所が変われば今回書いたことの内容も変わっていくと思っています。


### 自分が行っているテストの観点

```
構築しなくてはいけないもの
= 自分が構築時に気を付けていること
= テストで確認しないといけないこと
```

というわけで、自分がインフラを構築するときに確認しているポイントをリストします。

KKD(経験と勘と度胸)でやっていることをリストにしてみると割といろいろ考えていたことに驚いています。

<details>
<summary>機能を実現するために必要なリソースはそろっているか</summary>

インスタンスが1台欲しいといった単純な場合でも、意外といろいろなリソースに依存しています。
webアクセスするためにALB/Route53/IAM/VPC...
忘れ物がないかどうかを確認します。
そもそもその構成でやりたいことが実現できるかも確かめます。

</details>

<details>
<summary>管理しなければならないリソースが多すぎないか</summary>

多すぎると管理しきれなくなるので減らせる手段がないか確認します。

</details>

<details>
<summary>同僚が知っているツール/ミドルウェア/インフラリソースか</summary>

自分が使うだけなら問題ありませんが、他の人にも保守してもらうなら、使う対象の理解を深めるフェーズも必要になってきます。
誰も知らない、誰も保守していないようなツールを使うのはリスキーなので、そういうことがないかも確認します。

</details>

<details>
<summary>必要なミドルウェアがインストールされているか</summary>

素のサーバにミドルウェアをインストールしたり、
Dockerfileを作成したりします。
その時必要なミドルウェアがそろっていて、必要な設定がされているかを確認します。
カーネルチューニングやタイムゾーンの設定、そういうのもします。

</details>

<details>
<summary> リソースを作成した意図を後から追えるようにする</summary>

コンソールから構築してると、いつ、なぜ、だれが、どうやって、といったことが分からなくなります。
terraform/ansible/chef/cloudformation/shellscriptなんでもいいのでこれらをバージョン管理します。
ただ、とりあえずマネコンから作って、ということをやってると、管理されない状態になっちゃったりします(おそらく世のスーパーエンジニアたちはそんなことしない)。

</details>

<details>
<summary> パフォーマンスは問題ないか</summary>

Cloudであまり良くない傾向が生まれやすいなと思うのは、アルゴリズムの改善で性能アップを図るのではなく、
スケールアウト・スケールインで解決しようとすること。
なのでパフォーマンスを計測しなくてもお金さえかければ何とかなることは結構ある気がします。
ただ、少なくとも性能の限界がどこまでなのか、ちゃんとスケールするのか、といったことは確認しておきたいポイントです。

また個々のリソースをベンチマークテストをしておくと、しないにしてもインターネットに転がっているベンチマークの情報を集めておくと、
リソースのグラフを見ながら、今後タイプを下げたり、上げたり、台数減らしたり、といった戦略が立てられます。

なので、必要そうな部分のパフォーマンスは計測しておきます。
パフォーマンスは継続して計測できることが理想だと思います。

</details>

<details>
<summary> 継続的に動かしても問題ないか</summary>

緩やかに死を迎えることがあります。
メモリの増加やディスクがあふれたりとかですね。
パフォーマンスが問題なくても、それが持続できるかどうかは別問題なので、
負荷テストやるときはできれば数日単位で負荷をかけると、リソースの動きが確認できます。

</details>

<details>
<summary> 再起動しても問題ないか</summary>

再起動しても問題ない設定になっているか確認します。
再起動したら、このプロセス立ち上がってなかった！という悲惨な状態にならないようにしたいです。

</details>

<details>
<summary> 設定変更がしやすい構成になっているか</summary>

設定の変更を手動でやっていると、変更差分が出てくる可能性が上がったり、
間違って落としちゃったということも出てくるのでダメです(ダメなことをよくやってます)。

設定変更は自動化できているかどうか、
そもそも構成が自動化しやすい構成になっているかを確認します。

</details>

<details>
<summary> 設定変更時にダウンタイムは発生しないか</summary>

いったんサービスを停止しなくてはならないのか、そうではないのか。
停止しなければいけないのならば、停止時に連絡する先や、メンテナンス画面の準備など、それらができているかを確認します。

</details>

<details>
<summary> 冗長構成が担保されているか</summary>

多重化は基本で、SRE本なんかでもN+2の構成にしろと話してますね。
どれか一つを落としてもサービスが継続されるかなどを確認します。

</details>

<details>
<summary> バックアップは取れているか</summary>

いつ、どこに何世代分置くのか。
どうやってバックアップから復元するのか。
これらを決めて、正しく動いているのかを確認します。

</details>

<details>
<summary> ログは取れているか</summary>

アプリケーションのログ、ミドルウェアのログ、OSのログ、いろんなログがありますが、なるべくとっておいたほうがいろいろ役に立ちます。
何を保存しておくか、いつまで保存しておくか決めて設定します。
ちゃんとその通りに動いているか確認します。

</details>

<details>
<summary> 監視は設定されているか</summary>

まず、まずい状態を定義して、死活監視、リソース監視を設定します。
設定に漏れがないか確認します。

</details>

<details>
<summary> リソースやログは可視化できているか</summary>

リソースはグラフにしてると良いです。
システム毎にダッシュボードなんかがあると便利ですね。
それらが存在しているかを確認します。

</details>

<details>
<summary> 復旧手順が用意されているか</summary>

インスタンスが勝手に復旧してくれればいいのですが、裏で物理的なものが動いている以上、
このインスタンスがぶっ壊れたらどうするか、ということは考えておいたほうが良いです。
考えておいたほうが良いので考えているか確認します。

できればシステムが勝手に復旧するようにしておきます。
復旧までの時間も決めた時間内に収まるようにしておきます。

</details>

<details>
<summary> 適切な権限を持つ人が適切なリソースにアクセスできるようになっているか</summary>

知らない人がsshログインできるようになっているとかしてるとやばいですね。
このIPからしかアクセスされたくないのにインターネットからアクセスできるようになっていたとかもやばいですね。
個人情報に誰でもアクセスできるようになっていたらやばいですね。
そういうことがないかを確認します。

</details>

<details>
<summary> バージョンは最新か</summary>

バージョンを塩漬けにしないようにするという話です。
バージョンを上げやすいような構成にしているのか、
そうじゃなくても定期的にバージョンが上がる仕組みになっているか。
保守しなくなったら終わりなのでどうしていくのか確認します。

</details>

<details>
<summary> 開発環境と本番環境をどこまで似せるか</summary>

開発環境と本番環境の相違点がないことが理想的ですが、相違点がどうしても出てきてしまう場合もあります。
その時どこが違っているのか、本質的にそれで問題ないかは事前に確認しておきます。

</details>

<details>
<summary> 使っている技術は最新か</summary>

最新が良いわけじゃありませんが、すたれているものを使うと、情報がなくて、困ったときに困ります。
なので技術的に流行っているもので、それが要件に合致しているのが最高です。

</details>

<details>
<summary>引っ越しできるか</summary>

個々のインスタンスは簡単に移せるとしても、クラウドベンダごと移せるかということを考えます。
ただ半年に一回くらい考えるくらいです。あんまり確認してません。
AWSがつぶれるより先につぶれる会社がどれだけいるかをよく想像します。

</details>

<details>
<summary>複製しやすいか</summary>

今の時代オートスケールだったり、マルチプラットフォームだったりは当たり前の時代になりました。
(なりましたと言って昔の時代をあんまり知らないけど)
状態を持つサーバは複製がしにくいです。
イミュータブルに作るってやつです。
可能な限りイミュータブルに作られているか確認します。

</details>


### 品質特性に当てはめてみる

|特性  |観点|
|:-----|:---|
|機能性| 機能を実現するために必要なリソースはそろっているか <br> 必要なミドルウェアがインストールされているか <br> 適切な権限を持つ人が適切なリソースにアクセスできるようになっているか |
|信頼性| 再起動しても問題ないか <br> 設定変更時にダウンタイムは発生しないか <br> 冗長構成が担保されているか <br> 復旧手順が用意されているか |
|使用性| 管理しなければならないリソースが多すぎないか <br> 同僚が知っているツール/ミドルウェア/インフラリソースか <br> 使っている技術は最新か |
|効率| パフォーマンスは問題ないか <br> 継続的に動かしても問題ないか |
|保守性| リソースを作成した意図を後から追えるようにする <br> 設定変更がしやすい構成になっているか <br> ログは取れているか <br> 監視は設定されているか <br> リソースやログは可視化できているか <br> バージョンは最新か <br> 開発環境と本番環境をどこまで似せるか |
|可搬性| バックアップは取れているか <br> 引っ越しできるか <br> 複製しやすいか|

これで漏れがないかというとそんなこと誰にも分かりません(ありそう)。

ただ、現在の自分の頭にある観点はこれで、特性的にも全部埋まってるので、これが私のユースケースにおける全体像と言いきります。

## どのタイミングでテストするのが適切か

やることの全体像がはっきりしたのは良いのですが、そもそも単体テストでは何やるの、
どのタイミングで何をテストするの?といった疑問は解消されていません。

そのため、ここまでに挙げたリストをどのタイミングで実施できるかを確認して行きたいと思います。

### 開発プロセスにおけるテストの関係

開発プロセスの基本はV字プロセスですね。

どんな開発手法でもこのサイクルの頻度が変わるだけで、
やることは全く変わりませんのでこれをベースに考えます。

```
要件定義 ===================> システムテスト
    基本設計 ===========> 結合テスト
        詳細設計 ===> 単体テスト
                 実装
```

そして、要件定義/設計の段階で不具合をつぶすことが、プロジェクトを炎上させない最良の選択であることは、経験されていることかと思います。

このサイクルのなるべく早い段階で不具合を発見できれば良いのですから、
テストもなるべく早い段階ですることがベストと言えます。

### どのタイミングでテストが可能かを考える

ということで、単体、結合、システムテストのうち、どのタイミングが最速で確認できるポイントなのかをまとめます。

なんとなくの感覚で分類してみました。

* 単体テスト
  * 再起動しても問題ないか
  * 必要なミドルウェアがインストールされているか
  * バージョンは最新か
  * リソースを作成した意図を後から追えるようにする

* 結合テスト
  * 機能を実現するために必要なリソースはそろっているか
  * 適切な権限を持つ人が適切なリソースにアクセスできるようになっているか
  * 設定変更時にダウンタイムは発生しないか
  * 冗長構成が担保されているか
  * 管理しなければならないリソースが多すぎないか
  * 同僚が知っているツール/ミドルウェア/インフラリソースか
  * 使っている技術は最新か
  * 設定変更がしやすい構成になっているか
  * 開発環境と本番環境をどこまで似せるか
  * 監視は設定されているか
  * バックアップは取れているか
  * リソースやログは可視化できているか
  * ログは取れているか
  * 複製しやすいか

* システムテスト
  * 復旧手順が用意されているか
  * パフォーマンスは問題ないか
  * 継続的に動かしても問題ないか
  * 引っ越しできるか

この分類はただの分類で、各項目要件によっては前後にずらせるかもしれません。

ただこうして眺めてみると、どれが自動でテストが出来て、どれが出来ないかが見えてきます。
また、単体テストのユースケースや、結合テストのユースケースのイメージが湧いてきました。

## まとめ

単体テストでテストしたい観点が見えてきました。
前回見たvaultの例では確かに自分が確認している観点が含まれており、分類も自分の認識とずれがないことが分かりました。

ここでまた疑問が出てきました。
Ansibleで作られたリソースはServerspecで確認しなくても良い、という議論があったと思います。
Ansibleで確実に担保できるからテストしなくて良いという話になるのだと思いますが、
Ansibleはどこを保証してくれているのかがいまいち整理できていません。
今回リストして分類したことを軸にこのあたりを考察していければいいと思います。

また、結合テストでの確認ポイントが多いことから、
システム構築の設計でこのあたりを考慮しておかないと、
後でつらい目にあうこともわかりました。
テストでダメなことを発見するのではなく、
あらかじめ設計に盛り込むためにレビュー観点として盛り込んでおきたいです。

最後に言っておきたいのは、これらのレビュー観点があると安心して、これ以外を見ないということが出てきたりします。
自分で考えないというやつです。
これからも、まずすべてのレビューは自分で何が必要かを考えてからレビューを行い、
最終的にこのリストで抜けがないか確認していく、という方法をとっていきたいです。


