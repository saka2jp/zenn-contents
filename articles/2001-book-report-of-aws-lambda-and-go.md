---
title: "「Goで学ぶAWS Lambda」から学んだこと。"
emoji: "📖"
type: "tech"
topics: [Serverless, Golang, AWS, 書評]
published: false
published_at: 2020-01-04 07:45
---

https://booth.pm/ja/items/1034858

今回は技術同人誌を読みましたが、後に正式出版されたみたいです。

# 概要

- 著作： **「[Go で学ぶ AWS Lambda](https://amzn.to/3ZkU2vX)」**
- 著者： **杉田 寿憲**
- 著者経歴：株式会社メルペイのバックエンドエンジニア。Go と GCP のマイクロサービスと闘争中。

# 目的

Golang × SAM での開発の勘所を掴む

# 要約

以下の 3 つのプロジェクトを通じて Golang × SAM における開発方法やパターンを紹介する。

- S3 の Put イベントをトリガーとした単一 Lambda の構築
- SNS と SQS によるファンアウトパターンの構築
- API Gateway と DynamoDB を使った URL 短縮 API の構築

# 感想

さすがメルペイの現役エンジニアということもあり、内容はかなり実践的だったように感じた。

プロジェクトを 3 つ実際に開発してみるという構成がよかった。それぞれ写経することで反復練習になり、SAM や Lambda に慣れ親しめた。また、環境構築もかなり実践的だった。anyenv や saw、direnv などは知らなかったので今後も活用したい。

# 次のアクション

Golang × SAM × Cognito で構成する認証有りのプライベートな API を作る。以下の文献がよさそう。

- [API Gateway + Cognito Auth + Cognito Hosted Auth Example](https://github.com/awslabs/serverless-application-model/tree/master/examples/2016-10-31/api_cognito_auth)
- [Controlling Access to API Gateway APIs](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-controlling-access-to-apis.html)
- [AWS SAM で Cognito UserPools の認証付き API を作成する](https://blog.youyo.info/post/2018/12/05/sam-apigateway-authorizer/)
