---
title: "「Clean Coder」から学んだこと。"
emoji: "📖"
type: "idea"
topics: [書評]
published: true
published_at: 2018-03-28 10:25
---

# 概要

- 著作： **「[Clean Coder](https://amzn.to/41JWkq6)」**
- 著者： **Robert C. Martin**
- 訳： **角 征典**
- 著者経歴：
  　 Object Mentor,Inc.の創業者であり代表。Object Mentor 社は、オブジェクト指向ソフトウェア設計のコンサルティングやトレーニング・スキル開発を行う。

# 要約

タイトルに「Clean Coder」とありますが、「リーダブルコード」のような関数や変数の命名規則や美しいフォーマットに関する内容ではありません。本書は、見積もりや時間管理などのソフトウェア開発のプロとしての振る舞いについて記述が大部分を占めています。また、品質が高いコードを保つために TDD を強く推奨しているのも特徴です。

# 面白かった章とその理由

**第 6 章 練習**

ボウリングゲーム、素因数分解、ワードラップなどのプログラミング練習方法を知ることができ、習慣として取り入れるべきだと感じました。

理由としては、細かい単位でアウトプットができるためです。エンジニアとしてのアウトプット方法として、ブログや記事の投稿、自作プロダクトのローンチは意識していましたが、それらはアウトプットの単位が大きいということをこの章を読んで感じました。毎日の習慣とするために、15 分から 30 分くらいで負担にならない程度のアウトプットを取り入れたいと思います。

ちょっと調べてみた感じだと、Paiza が良さそうな気がしました。

# Robert C. Martin が考える「プロ」まとめ

本書を読んでいて感じたのは「プロ」という単語がよく出てくるなということです。要約でも触れましたが、本書は Robert C. Martin が考えるソフトウェアエンジニアのプロとしての振る舞いが綴られています。これらの中で身につけておきたいと感じた考え方や実践していきたいことをまとめました。以下は全て引用です。

> 間違いが避けられないとしても、それでも責任をとるのがプロだ。したがって、プロ意識を高めるために最初に心がけることは、謝罪だ。

> 計算してみよう。1 週間は 168 時間だ。雇用主に 40 時間、君のキャリアに 20 時間を使える。残りは 108 時間だ。睡眠に 56 時間使うとして、残りの 52 時間は自由に使える。そんなことはしたくないと思うかもしれない。それでもいい。だが、それならプロとは言わないでもらいたい。プロは自分の専門知識の手入れに時間を書けるものだ。

> 新人を職場に慣れさせるためには、横に座ってやり方を見せてあげるのがいい。プロには若手を指導する義務がある。決して放置してはいけない

> プロは頼まれたことすべてに「イエス」と言う必要はない。しかし、「イエス」と言えるような創造的な方法を懸命に探さなければいけない。

> 結局のところ、気が散ったり時間を失ったりするような割り込みは避けられないものだ。そのようなときは、次は自分が誰かの時間に割り込むかもしれないと考えよう。プロの態度は、親切に快く手伝ってあげることだ。

> 医師はミスをした手術を再度やってみたいと思わない。弁護士はミスをした訴訟を再度やってみたいとは思わない。ミスが多いとプロとは見なされないからだ。同様に、バグの多いソフトウェア開発者はプロではない。

> プログラマが陥る行動のなかで最もプロとしてふさわしくないのは、作業が終わっていないのに終わったとウソをつくことだろう。これはあからさまなウソだ。最悪である。

> プロは練習をする。

> プロであれば、チームで最高のソフトウェアを作る。

> プロの開発チームに必要なのは、優れたテスト戦略だ。

> 受動的攻撃性のある人もいる。議論の終わりに同意したと見せかけて、解決策に従事しないなどの妨害行為をするのだ。そして、「これは彼らが望んだやり方だ」などと言う。プロらしからぬ最悪の振る舞いだ。決してこんなことをしてはいけない。同意したならば、従事しなければいけない。

> プロの開発者は、睡眠のスケジュールを管理して、翌朝までに集中力のマナを回復している。

> プロは 1 つの考えに固執しない。他の考えを広く受け入れて、行き詰まったら他の選択肢を使う。

> プロは、自分ができるとわかるまでコミットメントしない。簡単なことだ。できるかどうかわからないことにコミットメントを求められたら、断ればいいのだ。

> プロのソフトウェア開発者は、ビジネスが計画を立てるのに使えるような実用的な見積もりを提供する方法を知っている。守れないような約束はしない。果たせないようなコミットメントはしない。

> プロがコミットメントするときには、具体的な数値を挙げて、それを達成する。

> プロは一緒に働く。

# 仕事に活かせそうな知識、活かせそうな状況と活かし方

- PERT や広域デルファイ法などの見積もり術を身につける。
- TDD 原則に従った開発をする。
- コードの設計の変曲点だと感じた時はすぐに取り掛かる。後回しにもできるが今変更するよりも絶対にキツくなる。
- 精神力を向上させるために定期的な運動をする。