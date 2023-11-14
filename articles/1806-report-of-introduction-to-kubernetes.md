---
title: "入門Kubernetesに参加してきました"
emoji: "🏃‍♂️"
type: "tech"
topics: [Kubernetes, 勉強会]
published: false
published_at: 2018-06-06 10:14
---

# はじめに

https://supporterzcolab.com/event/406/

サポーターズ CoLab の『入門 Kubernetes』に参加してきました。

GKE のチュートリアルをやったことがあるのと、GKE 入門の勉強会に参加したことがあるくらいで、k8s はまだまだ理解不足だったので今回の勉強会に参加しました。

とてもわかりやすかったです！
Docker の前提知識があれば誰でも聞ける内容だったと思います。

「こういう課題があったからこういうサービスや概念が生まれた」

のようにサービスや概念の紹介だけではなく、「どういった課題を解決するため」という技術が生まれた背景やデファクトスタンダードになった理由とともにストリー仕立てに構成されていたため非常に飲み込みやすかったです。

また、プレゼンの仕方も勉強させいただきました。ライブコーディングはたまに見ますが、ホワイトボードを使う講義は初めてだったので新鮮でした。

要所要所で笑いを誘うテクニックも含め、聴講者を集中させる技術を自分も磨きたいです。

ホワイトボードと QA の部分のみですが以下メモ書きです。

https://bit.ly/2KEUUWu

## 自己紹介

- 片岡航平さん(@p\_\_oka)
- 株式会社ワクテカ CTO
- FITRA, FITRA Workout (DMM 英会話の Fitness 版)

## Docker とは

- コンテナ型仮想化環境
- プロセスを隔離させる

## Kubernetes

- コンテナオーケストレーションサービス

## Case Study

- Rails アプリケーション
- Rails のプロセスを常に 4 つ起動することを保証したい
  - `deployment`
    - image とプロセス数を定義する
    - オートスケールも可能（4~10）
    - `RollingUpdateStrategy`
- ロードバランシングを適切に設定することが難しい
  - `Service`
  - Service 間での通信（マイクロサービス）
  - 外部からのアクセスがない場合は必要ない
- どうやって外部に公開するか
  - `ingress`
    - `Service` との連携する
    - 一つのドメインで異なる `Service` に通信できる
    - できるが推奨されていない
- 環境変数の管理
  - ConfigMap
  - ハードコードしかできない？大きな問題
  - image を置換する
  - CI で sed しまくる
  - `Kubernetes Helm`

## コンテナランタイマー

- gVisor
- Docker は影響力を失いつつある
  - 抽象レイヤーがないという脆弱性がある

## クラスタ

Docker コンテナを実行するために用意された実行環境

## ノード

物理（仮想）マシン

## ロギング

- GCP の場合は意識せず stackdriver にたまる
  - stackdriver は監視もできる
- アプリケーションのログは全て標準出力にだす
- DataDog は Kubernetes 完全対応
- サイドカー（request ヘッダに requestID を入れる）

## Istio(イスティオ)

- `Envoy` を入れるなら `Istio` ごと入れるとよい

https://speakerdeck.com/ladicle/istiotogong-nimaikurosabisunili-tixiang-kae

## Red Hat Openshift

- 本番環境適用に向けた多様な機能
