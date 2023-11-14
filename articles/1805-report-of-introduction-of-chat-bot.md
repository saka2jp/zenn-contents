---
title: "チャットボット設計・開発入門に参加してきました"
emoji: "📖"
type: "tech"
topics: [勉強会]
published: false
published_at: 2018-05-28 11:09
---

# はじめに

https://supporterzcolab.com/event/391/

サポーターズ CoLab で開催されたチャットボット設計・開発入門に参加してきました。業務でチャットボットの開発があるので設計・開発における勘所を知れたらなと思ったのが動機です。

# スライド

https://www.slideshare.net/slideshow/embed_code/key/3tTuThTC3e7OwM

# TL;DR

- チャットボットはとにかく小さく始めよう。
- チャットボットにしかできない機能に絞ろう。
- [ズボラ旅](https://www.cocolocala.jp/lp/zubora) のようなチャットボットサービスがウケる。

# 感想

- 最終的に自然言語処理を伴う機能開発をすることが多くなりそうなので Python で開発すると良さそう。
- 勤怠管理システムにおけるチャットボットにしかできない機能とは何か。
- チャットボットと RPA との組み合わせは面白そう。
- Facebook for developers の Messagenger プラットフォームで紹介されている[デザインのベストプラクティス](https://developers.facebook.com/docs/messenger-platform/introduction/general-best-practices)を設計や開発時に取り入れると良さそう。

# 以下メモ書きです。

## お前誰よ

- 岡野健三さん
- [株式会社アポロ](https://apolocorp.jp/)CEO
  - [APOLO](https://apolocorp.jp/apolochat/)
  - [ボトログ](https://botolog.com/)

## チャットボットとは

- メッセンジャーを用いた対話型アプリケーション

## 人工知能使うんでしょう？

- AI は必要ないこともある
- 「A Bot is a app!」 by Christopher Harrison

## どんなものがある？

- Facebook Messenger
- Wechat
- LINE
- etc...

## ユースケース

- 全ての課題を解決してくれるものではない
- 「私の仕事は、ボットを作るなと、人に言うこと」 by Christopher Harrison
- 「それチャットボットにする必要ありますか？」 by 及川卓也

## アンチパターン

- オンラン上で検索したり、探したりすることに慣れているもの
- 限りなく人間に近いコミュニケーションを目指すもの
  - Siri, りんなちゃん
- アプリ・WEB でできること
  - 天気確認
  - メール通知
  - バイト探し
  - etc...

## グッドパターン

1. Knowledge Base Bots
   - 何かすでに知識・ノウハウが用意されており、それらに対するアクセス方法が乏しいもの
1. 専門家を代替するもの
   - [DocsApp](https://www.docsapp.in/) : 医療チャットボット
     - [「インドの医療アプリ」が安心便利で何もかも革新的だった](https://www.houdoukyoku.jp/posts/15647)
   - [DoNotPay](https://www.donotpay.com/) : 法律チャットボット
     - [駐車違反に異議あり！大学生が開発した弁護士ボット「DoNotPay」の実力](https://www.ibm.com/think/jp-ja/watson/donotpay/)
   - [LEBER](https://leber11.com/) : 医師のチャットボット
     - [ドクターシェアリングアプリ「LEBER」運営の AGREE が、約 1 億円の第三者割当増資を実施](http://venturetimes.jp/venture-news/technology-medicine/36389.html)
1. 電話や対面での会話を代替するもの
   - ヤマト運輸
     - [ヤマト宅急便を LINE で使いこなせ！不在通知や再配達依頼もできて便利！](https://fatherlog.com/line/line-tips/3766)
   - ペコッター
     - [飲食店予約代行アプリにリニューアルした「ペコッター」が 1 億円を調達](https://jp.techcrunch.com/2018/02/08/bright-table-fundraising/)
   - こころから
     - [旅行アプリ「ズボラ旅 by こころから」がサービス開始も、発表から約 3 時間後でダウン](https://news.nifty.com/article/economy/economyall/12144-329387/)
1. ランディングページなどによるユーザー獲得を代替するもの（メールに比べ 6 倍くらいのコンバージョン）
   - Lemonade
     - [ミレニアル視点で考える、これからの保険ーー「Cover」と「Lemonade」が変革する保険業界](http://thebridge.jp/2018/02/cover-and-lemonade-insurtech)
   - 転職サイト Green
     - [【チャットボット発表】エンジニア転職に強い「Green」を展開するアトラエに Facebook Messenger "会話広告"パッケージの「fanp Ad」導入・運用をスタート](https://prtimes.jp/main/html/rd/p/000000023.000019209.html)
1. ソーシャル性(Facebook Messenger, LINE Bot)
   - メッセンジャーサービス上ですでに繋がっているので、再び繋がる必要性がない
   - 人狼 Bot
     - [【登録者 45 万人超】人気の LINE で使える人狼チャットボットの開発者とは？](http://mag.botolog.com/archives/37)
1. LINE Beacon を利用した位置情報活用
   - [ZOZO TOWN やユニクロも利用する「LINE Beacon」とは？ 使い方や活用事例を解説](https://unleash.tokyo/2017/12/31/line-beacon/)
1. タスク自動化
   - Slack

## 未来なるキラーボット

- 「ただスマホの時は誰も Uber を予想できなかった。限られた数個の Bot が世界を変えるだろう」 by Amir Shevat

## 開発フロー

1. 構想
2. ユーザーストーリーを全て書き出す
3. ビジュアル＆コピーブランディング
4. 開発・デザイン
5. テスト
6. リリース・改善

## 構想

- MVP を目指して解決する課題とそれを解決する最低限の手段を決める（チャットボットだけで完結するサービスは特に気をつける）
  - その機能はチャットボット以外で本当にできないのか？

## ユーザーストーリーを全て書き出す

- ユーザーがたどる状態・遷移を図にすると抜け漏れがなくわかりやすい

## ビジュアル＆コピーブランディング

- テンプレがある程度確立されているためそんなに難しくない
- キャラクター作りがチャットボット開発における特徴

## 開発・デザイン

- メッセージが同時に大量に送られた場合を想定して、どうトラフィックを裁くか考える
  - 非同期的な処理を心がける
  - Node.js で開発する場合 promise などは必須
- 基本的には洗濯しの画像や以前説明したビジュアル・コピーブランディングの部分が大きい

## テスト

- ターゲットユーザーとなる人に実際に触ってもらおう
  - あくまで前提条件のみ伝えて、使い方は説明しないでおく

## リリース・改善

- リリースはできるだけ最小限の機能でしよう
- 会話のログ分析が重要
  - アナリティクスツール [Botanalytics](https://botanalytics.co/)

## APOLO のしくじり

1. 機能を詰め込みすぎた
2. 音楽を探すことはアプリで十分できていた
3. ドコモの雑談対話 API を搭載することで期待感を煽ってしまった

**チャットボットだからこそできることに機能を絞る**  
**機能が多すぎると使われなくなってしまう！！**

## 今後の方向性〜海外〜

- マネタイズを考えているチャットボットは、B 向け（Slack を利用したもの）が多い
- C 向けチャットボットでどんどん小さなヒットが生まれている
  - Poncho, DoNotPay, Foxsy
- Facebook(wit.ai), Google(Api.ai), Microsoft(Maluuba)が自然言語処理系スタートアップに投資・買収
  - チャットボット × AI 領域に挑むのは敵が巨大すぎる

## 今後の方向性〜国内〜

- チャットボット以前より盛り上がってない
  - Messanger や LINE Bot などのプラットフォームができたことによる一過性のものではないか
  - インターネットやブロックチェーンのようなインパクトはない
- 国内プレイヤー
  - カスタマーサポート用のチャットボット開発
  - チャットボット + 広告（新規ユーザーの獲得を目指すもの）
  - 日本語の自然言語処理に強みを持つ AI 系企業
    - 規模が小さい日本のマーケットに乗り込んでくる海外ベンダーはいない
