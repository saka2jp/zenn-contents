---
title: "Alexa Salon #1 に参加してきました"
emoji: "🏃‍♂️"
type: "tech"
topics: [Alexa, 勉強会]
published: false
published_at: 2018-06-16 05:47
---

# 感想

https://dev.classmethod.jp/news/180614-alexa-salon/

Alexa Salon に参加してきました。スキル開発をするにあたって色々勉強させていただきました。じゃんけん大会負けてしまいましたが、本も読んでみたいです。

https://amzn.to/3YnehIb

SIN さんのブログとてもわかりやすかったのでおすすめです。

https://dev.classmethod.jp/cloud/alexa-sdk-v2-second/

# 基礎から学ぶ VUI の勘所

## 自己紹介

- 清野剛史さん
- クラスメソッド・テクニカルエヴァンジェリスト

## VUI とは

- Voice User Interface
- CUI - GUI - Web UI - Touch UI (mobile) - VUI
- UX: User Experience

## 会話における法則

- Voice Communication
- メラビアンの法則（視覚情報：58%、聴覚情報：38%、言語情報：7%）
  - 言語情報は 10%未満
  - Echo は 20~30%で戦う必要がある

## VUI 設計で気をつけること

- NUDGE
  - 行動経済学用語
  - ユーザーが相手の自然かつ無意識に意図通りの行動を起こすように環境を整え促すこと

## 留意点

- 選択肢を例示する
- はい/いいえで答えられるように促す
- 一息で言い切れる長さ
  - ユーザーが鬱陶しいと思わないように
  - 書き言葉をそのまま使っては行けない
  - 録音して長いと思ったところは切る
- 「〜ですか？」：は等価、「〜ですね？」は強調
  - はいと言ってほしい時など

## GRAPH BASE UI vs FRAME BASE UI

- VUI では FRAME UI が良い

## MULTI-TERN

- ユーザーがたくさん話しすぎるのはだめ
- ピンポンの会話にする

## VUI 設計の作り方

- 目的を決める
  　- なにをしたいか
  　- そのために UX 的に音声でどう体験してほしいか。
  　- どこまで VUI にするか。
  　- VUI 以外との連携は。
  　- ユースケースを想像する。毎日使うのか、必要な時に使うのか。
- キャラクター設定
  - ペルソナを作り、発話に一貫性を持たせる
  - ユーザーとの距離感
  - Alexa の立ち位置
- ハッピーパス
  - ストレートな対話を作り、これを軸に
- INTENT/SLOT の設計
  - State-Intent-Slot のスコープを決める
  - Intent はなるべくシンプルにまとめる
- UI 図
  - ハッピーパスを肉付けする
  - 重要な Slot と必須ではない Slot を区別しておく
  - Slot の最中に別 Intent に飛ぶケースを想像しておく
  - 結構修正するので具体的な発話を書かない
- UTTERANCE 表の作成
  - 規模によっては以外と重要
  - 被ってないか確認
- シノニム/バリエーション
  - ひたすら想像力
  - 5 種類をランダムに
  - 頻度によってメッセージを帰る
- エラーハンドリング
  - キャラクラー設定を元に判断する
  - 謝りすぎない
  - 疑問形で終わる
  - エラー回数によって選択肢を明示する
  - ストップへの誘導

## まとめ

- VUI は Alexa 開発のキモ
- 図を書くまえの準備がクオリティを上げる

# Alexa-SDK について

https://www.slideshare.net/furuya02/alexa-sdk-alexa-salon

## 自己紹介

- 平内真一さん
- アプリケーションエンジニア
- ブログを業務時間に書いていい

## Alexa SDK

- 煩わしい作業を SDK に押し付けて、開発者はロジックに集中できてウハウハ
- リクエスト・レスポンス JSON の抽象化
- データの保存
- 処理のルーティング（ハンドラ登録）
- ステート（状態）

## バージョン 1 と 2

- 4/18 以前
- はじめての Alexa スキル開発
- Lambda のテンプレート

## バージョン 2

- ハンドラの実装法穂が変わった
- データ永続化の記述が明確になった
- コア機能のみの利用可能
- Node8 対応(async/await)
- DynamoDB 以外も利用可能
- Alexa サービス API のラッパー提供
- Java + Node.js

https://dev.classmethod.jp/cloud/alexa-sdk

## 変更点

- tell, ask の廃止 --> ResponseBuilder
- ステートの廃止
- ハンドラのインターフェース canHandle
- データ永続化の要領
- 例外時の処理

## 最後に

- Ver1 の方が簡単に描ける
- Ver1 に未来はあるのか・・・？
- これから取り掛かるなら Ver2 でいいかも
- 個人的には、Ver2 押し
  - データ永続化が明確
  - canHandle も慣れると悪くない
  - Alexa API へのアクセスが超簡単
  - async / await 最高
  - TypeScript で書ける

# 大質問大会

Q.対話モデル・対話フロー設計時のオススメの方法があれば教えてください。

- インテントから作る
- ハッピーパス

Q.開発方法、事例、活用ポイントについて

- Cloud9
- alexa-conversation テスト

Q.ニーズが高いスキルはどのようなスキルか

- Amazon 的には子供向けスキル
- US でもキッズ向けが人気
- キャラクターの声がなんかしてくれる

Q.広告ポリシー

- 子供向けは広告だめ

Q.勝手に喋り出す機能はないのか？

- プッシュ通知機能はある

Q.バックエンドに機械学習を絡めることはできるか

- あったら面白そう
