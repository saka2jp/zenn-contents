---
title: "第3回 Google Cloud INSIDE Games & Appsに参加してきました"
emoji: "🏃‍♂️"
type: "tech"
topics: [勉強会]
published: false
published_at: 2018-03-21 09:55
---

# はじめに

はじめて[Google Cloud INSIDE Games & Apps](https://developers-jp.googleblog.com/2018/01/3-google-cloud-inside-games-apps.html)に参加してきました。

> このイベントは、事例をもとにアプリ開発の裏側を徹底解説することで、より実践的な知識と経験の共有の場となることを目指しています。

とありましたが、今回のテーマが「マイクロサービスのアーキテクチャと事例紹介」でしたので、GCP でマイクロサービスをどう実現したか（するか）という内容が主でした。アプリ開発に関しては Firebase が少し出てきたかなくらいです。以下簡単ですが、内容の共有です。詳細はスライドを確認してください。

# Google Cloud Platform で実現するプロダクションレディマイクロサービス

https://www.slideshare.net/slideshow/embed_code/key/fnEuOGwc7lvx0l

GCP を使えばプロダクションレディマイクロサービスが簡単に構築運用できますよという話でした。
[プロダクションレディマイクロサービス](https://amzn.to/3IPourg)によると、以下の原則を備えたマイクロサービスを作れば、本番トラフィックを任せられる可用性の高いシステムが構築できるとのことです。

- 安定性
- 信頼性
- スケーラビリティ
- 耐障害性と大惨事への対応
- パフォーマンス
- 監視
- ドキュメント

上記の原則を備えたマイクロサービスを GCP でどう実現するか。

- 安定性 / 信頼性
  - 標準的なデプロイパイプライン
    - `GAE` : カナリアリリース・A/B テスト
    - `GKE` : Spinnaker を併用
- スケーラビリティ
  - `GAE` : Web アプリのための統合 PaaS
  - `GKE` : マネージドの Kubernetes (KaaS)
- 耐障害性と大惨事への対応
  - サービスごとにリリースチェックリストがある
- パフォーマンス
  - タスクの処理効率
    - `Task Queue` : App Engine に組み込み済みのキュー
    - `Cloud Pub/Sub` : Publisher/Subscriber 型のメッセージサービス
  - データ処理
    - `BigQuery` , `Cloud SQL` , `Cloud Storage` など
    - [最適なストレージを選択するための Y/N チャート](https://cloud.google.com/storage-options/)
- 監視
  - `Stackdriver` : 運用のための統合ツール -ドキュメント
  - `Cloud Endpoints` : 標準化されたエンドポイントを提供

チュートリアル

- [Google App Engine](https://cloud.google.com/appengine/docs/standard/python/tutorials)
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/tutorials/)
- [Google Endpoints](https://cloud.google.com/endpoints/docs/frameworks/tutorials)

# バス NAVITIME on GKE, Inside of Bus NAVITIME.

https://www.slideshare.net/slideshow/embed_code/key/p8MvREyZC6BwWW

GKE に移行した手順と苦労した点の話でした。

# GCP 本格採用で遭遇した課題とマイクロサービス的解決

https://www.slideshare.net/slideshow/embed_code/key/n8vIRLVnq9Q7gt

CQRS という切り口で C（Command：更新系）を GAE flexible × Scala で Q（Query：参照系）を GAE standard × Go で[Unipos](https://unipos.me/ja/)を開発したという話でした。
GAE flexible はインスタンスの起動時間が分単位だったり、セキュリティの関係で（？）週に 1 度リブートされたりするため注意が必要とのことです。

# おわりに

はじめての六本木ヒルズは怖くて終始挙動不審だったと思います。セッションの内容はマイクロサービスに関する話が主でした。もう少しアプリ開発ならではの話も聞きたかった気もしますが、すごく勉強させていただきました。今回は予定があり懇親会に参加できなかったですが、次回はぜひ参加したいです。
