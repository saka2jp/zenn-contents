---
title: "JapanContainerDays v18.12 これだけは目を通しておきたいセッションベスト3"
emoji: "💡"
type: "idea"
topics: [GitLab, Kubernetes, Docker, 勉強会]
published: false
published_at: 2018-12-08 05:48
---

# はじめに

[Kubernetes2 Advent Calendar 2018](https://qiita.com/advent-calendar/2018/kubernetes2)の 8 日目の記事です。

https://qiita.com/advent-calendar/2018/kubernetes2

12/4・5 で開催された JapanContainerDays v18.12 に参加してきました。

https://containerdays.jp/

どのセッションもとても勉強になり甲乙はつけがたく、また、自分自身甲乙をつけられる立場ではありませんが、本記事では**個人的におすすめ**だったセッションを「これだけは目を通しておきたいセッションベスト 3」として**独断と偏見で**ご紹介させていただきます。

※注意

- ECS でのアプリケーション構築・運用経験があります。
- GitLab を業務で使っています。個人的にも GitLab と GitHub の両方を使っています。
- Kuberntes は独学で「[入門 Kuberntes](https://amzn.to/3EZyBZj)」・「[コンテナ・ベース・オーケストレーション](https://amzn.to/3kSWiLZ)」を読んだことがあり、GKE や AKS の KaaS でサンプルアプリケーションをデプロイしたことがあるレベルです。
- 参加していないセッションは考慮していません。

# これだけは目を通しておきたいセッションベスト 3

## 1. Microservices on Kubernetes at Mercari

### スライド

https://speakerdeck.com/tcnksm/microservices-platform-on-kubernetes-at-mercari

### [YouTube](https://youtu.be/6TNqJb97gH0?t=1545)

Twitter で多くの反響を集めたキーノートの一つでした。  
動画とスライドをぜひ見ていただきたいのですが一応まとめておきます。

### テーマ

- なぜメルカリがマイクロサービスアーキテクチャの基盤に Kubernetes（そもそもコンテナ技術）を選択するのか
- Kubernetes を利用して今後どうしていきたいのか

### テーマに対する解答（独自の解釈を含む）

- なぜメルカリがマイクロサービスアーキテクチャの基盤に Kubernetes（そもそもコンテナ）を選択するのか  
  -> ビジネス競争力のあるソフトウェアをリリースするため  
  -> ビジネス競争力を高めるためにはスピード感が大切  
  -> スピード感をマイクロサービスアーキテクチャと Immutable Infrastructure によって実現する  
  -> マイクロサービスアーキテクチャによる独立・自立したチーム作りで自走可能なチームがスピード感を生み出す  
  -> Immutable Infrastructure による決定論的なデプロイで本番環境へのリリースの心理的安全性を高め、開発者のデプロイに対するハードルを下げることでスピード感を生み出す  
  -> マイクロサービスアーキテクチャ・Immutable Infrastructure を実現するためにはコンテナが向いている  
  -> オーケストレーションサービスには ECS・Heroku などの PaaS ではなく拡張性の高さとエコシステムの豊富さが特徴である Kubernetes を選択する

- Kubernetes を利用して今後どうしていきたいのか  
  -> Kubernetes の拡張性の高さとエコシステムの豊富さを活用して柔軟にビジネス要件に対応していく

### 感想

組織として「なぜその技術に取り組むのか」という定義ができているのが大変素晴らしいと感じました。  
このトークを聞いて思い出したのは[TED](https://www.ted.com/)で公開されているサイモン・シネックさんによる「[優れたリーダーはどうやって行動を促すのか](https://embed.ted.com/talks/lang/ja/simon_sinek_how_great_leaders_inspire_action)」という講演です。

https://www.ted.com/talks/simon_sinek_how_great_leaders_inspire_action?language=ja&utm_campaign=tedspread&utm_medium=referral&utm_source=tedcomshare

ここで紹介されているのは、偉大な指導者や組織に共通しているあるパターンです。  
それは「WHAT」ではなく「WHY」から考え行動するというもので、それが結果的に人の心を動かすそうです。  
なぜコンテナなのか、なぜ Kubernetes なのかが最初に共有されていたため、今後の展望も納得感があり多くの人が共感し Twitter での反響に繋がったのではないかと思いました。

自分自身も学習する際は、なぜその技術を身につけるのかという自問自答をしていきたいです。

## 2. GitLab によるコンテナ CI/CD パイプラインのこれから

### スライド

https://www.slideshare.net/ShingoKitayama/gitlab-auto-devops-with-container-cicd

GitLab ユーザとして押さえておきたい内容だと感じたためピックアップしました。

### テーマ

- なぜ GitLab の Auto DevOps がよいのか
- GitLab の Auto DevOps とは何か

### テーマに対する解答（独自の解釈を含む）

- なぜ GitLab の Auto DevOps がよいのか  
  -> 多くの企業は DevOps パイプラインにおいて 3~6 個のツールを利用しており、技術選定コストや学習コストがかかっているという課題があるため  
  -> GitLab は DevOps パイプラインにおけるツールを包括して提供している（Complete DevOps）  
  -> ソフトウェア開発に時間を費やす環境を提供するため  
  -> CI/CD パイプラインに必要な設定を動的に行う仕組み（PaaS）を提供している=>Auto DevOps

- GitLab の Auto DevOps とは何か  
  -> Auto DevOps is PaaS for Kuberntes  
  -> 開発者はコードを書いてプッシュするだけ。残りは全て Auto DevOps がよしなにやってくれる  
  -> 残りとは、コード品質チェック・コードや Docker イメージのセキュリティチェック・依存関係チェック・一時的なアプリケーションレビュー環境の構築・Kubernetes クラスターへのデプロイ・Sitespeed.io による Web ページのパフォーマンス測定・Prometheus による監視など

### 感想

GitLab をまだまだ使いこなせていないことを実感しました。
初期の導入を手動で行なったり定常的な作業を手動で実施していたりという部分が何箇所か思い当たったので、そういう部分は Auto DevOps で自動化できそうだなと感じました。

Auto DevOps は導入は大変かもしれませんが運用は比較的楽になるのではと感じました。
また、Auto DevOps は Google アカウントにサインインしていれば GitLab から動的に GKE クラスターを作成してくれるという機能があったり、昨日[Azure から GCP への移行](https://gihyo.jp/admin/clip/01/linux_dt/201806/26)が話題になりましたが GitLab としても GCP との連携を推し進めていたりと、GitLab と GCP の親和性はかなり高いです。
Auto DevOps を導入するのであれば、GCP を利用するのが良さそうです。

## 3. 本番環境の Kubernetes マニフェストに 最低限必要な 7 のこと

### スライド

https://speakerdeck.com/masayaaoyama/jkd1812-prd-manifests

### テーマ

- 本番環境の Kubernetes マニフェストに 最低限必要な 7 のことは何か

### テーマに対する回答

- 本番環境の Kubernetes マニフェストに 最低限必要な 7 のことは何か  
  -> コンテナのライフサイクル（起動時）  
  -> コンテナのライフサイクル（停止時）  
  -> ヘルスチェック  
  -> メンテナンスとアップデート  
  -> スケジューリング  
  -> リソースの割り当てと基準  
  -> カーネルパラメータのチューニング  
  -> インターネットからのアクセス制御

### 感想

とりあえずこの 7 つの Tips を抑えておけば本番環境に Kubernetes を導入することができる（運用はまた別）とのことで、わかりやすく端的にまとめられている資料でした。

特に初期化処理の方法やコンテナ停止時の挙動に関する話はとても参考になりました。

[Kubernetes 実践ガイド](https://amzn.to/41Jl8yG)も合わせて読みたいですね。

# 全体の感想

> Kuberntes はコンテナオーケストレーションとしてデファクトスタンダードとなった by Chris Aniszczyk / キーノートより

Kubernetes の話を聞きにいったつもりでしたが、Kubernetes とその周辺のエコシステムを活用していかに運用を楽にするか、いかにビジネス競争力が高いソフトウェアを作るかという話にシフトしているようでした。

コンテナ ⇨Kubernetes⇨ クラウドネイティブ（ここの話が多かったです）

事実、JapanContainerDays は今年で終わりで来年からは Cloud Native Days 2019 となります。  
勉強になることだらけでとても刺激的な 2 日間でした。運営のみなさまありがとうございました。  
次回は一回り成長して参加したいです。

（聞けていないセッションが結構あるのでおすすめのセッションがあればぜひ教えてください！）
