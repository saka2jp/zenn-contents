---
title: "GKEで学ぶKubernetes入門に参加してきました"
emoji: "😀"
type: "🏃‍♂️"
topics: [Kubernetes, 勉強会]
published: true
published_at: 2018-06-07 12:34
---

# はじめに

https://supporterzcolab.com/event/291/

サポーターズ CoLab の 「GKE で学ぶ Kubernetes 入門」に参加してきました。講師は[株式会社カブク](https://www.kabuku.co.jp/) の 吉海さんという方でした。

# スライド

https://speakerdeck.com/tinjyuu/gkedexue-bukubernetesru-men

- 意味はギリシャ語で航海長
- [Google から世界へ : Kubernetes 誕生の物語](https://cloudplatform-jp.googleblog.com/2016/08/google-kubernetes.html)
- Google が最初のプロジェクトとして[CNCF](https://www.cncf.io/)に寄贈
- Kubernetes はなぜ注目されているのか
  - コンテナ管理ツールの需要が高い
  - GKE, EKS, AKS の台頭
- GKE はお手軽コマンドで Kubernetes を構築可能
  - kubectl create "Deployment の定義ファイル(yaml)"
  - kubectl get pod（動作確認）
- 料金はクラスタのノードを構築している GCE のインスタンス料金のみ
- GKE をなぜ使うのか
  - 簡単に構築できる
  - 導入コスト・運用コストが低い
  - オンプレにクラスタを一から構築するのは大変
  - StackDriver logging を設定するだけでログを簡単に管理できる
  - Kubernetes は Google 自らが開発元である
  - Google はどのベンダーよりも早い
- 概念や基本用語について
  - [「Kubernetes によるスケーラブルな Web アプリ環境の構築」連載一覧](https://codezine.jp/article/corner/714)
- 無料範囲内で CloudShell を利用すると余っているインスタンスを利用するため動作がもっさりする
- GKE のマスターは隠蔽されているため、マスターの内部的な動きを知るためにはソースコードを読むしかない
- マルチマスター（冗長構成）は可能だが公式非推奨
  - 複雑すぎて運用が大変
  - マルチリージョンにするのがよい？
- コミュニティ
  - [Kubernetes Meetup Tokyo](https://k8sjp.connpass.com/)
  - [GCPUG](https://gcpug-tokyo.connpass.com/)
  - [Slack](http://slack.k8s.io/) #jp-events #jp-users
- 書籍
  - [Software Design 2018 年 3 月号](https://amzn.to/3Jez2BB)【第 2 特集】Docker をさらに便利に使いたい!
  - [入門 Kubernetes](https://amzn.to/3KXSeot)
- 吉海さんのおすすめ書籍
  - [SRE サイトリライアビリティエンジニアリング](https://amzn.to/3JeQJB2)
  - 運用周りに興味があるなら一読すると幸せになれるらしい
- Google Assistant のアプリを作ると GCP のクレジットと T シャツをゲットできる
- [カブクククック](https://www.wantedly.com/projects/154408)
