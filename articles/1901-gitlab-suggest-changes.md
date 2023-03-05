---
title: "GitLab 11.6の新機能「Suggest Changes」が便利なのでオススメ"
emoji: "🦊"
type: "tech"
topics: [GitLab]
published: true
published_at: 2019-01-22 08:59
---

「Suggest Changes」は一言でいうと、レビューと修正がブラウザで完結しちゃう機能です。具体的には、

1. レビュワーはソースコードに対して変更の提案ができる
1. デベロッパーは提案を Apply できる

ということができます。GitHub でも[同じような機能がリリースされ](https://help.github.com/articles/incorporating-feedback-in-your-pull-request/#applying-a-suggested-change)ています。

![image](/images/hatena/20190122200050.png)

使い方は `insert suggestion` というボタンを押して修正したいソースコードに書き換えるだけです。

![image](/images/hatena/20190122200107.png)

![image](/images/hatena/20190122200124.png)

詳細は以下のドキュメントをご確認ください。

https://gitlab.ssl.iridge.jp/help/user/discussions/index.md#suggest-changes

タイポなどの細かい修正を提案したり複数の修正を提案したりする時に便利ですが、提案したコードでパイプラインが失敗したらかっこ悪いので用法用量にはお気をつけください。

あと、既知のバグかはわかりませんがハイライトのテーマがブラック系（特に Monokai）だと該当コードが見えにくいという罠があるので注意してください。

![image](/images/hatena/20190122200544.png)

![image](/images/hatena/20190122200558.png)

追記：  
上記のバグですが、11.8 で修正されるみたいです。 tnir さんにご指摘いただきました。

![tweet](https://twitter.com/tn961ir/status/1088321239356125190)
