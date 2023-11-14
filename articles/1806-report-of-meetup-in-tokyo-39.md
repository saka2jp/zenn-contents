---
title: "Meetup in Tokyo #39 Testing & Engineering に参加してきました"
emoji: "🏃‍♂️"
type: "tech"
topics: [勉強会, テスト, QA]
published: false
published_at: 2018-06-28 07:40
---

# はじめに

https://line.connpass.com/event/91423/

「Meetup in Tokyo #39 Testing & Engineering」に参加してきました。

普段サーバーサイドエンジニアとして働いており、テストに関しては CI でユニットテストを走らせるということくらいしかしていませんでしたが、ユニットテスト以外の自動化方法や、組織全体がテストに取り組む意義を理解し戦略的にテストを行うことの重要性を今回の Meetup で勉強させていただきました。

LINE ブログが更新されたので転載してさせていただきます。

https://engineering.linecorp.com/ja/blog/detail/316

# イマドキのソフトウェアのテストや QA の考え方

https://www.slideshare.net/YasuharuNishi/line-developer-meetup-in-tokyo-39-presentation

## 感想

具体的なアンチパターン事例や数々の解決手法、戦略を知ることができました。世の中には本当に色々な会社があるのですね。（白目）

1 時間には収まりきらないボリュームだったのでもっと聞きたかったです。松尾さん（@Kazu_cocoa）へのただならぬ愛を感じました。

## 自分なりまとめ

**品質保証やテストは組織を賢くするためにあらゆることをする最高にやりがいのあるクリエイティブな仕事である**

労働のスケールアップを保証する活動は概ね失敗するため、QA の本質を理解し知のスケールアップを保証する活動を組織的に行うことが重要である。

無理ゲーに重課金するのも、ネットで検索した新手法を導入するのも間違い。何かやればやった分だけ成功するかもしれないが根付かない。

AQUA 的観点で PJ ごとに戦略を立てよう。

## 勉強すること

- [PMBOK](https://wa3.i-3-i.info/word13106.html)
- [TQM](https://www.juse.or.jp/tqm/about/)
- [SaPID](http://www.juse-sqip.jp/juse_seminar/sapid_2015.html)
- [VSTeP](https://qiita.com/ToshiManaPlus1/items/11181b16a7a17ef03c9f)
- [SPLIT](http://home.kingsoft.jp/news/app/logmi/282805.html)

# LINE の UI 自動テスト事例

https://www.slideshare.net/linecorp/line-ui

## 感想

LINE Fukuoka の E2E UI テストの具体的な実践内容を知ることができました。「地味なことを地道にやることが大切」「開発チームが仕事をしやすくならなければ意味がない」という大園さんの発言がとても印象的で納得感がありました。

「Ayachan」というレポートツールを作成されたとのことでしたが、既存ツールのどこがマッチングしなくてツール作成に至ったかという話をもう少し詳しく聞きたかったですね。

大園さんも伊藤さんも「利益」というワードが出ていて LINE は素晴らしいチームだなと感じました。

## 勉強すること

- [JUnit](https://junit.org/junit5/)
- [Maven](https://maven.apache.org/)
- [Selenide](http://selenide.org/)
- [spring](https://spring.io/)

# An Agile Way As an SET at LINE

https://www.slideshare.net/linecorp/an-agile-way-as-an-set-at-line-103112825

## 感想

伊藤さんの実行力とそれを受け入れる LINE の柔軟性やスピード感を感じざるを得ないプレゼンでした。また、SET にはテスト自動化やアジャイルに関する圧倒的知識・経験と、社内調整力が求められることを知り、すごくやりがいがある仕事だと思いました。

「ビジネスに貢献する」ということを常に念頭に起き続けることは、SET だけではなくサーバーサイドエンジニアやインフラエンジニア、全ての職種に言えることであると感じたので、それについて考え続けていきたいですね。

技術的な知識欲を刺激して、デベロッパーが自発的にテストを書くようになったというお話は感動しました。「インパクトを与える」という観点を勉強させていただきました。

## 自分なりのまとめ

**アジャイルの知識・経験を活用し、ビジネスに貢献しよう！**

ビジネスの 3KPIs「売上・利益・従業員満足度」を判断基準として、課題の解決策を提示し、マネージャーに継続的合意形成をとる。合意形成をしやすくするために、定期的な（動作する）成果物を提出する。

## 勉強すること

- [sonarqube](https://www.sonarqube.org/)
- [MTTR](https://www.dospara.co.jp/5info/cts_str_pc_mtbf)
- [アジャイル宣言の背後にある原則](http://agilemanifesto.org/iso/ja/principles.html)
- [カオスエンジニアリング](https://ityarou.com/oshiete0001/)

# おわりに

帰りに「LINE_ENGINEERRING HAND-BOOK」を貰ってきました。

> LINE のエンジニアになるということは、技術力の高い仲間と共に働くこと、プロジェクトや機能に対しオーナーシップを持って世界中のユーザーに日々素敵な価値を届けることを意味します。

LINE かっこええ。
