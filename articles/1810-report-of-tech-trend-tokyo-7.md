---
title: "Tech Trend Tokyo #7に参加してきました"
emoji: "🏃‍♂️"
type: "tech"
topics: [機械学習, Python, 勉強会]
published: true
published_at: 2018-10-04 12:02
---

# はじめに

https://giginc.connpass.com/event/99291/

Tech Trend Tokyo #7 にメディア・ブログ枠で参加してきました。テーマは「【Python で機械学習】機械学習と言語処理を活用してみる」でした。

# 感想

PyConJP 2018 で「テキストマイニングによる Twitter 個人アカウントの性格推定」というタイトルで LT をしてきましたが、そこでランダムフォレストを使ったので知っている内容が多くあり良い復習になりました！

https://jumpyoshim.hatenablog.com/entry/report-of-pyconjp-2018-lt

GridSearchCV は知らなかったので勉強になりました！活用していきたいです。

早速アウトプットとして、今回のボストンデータセットの練習問題を GitHub にあげました。

私は pyenv や anaconda はトラブルが多くて好きじゃないので Pipenv を用いた環境構築をしています。  
利用される方は Python3.7 と Pipenv をインストールしてください。

https://github.com/jumpyoshim/ttt-7

中島さんのリポジトリはこちらです。

https://github.com/mychaelstyle/seminar

ネットワーキングも素敵な方とお話できましたし、ピザも美味しかったです。ごちそうさまです。ありがとうございました。  
今回お話できなかった方もぜひつながりましょう〜！  
フォローお待ちしております。

https://twitter.com/saka2jp

主催の株式会社 GIG のスキルシェアサービス **Workship**、  
講師の中島さんが CTO を務める IGS 株式会社の SPI、ES に続く第３の新卒採用ツール **Grow360** も面白そうなサービスなので要チェックですね。

https://goworkship.com/

https://grow-360.com/ja/

以下メモ書きです。（メモしきれなかった部分もあります。すみません。）

# メモ書き

## 自己紹介

中島正成さん（IGS 株式会社　執行役員 CTO）  
2011 年、株式会社メタップスの取締役 CTO として立ち上げに参画。機械学習とデータサイエンスのプロダクトインプリメントに取り組む。  
その後、エン・ジャパン株式会社経営戦略室、個人事業での技術アドバイザー、経営アドバイザーを経て、IGS 株式会社に執行役員 CTO としてジョイン。HR 領域、人材評価領域の A.I 活用プロダクト開発と、教育領域への A.I 活用プロダクト開発に取り組む。

## AI といいたくないけど AI の話 part4

- 第六回ある
- 今回はスコア予測の実践

## 回帰と判別

機械学習のアルゴリズムな大きく 2 つに分類できる

- 回帰モデル
  - 単回帰モデル
  - 重回帰モデル
- 判別（分類）モデル

## 教師なし学習

- 出力すべきものがあらかじめ決まっていないという点が教師あり学習と異なる。
- データの背後に存在する本質的な構造を抽出するために用いられる。

## 教師あり学習

- 事前に与えられたデータをいわば例題とみなしてガイドラインにのっとり学習させる。

## 教師あり学習

- 回帰
  - 線形回帰
  - ロジスティック回帰
  - SVM \*
- ツリー
  - 決定木
  - 回帰木
  - ランダムフォレスト \*
  - 勾配ブースティング
- ニューラルネットワーク
  - CNN
  - RNN
  - パーセプトロン \*

## 教師なし学習

- 階層型クラスタリング
- 非階層型クラスタリング
  - K-means
- トピックモデル
  - LDA
- 協調フィルタリング
- 自己組織化マップ

## 練習問題

- よくやるボストンデータセット
- 有名なボストンデータセットを利用して各種パラメータから住宅価格を予測してみよう
- ランダムフォレストを使うよ！楽で優秀
- プロトタイプ作成に有効

## ランダムフォレストとは

- 分類、回帰、クラスタリングに用いられる
- 決定木を弱学習器とする集団学習アルゴリズム

## ボストンデータセットとは

- scikit-learn に含まれている線形回帰などで使用するデータセット
- 米国ボストン市郊外における地域別の住宅価格のデータセット

## 精度を向上するためにはどうすれば良いか

- ハイパーパラメータチューニング
- データクリーニング

## ランダムフォレストのハイパーパラメータ

| ハイパーパラメータ | 説明                                                  |
| ------------------ | ----------------------------------------------------- |
| n_estimators       | バギングに用いる決定木の個数                          |
| max_features       | 最適な分割をするために考慮するフィーチャーの数        |
| max_depth          | 決定木の深さの最大値、過学習対策にこの値の調整が有効  |
| min_samples_split  | ノードを分割するために必要な最少サンプルサイズ        |
| min_samples_leaf   | 葉を構成するのに必要な最小限のサンプルの数            |
| max_leaf_nodes     | 生成される決定木における最大の葉の数                  |
| n_jobs             | フィットおよび予測の際に用いるスレッド数              |
| random_state       | 乱数シード                                            |
| warm_start         | True を設定するとすでにフィットしたモデルに学習を追加 |

## 自動でいい感じのチューニング

- sklearn の GridSearchCV を使う

```python
from sklearn.model_selection import GridSearchCV

params = {'n_estimators': [3, 10, 100, 1000, 10000], 'n_jobs': [-1]}
cv = GridSearchCV(
    RandomForestRegressor(),
    params,
    cv=10,
    scoring='mean_squared_error',
    n_jobs=-1,
    verbose=True
)
warnings.filterwarnings('ignore')
cv.fit(train_data_bs, train_labels_bs)
```

※ 注意

- GridSearchCV を利用する場合は、scikit-learn 以外に numpy, scipy が必要。
- `from sklearn.grid_search import GridSearchCV` だと `no module named 'sklearn.grid_search'` となってしまうため、 `from sklearn.model_selection import GridSearchCV` とする。
