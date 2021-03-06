---
title:  "入門 監視で実践していること,していないこと"
date:   2019-01-27 14:00:00 +0900
category: tech
tags: monitoring infrastructure
layout: post
---

O'REILLYの「入門 監視」を読み終えました。

日本語にするとタイトルが強いですね。

監視に関しては、常時オンコール状態ということもあり、非常に関心が強い分野です。

我流ですが様々なことは検討しているつもりではありました。

事実、読んでいくと、体感で7割は実践しているという感覚がありました。

ただこうして、俯瞰してまとめてくれている本があると自信につながります。

今後のことを考える上で、出来ていないこと、検討したけどしていないこと、自分の環境では不要なこと、これらをまとめておきたいと思います。

※出来ていないことをどうするか考えるほうが重要だと思うので出来ていることに関してはここでは特に考えないようにします。

## 本に書かれていること

あまりに詳しくまとめると、本そのものになってしまうので、ざっくりと。

* 1章
  * 何を監視するかをツールで決めることはやめよう
  * みんなで監視する項目を決めて運用しよう
  * 観測者の影響は気にしないようにしよう
* 2章
  * 監視の要素の基本
  * ユーザ視点で監視をしよう
  * SaaS使おう
  * 継続的に改善しよう
* 3章
  * アラートは意味のあるものを送ろう
  * アラートに対して手順書を用意しておこう
  * オンコールのローテをうまく組もう
* 4章
  * アラートの改善に統計をうまく使おう
* 5章
  * ビジネス視点のメトリクスを知ろう
* 6章
  * ユーザが実際に見ているページのロード時間を監視しよう
  * Javascriptの例外も見れると役に立つよ
  * フロントエンドのパフォーマンステストをしよう
* 7章
  * アプリケーションのリリースを監視しよう
  * 知りたいメトリクスは計測するようアプリケーションの中に埋め込もう
    * サーバレスの監視にはこれ
    * マイクロサービスの分散トレーシングはリクエストにIDを含める
  * 死活監視用のプログラムをアプリケーションに含めよう
* 8章
  * OS関連のメトリクスを知ろう
  * よく使う種類のサーバのメトリクスを知ろう
* 9章
  * SNMPつらいね
  * ストリーミング系のサービスはジッタに注意
* 10章
  * 監査用やコンプライアンス用に監視ツールがあるので活用しよう
  * 侵入検知用にツールが存在するので活用しよう
* 11章
  * 実践しよう


## 出来ていないこと

* みんなで監視する項目を決めて運用しよう
* ビジネス視点のメトリクスを知ろう
* ユーザが実際に見ているページのロード時間を監視しよう
* Javascriptの例外も見れると役に立つよ
* フロントエンドのパフォーマンステストをしよう
* 死活監視用のプログラムをアプリケーションに含めよう

## 検討したがやってないこと

* 侵入検知用にツールが存在するので活用しよう
* アラートに対して手順書を用意しておこう
* 監査用やコンプライアンス用に監視ツールがあるので活用しよう

## 自分の環境では不要なこと

* SNMPつらいね
* ストリーミング系のサービスはジッタに注意

## 今後どうするか

出来ていないことを俯瞰してみると大まかに分けて3つやらなくてはならないと感じました。

* ビジネスの視点を持つこと
* チーム全体で運用を回すこと
* フロントエンドをモニタリングすること

実はこれらは、検討したがやっていないことには含めていませんが、一度どうしようかと考えたところです。

やってないことは、現状は必要ないと組織的に合意されているものですが、これらにはそれが存在していません。

いずれやらなくちゃいけないなと思ってやっていなかったこと、といったほうが正しいかもしれません。

なぜやっていないのかも含めて、今後どうしようかも書いて行こうかと思います。

## ビジネスの視点を持つこと

組織の仕組み上ここの値を追うことにメリットが無いことが、現状やっていない最たる理由です。

アンチパターンのロールによる分割に当てはまるかもしれません。

組織を変えていくのはつらいですが、必要性を理解してもらうためにも、私から変えたほうが良いかもしれないと思いました。

そのためにも、まずは単純なメトリクスをダッシュボードに取り込むところから始めようかと思います。


## チーム全体で運用を回すこと

私の所属している組織では、パフォーマンスの監視はインフラ担当の仕事と思われていることが、現状やっていない理由です。

何度もこの文化を変えようとチャレンジしましたが、実現には至っていません。

私は主にインフラ周りを担当にしているため、アプリケーションの問題を追うことはかなり難しいと感じます。

そもそもアプリケーションのことを知らないのに、ユーザ視点のメトリクスの設計ってできるのかなと思っています。

明確な問題があるにもかかわらず、変更を受け入れてくれません。

この状況で運用を回すと、かなり疲弊します。

文化を変えるためにまだやっていないこととしては、DevOpsのプラクティスですが、調査していないのでこれを調査することから始めようと思います。


## フロントエンドをモニタリングすること

これは組織的に分断されていたことがやっていなかった原因です。

最近チームが統合されたので、ここはすぐにでも変えられるのではないかと思っています。

まずは計測した値を取得するところから初めて、あとは組織としてそのメトリクスを指標として改善に取り組む文化を作っていくことが目標です。


## まとめ

こう、まとめていくと、主に組織的な問題が原因で、出来ていないことがほとんどだとわかりました。

文化を変えていく必要があるなと思いましたが、これが一番大変ですよね・・・

どうしよう・・・


