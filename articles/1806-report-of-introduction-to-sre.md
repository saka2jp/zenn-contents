---
title: "SRE がよく利用するソフトウェアの理解と分類講座に参加しました"
emoji: "🏃‍♂️"
type: "tech"
topics: [SRE, 勉強会]
published: true
published_at: 2018-06-07 11:27
---

# 感想

https://supporterzcolab.com/event/395/

サポータズ CoLab の「SRE がよく利用するソフトウェアの理解と分類講座」に参加しました。

SRE エンジニアとして働くためには並大抵ならぬ努力がいることを感じざるを得ませんでした。

一方で、SRE エンジニアとしての働き方にかなり興味を持ったのも確かです。

## **「自動化」** ・**「制御してやったという達成感」**

SRE エンジニアとしてのモチベーションを尋ねたときの回答です。

それを聞いたとき、はじめて自分が書いた CI の yml が期待通りの挙動をしてくれたときに幸福感を感じたことを思い出しました。

コードをゴリゴリ書くことが 1 番の楽しみでしたが、なるほどそういう働き方もあるなという気づきをいただきました。

## **「一番困っていることから手をつけるのが良い」**

オススメの勉強方法です。まずは課題を見つけるところから少しずついらいろな技術に精通していけたらと思います。

裏の勉強会がかなり盛り上がっていたようですがこちらの勉強会に参加してよかったです。

帰宅後、それぞれのサービスをググって概要だけでも眺めてみましたがかなり全体像がつかめた気がします。

今回紹介していただいたサービスだけでも

な、なな、なんと

**43 個**

ありました。（驚）

全てに精通する必要はないですが、どういったツールなのか、どういう違いがあるのかくらいは理解しておきたいですね。

https://speakerdeck.com/katsuhisa91/sre-tools

以下、メモ書きです。

# メモ書き

## 自己紹介

- 北野勝久さん
- スタディスト SRE チームリーダー

## 困っていたこと

- コード管理されておらずブラックボックス化したインフラ
- 障害が発生してもモニタリング・管理が貧弱で状況がすぐに掴めない
- 何からやったらいいかわからないし周辺技術もたくさんある

## ひらすら勉強

- 効率最悪
- 時間の無駄

## これを知ってれば

- SRE の全体像をみれていれば...
- 闇雲に走る必要はなかった

## SRE?

- Google 発
- システム管理者, DevOps とは何が違うのか
- ソフトウェアエンジニアが、システム運用をしたらどうなるか？という観点で再定義されたシステム管理者の役割
- class SRE inplements DevOps
  - それ以外の役割も担う
  - それ以外の役割は各社で異なり厳密な定義はない

[https://youtu.be/uTEL8Ff1Zvk:embed:cite]

## 話す範囲

- 話さないこと
  - ベーススキルセット
- 話すこと
  - SRE の責務範囲の枠組み
  - ツール（AWS メイン）

## Infrastructure as Code

- システム構成図をコード管理する
  - [Terraform](https://www.terraform.io/)
  - [AWS CloudFormation](https://aws.amazon.com/jp/cloudformation/)
- プロビジョニング（nginx, MySQL のインストール）
  - [Ansible](https://www.ansible.com/)（SSH で実行可能）
  - [Chef](https://www.chef.io/chef/)（サーバーにインストールして実行）
- インフラのテスト
  - [Serverspec](https://serverspec.org/)
  - [awsspec](https://github.com/k1LoW/awspec)
- マシンイメージの作成
  - [Packer](https://www.packer.io/)

## Monitoring

- たくさんありすぎる
- [NewRelic](https://newrelic.com/)（使ってる会社が多い・有名）
- [DataDog](https://www.datadoghq.com/)
- [AWS CloudWatch](https://aws.amazon.com/jp/cloudwatch/)
- [GCP Stackdriver](https://cloud.google.com/stackdriver/)
- [Mackerel](https://mackerel.io/ja/)
- [Prometheus](https://prometheus.io/)
- [Zabbix](https://www.zabbix.com/jp/)
- [Elasticsearch](https://www.elastic.co/jp/products/elasticsearch) / [Kibana](https://www.elastic.co/jp/products/kibana)
  - ログを溜め込んだサーバーで検索・フィルタリング
  - グラフや表で可視化

## Log

- ログの転送ツール（フィルタリングや加工も可能）
  - [fluentd](https://www.fluentd.org/)（機能拡張しやすい・プラグインが豊富）
  - [Beats](https://www.elastic.co/jp/products/beats) / [Logstash](https://www.elastic.co/jp/products/logstash)
    - ex) パスを正規化してフィルタリング

## ChatOps

- Slack などのチャットツールからコマンドなどを入力することで何かしらの作業を行う
  - [Hubot](https://hubot.github.com/)

## CI/CD

- [Circle CI](https://circleci.com/)
- [Travis CI](https://travis-ci.org/)
- [Spinnaker](https://www.spinnaker.io/)
- [Jenkins](https://jenkins.io/)
- AWS Code シリーズ
  - [CodeBuild](https://aws.amazon.com/jp/codebuild/)
  - [CodeDeploy](https://aws.amazon.com/jp/codedeploy/)
  - [CodeCommit](https://aws.amazon.com/jp/codecommit/)
  - [CodePipline](https://aws.amazon.com/jp/codepipeline/)
  - [CodeStar](https://aws.amazon.com/jp/codestar/)
  - etc...

## Development Environment

- [Docker](https://www.docker.com/)
- [Virtualbox](https://www.virtualbox.org/) + [Vagrant](https://www.vagrantup.com/)

## Infrastructure Automation

- イベント駆動アーキテクチャでインフラ操作自動化することを意図して記載
- 「○○ が起きたら、XXX が起動し、△△ する」という XXX の部分
  - [AWS Lambda](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/welcome.html)
  - [StackStorm](https://stackstorm.com/)

## Security

- IDS：検知
  - [OSSEC](https://www.ossec.net/)
- IPS：侵入防止
  - [fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page) / [iptables](https://ja.wikipedia.org/wiki/Iptables)
- IDS/IPS
  - [Deep Security](https://www.trendmicro.com/ja_jp/business/products/hybrid-cloud/deep-security-data-center.html)
- WAF
  - [AWS WAF](https://aws.amazon.com/jp/waf/)
  - クローリング元の IP アドレスを監視し、一定期間に一定リクエスト数を超えたら IP 間の通信を遮断
- Audit
  - [AWS CloudTrail](https://aws.amazon.com/jp/cloudtrail/)

## Data

- データ分析基盤づくりを SRE が担当する会社も多い
- DWH
  - [Redshift]()
  - [BigQuery]()
- [Storage]() + [Query]()
  - [AWS S3]() + [Athena]()
- BI
  - [Redash]()
  - [Metabase]()
  - [DOMO]()
  - [AWS QuickSight]()

## Incident Management

- インシデント管理
  - [PagerDuty](https://www.pagerduty.com/)
- ステータス管理
  - [StatusPage](https://ja.atlassian.com/software/statuspage)（お客さん向け）

## まとめ

- 全体像を把握し、それぞれのツールの役割を理解するのが良い

## QA

- オススメの勉強方法や書籍
  - SRE book for FREE in English
    - Google と自分たちの違いを理解する
    - 分散処理が世界で最も強い会社
    - 自分の会社と Google との差分だけ認識して学習する
    - 書いてあることを全て実現するのは無理だし意味がない

https://www.publickey1.jp/blog/17/googlesite_reliability_engineering.html
