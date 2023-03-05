---
title: "ES(ECMAScript)の基礎を学ぶハンズオンに参加してきました"
emoji: "🏃‍♂️"
type: "tech"
topics: [勉強会, JavaScript]
published: true
published_at: 2018-11-28 02:45
---

# はじめに

https://js-builders.connpass.com/event/105770/

JavaScript Builders さん主催の「【初心者向け】JavaScript の次のステップ、ES(ECMAScript)の基礎を学ぶハンズオン」にブログ枠で参加してきました。

今回参加した動機は **Vue や React を学ぶ前に ES をちゃんと抑えておきたかったから** です。

普段の業務としては Python/Django で API の実装がメインでたまにインフラ構築といった感じで、JS はもちろん ES とはあまり縁がありません。

しかし、将来的にはバックエンド、インフラ、フロントエンドをカバーできるようなエンジニアになりたいので今回の参加に至りました。

> 今回のイベントでは、JavaScript を Progate や独学で学んだけど、その先の ECMAScript がわからない。
> React.js or Vue.js を触り始めたけど、実は ECMAScript がわからない。 そんな方が基礎を学べる機会になる為のハンズオンを行います。

まさにぴったりの勉強会ですね！

# 感想

![tweet](https://twitter.com/rolotokyo/status/1067007422135394305)

今回の勉強会のゴールが設定されていたのでそれに沿って振り返りたいと思います。

## 明日からドヤ顔で ES の歴史を語れるようになる事

まず、ECMAScript を一言で説明すると、

**JavaScript と JScript の仕様差異を一般化するために ECMA International という機関が JavaScript を標準仕様化したものが ECMAScript です。**

(　-`ω-)どや！

講師の村瀬さんが言語の成り立ちから今までバージョン遷移まで詳しく教えてくださりました。

ES は毎年 6 月にメジャーアップデートがあるかなり開発がさかんな言語みたいで、バージョンごとの豆知識も教えていただきました。

- ES6：主流なバージョン
- ES7：主要な 2 機能がリリースされたが、どっちも使わなくていいらしい
- ES2017：Async Await を使うためにバージョンアップする場合もあるんだとか
- ES2018：最近リリースされたばかりなので実績が少なく使用者は少ない

とりあえず ES6 を抑えておくのが基本らしいです。

この記事もわかりやすかったです。

https://qiita.com/naoki_mochizuki/items/cc6ef57d35ba6a69117f

JavaScript を始めるならまずは読んでおくべきサイトらしいです。

https://developer.mozilla.org/ja/docs/Web/JavaScript

ECMAScript についてはこちら。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Language_Resources

## ES の基礎がわかるようになる事

ES の基礎を理解するために JS のハンズオンで JS の基礎を理解してから ES のハンズオンで ES を理解するという流れでした。

**JS はここが使いづらかったから ES ではこう改善された** のようにストーリー仕立てで覚えることができてかなり理解が捗りました。

普段 Python を書いている身としては、JS の基本仕様にはかなり驚かされることが多かったです。
class 構文がなかったり const がなかったりモジュールのインポートができなかったり...

ここらへんが ES のアップデートでプログラミング言語としての基本仕様を満たしてきたんだなと感じました。
また、ES の構文はかなり書き方に柔軟性があるなと思ったのと、ワイルドカードインポートが認められているのが少し驚きましたね。

為藤さんのハンズオンの説明やサンプルがすごくわかりやすかったので、ES はほとんど触ったことありませんでしたがその場で理解できました。

ハンズオンのソースコードは JavaScript Builders さんの Slack チャンネルで公開されているので気になる方はコンパスのページからとんで確認してみてください。

あと、ハンズオンは PlayCode を使ってやったのですがすごい便利でした。

https://playcode.io/

# おわりに

2 時間半とは思えないほど凝縮された内容で、これが無料で受けられたので運営の方々には感謝しかありません。

会場提供してくださったナンバーナインさんにも感謝です！

https://no9.co.jp/

次のステップとして React か Vue を頑張っていきたいです。

ちなみに次回の勉強会にもブログ枠として参加予定なのでお願いしますー！

https://js-builders.connpass.com/event/105769/
