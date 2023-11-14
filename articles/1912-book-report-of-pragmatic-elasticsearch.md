---
title: "「Elasticsearch実践ガイド」から学んだこと。"
emoji: "📖"
type: "tech"
topics: [Elasticsearch, 書評]
published: false
published_at: 2019-12-27 11:54
---

# 概要

- 著作： **「[Elasticsearch 実践ガイド](https://amzn.to/3KUbl2D)」**
- 著者： **惣道 哲也**
- 著者経歴：1998 年に日本ヒューレット・パッカード株式会社に入社。半導体テスタのソフトウェア開発を担当し、その後、主に通信・金融系システムのインフラ構築・アプリケーション開発・プロジェクトマネージャに従事。2012 年より主にオープンソースソリューションの提案・設計・導入・技術コンサルティングを行う組織にて、Hadoop、クラウド、コンテナ/DevOps、Deep Learning 分野におけるテクニカルアーキテクトとして活動している。また、社内では全社横断的な技術コミュニティの日本担当リーダとして技術啓蒙を行うかたわら、社外でも数々のイベント・セミナーや技術コミュニティで登壇している。

# 目的

Elasticsearch のセマンティックを理解する

# 要約

Elasticsearch は索引検索型の全文検索を行うソフトウェア。検索性能や可用性が高い。また、REST API でリソースの操作が行える柔軟性の高さも兼ね備える。さらに、Elastic Stack と連携することでログ収集や可視化などができる。

# 感想

全文検索を単語を初めて知ったが非常に興味深かった。膨大なデータから特定のデータを瞬時に見つけなければいけないので、難しそうな仕組みなのかなという勝手な印象があったが、意外とシンプルなシステム構成だということがわかった。Elasticsearch の直感的な インターフェースはとても使いやすい。Elasticsearch に入門できるよい書籍だった。さらに手を動かして実践的なノウハウ身に付けていきたい。

# 次のアクション

実際に手を動かす。以下の文献などがよさそう。

- [Hello! Elasticsearch](https://medium.com/hello-elasticsearch)
- [Classmethod 連載](https://dev.classmethod.jp/server-side/elasticsearch-getting-started-01/)
