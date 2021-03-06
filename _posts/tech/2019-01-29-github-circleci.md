---
title:  "とあるDevOpsのセミナーを受けて"
date:   2019-01-29 18:00:00 +0900
category: tech
tags: github circleci cicd
layout: post
---

DevOpsに関してはあまり知りません。

そんな中GitHub + CircleCIでDevOpsするというセミナーがあったので参加してみることにしました。

そのまとめです。

※自分の理解をまとめているため、セミナーの内容そのままとは限りません。

## DevOpsとは

* 企業の競争力を上げるために開発を早くしたい
* 一方では品質は維持したままにしたい
* 有名な無限大のループ
* 原則
  * 組織間の壁を無くす
  * 失敗を受け入れる
  * 小さく変更する
  * ツールを使って自動化する
  * あらゆるものを計測する

## DevOpsの効果

* デプロイ頻度：46倍
* コミットからデプロイまでのリードタイム：1/440
* 平均障害復旧時間：1/170
* 変更失敗率：1/5

## CI/CDにおけるDevOpsの効果

* Continuous Integration
  * すべてのコミットに対してビルドとテストを繰り返すこと
  * 失敗した場合に素早く修正することが可能となる
  * できること
    * テスト
    * ビルド
    * 静的コード解析
    * 脆弱性チェック
    * テストサマリー
  * 何を解決するか
    * 早い段階でバグを発見できる
    * コードの品質が上がる
    * masterブランチの安全保証
* Continuous Delivery
  * CIに加えて本番環境またはテスト環境にリソースがデプロイされること
  * デプロイは手動で行う
  * 何を解決するか
    * 属人化の防止
    * 作業者の工数削減
    * 迅速なロールバック

## GitHubにおけるDevOpsの効果

* GitHubのプルリクエスト(GitHub Flow)を使えばコラボレーションを促進できる

## CircleCIの機能紹介

* ワークフロー
  * ステージごとのリスタート(ブラウザのテストなど)
  * スケジュール機能
  * マニュアル承認
  * ブランチ指定
  * タグ指定
* Dockerをサポート
* SSHデバッグ
* Orbs
  * 記述が上長なものはパッケージ化が可能

## GitHub

* GitHubActionsはCIツールを置き換えるためのものではない

## サイボウズの事例紹介

* [スライド](https://www.slideshare.net/miyajan/github-circleci)
* オンプレとクラウドそれぞれでCircleCIとGitHubを作成
* サーバ台数1,500台
* GitHubを使った感想
  * プルリクを使ったレビューが良い
  * タスクごとにトピックブランチを切っている
  * 小さな変更を繰り返すスタイルになった
  * デプロイメントパイプラインによる高速なフィードバッグが可能になった
* CircleCIを使った感想
  * Jenkinsを使っていた
  * Jenkinsは悪いわけではないけど規模が大きくなるに連れてプラグインの管理などが大変になってくる
  * 2.0から機能が増えて使いやすくなった
  * 認証のコントロールはGitHub側に任せることができる
* 実際のプロジェクト
  * git pushするたびインフラの変更を反映
  * 一日一回ゼロから環境が構築できることを確認
  * チームごとに環境を持つ(ブランチと環境を対応付けている)
  * 毎日8時で強制削除するので早く帰れる
  * チームをまたいでの共同作業が可能になった

## これを聞いて何をやろうと思ったか

ゼロから環境が構築できるか、といったところはずっとやりたいなと思っていたので、先行事例があってやはりやりたい気持ちが高まってきました。

terraformを利用している中でこれをやらないのはもったいないです。

私の所属している組織では毎日深夜0時に開発環境を落とすので、8時にするのは中々思い切った決断だなと思いました。

流石にこれは組織に合わせた運用になりそうですね。

## まとめ

だいたいの概念は知っていることだったので少し物足りなく感じました。

ただ、自分のところが実践できているかと言うと、サイボウズの事例ほどには出来ていないと感じています。

ブランチと環境の戦略など、自分も今通っている道なのでまだまだこれからだと思いました。
