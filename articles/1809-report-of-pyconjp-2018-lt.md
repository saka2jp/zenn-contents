---
title: "テキストマイニングによるTwitter個人アカウントの性格推定"
emoji: "🐍"
type: "tech"
topics: [Python, 勉強会, 機械学習]
published: false
published_at: 2018-09-17 09:32
---

# はじめに

https://pycon.jp/2018/

PyConJP 2018 で「テキストマイニングによる Twitter 個人アカウントの性格推定」というタイトルで LT をさせていただきました。

普段はサーバーサイドエンジニアとしてスマホアプリのバックエンドサーバー開発を主に Python/Django で行っています。

LT 参加の動機は、Python を使っている身として機械学習に触れてみたかったからです。

Web アプリケーション開発でしか Python を使ったことがありませんでしたが、Python といえばデータサイエンスのイメージがあります。

採択されてもされなくても、PyConJP の LT という機会で普段は知らない Python の一面を掘り下げてみようと思いました。

# スライド

https://speakerdeck.com/saka2jp/personal-character-estimation-with-twitter-pyconjp-2018

# YouTube

[01-105_Day1_Lightning Talk~Closing](https://www.youtube.com/live/bnC79CvKMbY?feature=share&t=1797)

# Gihyo

PyConJP の公式カンファレンスレポートで紹介していただきました。

> 正直「ちょっとストーカーの話っぽいけど大丈夫なの？」とスタッフ同士顔を見合わせる場面もあったのですが，終わってみると真面目でユーモアもある発表だったと思います。

https://gihyo.jp/news/report/01/pyconjp2018/0001?page=3

# Twitter の反応

Twitter で嬉しい反応をたくさんいただきました！ありがとうございます！

https://togetter.com/li/1281224:embed

# Twitter アカウントの性格推定

## ツイートの収集

まずはツイートの収集です。Twitter 社が提供する[Twitter API](https://developer.twitter.com/en.html)を利用してツイートを取得します。

クライアントの実装として、すぐにパッと思い付くのは `requests` を用いた実装です。

https://github.com/requests/requests

`HTTP for Humans` とデカデカと主張しているように、直感的に HTTP の通信を実装できます。

これは `urllib2` の使いにくさを酷くディスった表現ですが、Python3 で `urllib.request` が標準ライブラリとして提供されている今、
こちらを使うのも一つの選択肢ではないかと思います。

`urllib2` と比べ格段に使いやすくなっています。

https://docs.python.jp/3/library/urllib.request.html

例えば、AWS Lambda を Python3 で使う場合は外部ライブラリを import するためには少々手間がかかるため、 `urllib.request` を選択するのが良いと思います。

今回の LT では簡潔に実装できて、GitHub スター数もそこそこあったので `python-twitter` というライブラリを用いることにしました。

https://github.com/bear/python-twitter

```python
import twitter

api = twitter.Api(
    consumer_key=CONSUMER_KEY,
    consumer_secret=CONSUMER_SECRET,
    access_token_key=ACCESS_TOKEN_KEY,
    access_token_secret=ACCESS_TOKEN_SECRET
)

tweets = api.GetUserTimeline(screen_name='saka2jp', count=2)
```

collections の namedtuple のようなデータ構造で返却されるため、注意が必要です。

IN

```python
[tweet.text for tweet in tweets]
```

OUT

```python
['RT @akucchan_world: 吉村さんのLTです。\n#pyconjp https://t.co/ZCvHY9ebcw',
 'PytestのTDD実践ためになった #pyconjp']
```

## 文章のベクトル化

続いて、収集したツイートを数値計算しやすいようにベクトル化します。

ベクトル化の手法として、3 つほど確認できました。

https://tifana.ai/words/natural-language-processing/9302.html

https://deepage.net/bigdata/machine_learning/2016/09/02/word2vec_power_of_word_vector.html

https://deepage.net/machine_learning/2017/01/08/doc2vec.html

Word2Vec や Doc2Vec に関しては、まだ咀嚼しきれない部分があったため、今回は BoW を選択しました。

BoW は、単語の頻出度のみを考慮して、単語の頻出度をベクトル化したものが最も近いデータを推定結果としているだけです。

Word2Vec や Doc2Vec は単語の並び順なども考慮するらしく、推定精度が上がるらしいです。まだまだ勉強不足、今後の課題です。

### 形態素解析

ベクトル化するために、まずは形態素解析をする必要があります。

形態素解析のツールとして、3 つほど確認できましたが、環境構築が容易で速度もはやい MeCab を選択してみました。

IN

```python
import MeCab

tagger = MeCab.Tagger('mecabrc')
data = []
for tweet in tweets:
    node = tagger.parseToNode(tweet.text)
    words = []
    while node:
        meta = node.feature.split(',')
        if meta[0] == '名詞':
            words.append(node.surface.lower())
        node = node.next
    data.append(words)
print(data)
```

OUT

```python
[['rt', '@', 'akucchan', '_', 'world', ':', '吉村', 'さん', 'lt', '#', 'pyconjp', 'https', '://', 't', '.', 'co', '/', 'zcvhy', '9', 'ebcw'], ['pytest', 'tdd', '実践', 'ため', '#', 'pyconjp']]
```

※ 形態素解析の際、実際には URL や@アカウント名、リツイートなどを正規表現で空文字で置換することで、形態素解析された際に意味をなさない単語を排除します。

MeCab と Janome に関しては pip でインストールできるため利用しやすいですね。

JUMAN は環境構築がやや手間ですが、かなり高精度の形態素解析をしてくれるみたいです。

https://pypi.org/project/mecab-python3/

https://pypi.org/project/Janome/

http://nlp.ist.i.kyoto-u.ac.jp/index.php

### 特徴ベクトル

形態素解析した単語群は `gensim` を用いて辞書を作成し、ベクトル化します。

```python
from gensim import corpora, matutils

dictionary = corpora.Dictionary(data)
data_train = []
for datum in data:
    bow = dictionary.doc2bow(datum)
    dense = list(matutils.corpus2dense([bow], num_terms=len(dictionary)).T[0])
    data_train.append(dense)
print(data_train)
```

OUT

```python
[[1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [1.0, 1.0, 0.0, 0.0, 0.0, 0.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0]]
```

https://github.com/RaRe-Technologies/gensim

## 文書分類器

ここまででデータを揃えることができたので、正解ラベルを用意して学習を行います。

正解ラベルはエゴグラムの診断結果を利用します。Twitter 上にエゴグラムの診断結果をツイートするアカウントが多く存在するためです。

以下の 23 パターンがあります。

- ネクラ厭世タイプ（Ｗ型）
- 明朗楽観タイプ（Ｍ型）
- 優柔不断タイプ（Ｎ型）
- ハイパワータイプ（逆Ｎ型）
- 頭でっかちタイプ（逆Ｖ型）
- お手あげタイプ（Ｖ型）
- 典型的ネクラタイプ（Ｕ型）
- ぼんぼんタイプ（逆Ｕ型）
- 頑固オヤジタイプ（左上がり型）
- ガキ丸出しタイプ（右上がり型）
- ハイレベルタイプ（オールＡ型）
- 中庸タイプ（オールＢ型）
- 原始人タイプ（オールＣ型）
- ルーズタイプ（ＣＰ欠乏型）
- クールタイプ（ＮＰ欠乏型）
- 現実無視タイプ（Ａ欠乏型）
- 自閉症タイプ（ＦＣ欠乏型）
- 気ままタイプ（ＡＣ欠乏型）
- 口うるさタイプ（ＣＰ型）
- お人好しタイプ（ＮＰ型）
- コンピュータタイプ（Ａ型）
- 自由奔放タイプ（ＦＣ型）
- 自己卑下タイプ（ＡＣ型）

### 機械学習

`scikit-learn` を利用することで簡単に機械学習ができます。

IN

```python
from sklearn.ensemble import RandomForestClassifier

clf = RandomForestClassifier()
label_train = ['ネクラ厭世タイプ（Ｗ型）', '明朗楽観タイプ（Ｍ型）']
clf.fit(data_train, label_train)
clf.predict(data_train)
```

OUT

```python
array(['ネクラ厭世タイプ（Ｗ型）', '明朗楽観タイプ（Ｍ型）'], dtype='<U12')
```

学習させたデータを推定してみると、当然同じタイプが推定されることが確認できます。

https://github.com/scikit-learn/scikit-learn

# 感想

たとえ数学的な知識に疎くても、gensim や scikit-learn といったライブラリを利用すれば簡単に実現できてしまうのが Python のすごいところだと感じました。

今後は数学的な知識をより深めたり、機械学習を Web アプリケーションに組み込んだりしてみたいです。今後も Python を使っていろいろなチャレンジをしていきたいです。

# おわりに

今回は PyConJP ということで Python の `gensim` や `scilkit-learn` などのライブラリを使って文書分類に挑戦してみましたが、Python にとらわれなければ他の選択肢が考えられました。

Facebook が開発している OSS の fastText や、Google のサービスである Google Natural Language API、AWS のサービスである Amazon Comprehend などを利用するともっと簡単にテキスト分析ができるかもしれません。

https://fasttext.cc/

https://cloud.google.com/natural-language/?hl=ja

https://aws.amazon.com/jp/comprehend/

# 参考文献

以下の記事を大変参考にさせていただきました。深く感謝いたします。

- https://qiita.com/yasunori/items/31a23eb259482e4824e2
- https://qiita.com/3000manJPY/items/a0652d488ce3c956613d
