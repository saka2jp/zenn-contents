---
title: "DjangoCongress JP 2018 に参加してきました"
emoji: "🏃‍♂️"
type: "tech"
topics: [勉強会, Django]
published: false
published_at: 2018-05-20 05:57
---

# はじめに

https://djangocongress.jp/

DjangoCongress JP 2018 Day1 カンファレンスデーと Day2 スプリントデー両日に参加してきました。

五月病の怠惰を吹っ飛ばしてくれた最高のカンファレンスでした。毎年この時期にやってほしいですね（笑）

参加動機としては、Day1 は会社の先輩社員の方が登壇予定だったためで、Day2 は OSS へのコントリビュートに興味があったためです。

Day1, Day2 に参加した感想をそれぞれまとめておきます。

# Day1 カンファレンスデー

カンファレンスの内容に関しては、djangocongress.jp にセッション概要やスライドのリンクがあります。

また、DjangoCongress JP 2018 の参加者の方は優秀な方が多くて、Day 1 終了後 2 時間もしないうちにいくつも参加レポートが投稿されていたので、スライドの紹介やセッションごとの感想は今回は割愛します。Twitter のタイムラインで見逃してしまった方はぜひ確認してみてください。

http://blog.mtb-production.info/entry/2018/05/19/185344

http://katekichi.hatenablog.com/entry/2018/05/19/Django_Congress_JP_2018%E3%81%AB%E5%8F%82%E5%8A%A0%E3%81%97%E3%81%A6%E3%81%8D%E3%81%9F

http://nikkie-ftnext.hatenablog.com/entry/2018/05/19/230452

https://xaro.hatenablog.jp/entry/2018/05/19/183000

http://thinkami.hatenablog.com/entry/2018/05/20/131518

DjangoCongress に参加してみていくつか収穫を得ることができました。

1. 業務に活かせそうな内容が数多くあった
1. Django の最新の動向を探ることができた
1. もっと勉強しなくてはという刺激になった
1. 良い参考文献をたくさん知ることができた

これが 1000 円で得られたのですごく満足度の高いカンファレンスでした。

実は私は、Hayao Suzuki さんが発表されていた「レガシーアプリケーションの現代化」というセッションで紹介されている、架空のプロジェクトの 4 人のメンバーのうちの 1 人です。

新卒 2 年目の私にとっては勉強させていただくことばかりでしたが、一方で、レガシーアプリケーションを触るのはモチベーション的に辛いものがあります。エンジニアであればやはり新しい言語やフレームワークを用いて 1 から作りたいですよね。

幸いにも最近新しい案件が舞い込んできているので、レガシーアプリケーションの現代化での経験や DjangoCongress で学んだことを活かして新規案件に取り組みたいです。

# Day2 スプリントデー

OSS へのコントリビュートに興味があったので、参加を決意しました。スプリントは未経験で Django 歴も 1 年と浅かったので不安な気持ちが大きかったですが、わかりやすくてよくまとまっているガイダンスがあったおかげで作業を進めることができました。

ガイダンスの一部

## Contributing to Django Core

1. Done the tutorial.  
   https://docs.djangoproject.com/ja/2.0/intro/tutorial01/
1. Done Contributing tutorial.  
   https://docs.djangoproject.com/ja/2.0/intro/contributing/
1. See Dashboad, Find nice issue and break it.  
   (Don't share this URL to SNS)

## Translating Django Docs

1. 翻訳ガイドからユーザー登録をする  
   http://djangoproject.jp/translate/
1. Transifex 上で翻訳
   - Intro や topics 内の翻訳がおすすめ
   - 興味のある＆よく読むところからどうぞ
   - 他の人の翻訳をレビューするのも OK

## Translating DjangoGrilsDocs

1. 翻訳ガイドを見て進める  
   https://bit.ly/2rWD1tE

Django2.0 は触ったことがなく Django チュートリアルもがっつりやったことがなかったので、いい機会だと思いチュートリアルを一からやってみました。

Django1.11 からの大きな変更点である `django.urls.path()` を用いたよりシンプルな URL ルーティング構文以外は知っていることが多かったのでやる必要はなかったかもしれません。

そのあと、[「Django への初めてのパッチを書く」](https://docs.djangoproject.com/ja/2.0/intro/contributing/)というチュートリアルを行いました。
django/django をフォークして過去のパッチを真似てプルリクを出すというチュートリアルです。

初めて django/django のテストを読んだのですが、1 つのテストケースメソッドにいくつものアサーションがあって、心が少しざわざわしました。（[ルール: 各テストケースメソッドは、 1 つのことだけをテストする](http://docs.pylonsproject.jp/en/latest/community/testing.html#id5) : Day1 の tell-k さんによる 「できる！Django でテスト!」でも紹介されています。気になる方は djangocongress.jp からチェック！）

機能追加などのより本格的なソースコードへのコントリビュートをするためには、もっと素振りが必要だと感じました。

一方で、今のレベルでも翻訳やタイポの修正などコントリビュートできることはあるので、積極的に活動していきたいですね。

以下の 2 つが、記念すべき人生初コントリビュートです。

![image](20180520202350.png)

![image](20180520202350.png)
