---
title: "保守性・可読性の高いPythonコードを実装するためにはどうすればよいか"
emoji: "🐍"
type: "tech"
topics: [Django, Python]
published: false
published_at: 2019-01-08 10:13
---

# はじめに

> コードは理解しやすくなければいけない。

引用：[リーダブルコード ―より良いコードを書くためのシンプルで実践的なテクニック](https://amzn.to/3kFScHi)

コードの保守性や可読性を高めるために我々エンジニアはどんなことができるでしょうか？

- テストを書く
- 推奨されているコードスタイルに準拠する
- コメントを書く
- [DRY 原則](https://xn--97-273ae6a4irb6e2hsoiozc2g4b8082p.com/%E3%82%A8%E3%83%83%E3%82%BB%E3%82%A4/DRY%E5%8E%9F%E5%89%87/)に則る
- 変更・拡張しやすく設計する
- ログを出力する・監視する
- 適切な命名をする
- etc...

まだまだ意識すべきことはあると思いますが、上記の項目はエンジニアであれば恐らく一度は目にしたことがあるような内容であり、暗黙的に了承されたいルールです。
しかし、これらはただの心構えであり、体現するために実際には以下のような項目に落とし込む必要があります。

- カバレッジを計測・可視化する
- コードスタイルを統一する
- 型チェックする
- コードのメトリクスを計測する
- ロガーを設定する
- etc...

本記事ではこれらの項目を Python で実践する方法を紹介させていただきます。

# カバレッジを計測・可視化する

カバレッジを計測することでユニットテストで実行されたプログラムの割合を知ることができます。
カバレッジが 100%であれば、そのプロジェクトにおけるプログラムの全ての行がユニットテストで少なくとも 1 度は実行されているということを意味します。

> 有名なプログラマーたちは「テストされていないコードは、壊れているコードだ」と言っています。

引用：[独学プログラマー](https://amzn.to/3mfUJYT)

テストがされていないということは実際の環境でそのプログラムが実行されるまで期待通りに挙動するか確かではないということです。そのようなプロジェクトに一体誰が変更を加えたいと思うでしょうか？

> カバレッジを 100%にした方がいいということだろうか？したほうがいいじゃない。そうしろといっているんだ。書いたコードはすべてテストしなければいけない。まる。

引用：[Clean Coder](https://amzn.to/3JgewAo)

カバレッジを 100%で維持し続けるというのは現実的な目標だとは思えませんが、カバレッジが高い水準であるということはソースコードの変更に対するエンジニアの心理的なハードルを下げることにつながります。

結果、サードパーティや OS などのバージョンアップを迅速に適用できたり、機能追加を行いやすくなったりします。そのためにまず実行するべきはプロジェクトのカバレッジを計測し現状を認識することです。計測した結果、カバレッジが高い水準とは言えない場合にするべき行動は、テストを書き加え、先にテストを書く文化を作ることです。

> 「テストなんかあとで書けるよ」と言うかもしれない。だが、それは違う。あとで書くことはできないのだ。本当に。あとで書けるテストも少しはあるだろう。注意深く計測していけば、あとでカバレッジ率を高めることもできるだろう。だが、あとで書いたテストは防御なのだ。先に書くテストは攻撃である。あとで書くテストは、コードを書いた人や問題の解決方法を知っている人が書くものだ。こうしたテストは、先に書くテストほど鋭いものではない。

引用：[Clean Coder](https://amzn.to/3JgewAo)

Python でカバレッジを計測するには unittest であれば [Coverage.py](https://github.com/nedbat/coveragepy), pytest であれば [pytest-cov](https://github.com/pytest-dev/pytest-cov) がよく見かける選択肢ではないかと思います。

## Coverage.py

Coverage.py は、Python コードのカバレッジを計測するためのサードパーティです。
テストでコードのどの部分が実行されたのか監視することで、実行されたコードの割合を計測したり実行されていないコードを特定して可視化してくれたりします。

### 1. インストールする

```sh
$ pip install coverage
```

### 2. `coverage run` コマンドでプログラムを実行しデータを集計する

```sh
$ coverage run my_program.py arg1 arg2
```

### 3. `coverage report` コマンドで結果をカバレッジレポートをコンソールに出力する

```sh
$ coverage report -m
Name                      Stmts   Miss  Cover   Missing
-------------------------------------------------------
my_program.py                20      4    80%   33-35, 39
my_other_module.py           56      6    89%   17-23
-------------------------------------------------------
TOTAL                        76     10    87%
```

### 4. `coverage html` でより詳細なカバレッジレポートを HTML で出力する

```sh
$ coverage html
```

`.coveragerc` という設定ファイルにカバレッジを計測しないファイルを指定したり、カバレッジレポートを生成するディレクトリ名を指定したりすることができます。下記はその例です。

```sh
[run]
branch = True
source = .

[report]
ignore_errors = True
omit =
  */migrations/*
  */__init__.py
  */const.py
  */tests.py
  */tests/*
  */apps.py
  */manage.py
  */wsgi.py
  */urls.py
  */settings.py

[html]
directory = coverage
```

## pytest-cov

pytest のプラグインでカバレッジを計測するためのサードパーティです。Coverage.py のオプションはほぼすべてサポートしているようです。

### 1. インストールする

```sh
$ pip install pytest-cov
```

### 2. `--cov` オプションでカバレッジレポートをコンソールに出力する

```sh
$ py.test --cov=myproj tests/
-------------------- coverage: ... ---------------------
Name                 Stmts   Miss  Cover
----------------------------------------
myproj/__init__          2      0   100%
myproj/myproj          257     13    94%
myproj/feature4286      94      7    92%
----------------------------------------
TOTAL                  353     20    94%
```

### 3. `--cov-report=html` オプションでより詳細なカバレッジレポートを HTML で出力する

```sh
$ py.test --cov=myproj --cov-report=html tests/
```

`pytest.ini` というファイルに pytest のオプションを設定できます。毎回指定するようなオプションはこのファイルに設定しておくと便利です。下記はその例です。

```sh
[pytest]
addopts=
  --cov=.
  --cov-report=term-missing
  --cov-config=.coveragerc
```

## カバレッジを可視化する

せっかく計測したカバレッジは可視化しなければ意味がありません。

GitHub であれば CI/CD ツールや[Codecov](https://codecov.io/) のようなサービスをうまく活用することで README にカバレッジバッチを追加するとよいと思います。

![image](/images/hatena/20181229124915.png)

GitLab であれば [Badges](https://docs.gitlab.com/ce/user/project/badges.html) という機能を利用するとカバレッジバッチをプロジェクトのトップページに自動追加してくれます。

![image](/images/hatena/20181229125655.png)

## カバレッジレポートを可視化する

ついでに HTML で出力したカバレッジレポートも可視化しましょう。

https://saka2jp.gitlab.io/django-polls/

上記の URL は HTML で出力されたカバレッジレポートのサンプルです。GitLabPages に出力された HTML をホスティングしています。GitHub であれば GitHubPages を利用するとよいと思います。

このようにカバレッジやカバレッジレポートは誰もがすぐに確認できる状態にしておくとよいでしょう。

## ユニットテストの書き方

今回は Python でのテストの書き方については言及しませんが、わかりやすい参考文献がありますので紹介させていただきます。[DjangoCongress JP 2018](https://djangocongress.jp/) で発表されたセッションで tell-k さんによる「できる！Django でテスト！」です。

https://tell-k.github.io/djangocongressjp2018/#1

また、こちらの資料でも紹介されていますが、aodag memo の「[効果的な unittest - または、callFUT の秘密](http://pelican.aodag.jp/xiao-guo-de-naunittest-mataha-callfutnomi-mi.html)」は個人的にもとても勉強になったブログです。

# コードスタイルを統一する

開発者が同じコーティング規約に準拠していることはとても重要です。統一されたスタイルガイドを利用することはコードの可読性を高めてくれるためです。

Python で推奨されるスタイルガイド、コーティング規約についてが [PEP8](https://www.python.org/dev/peps/pep-0008/) に記載されています。

例えば、インデントの仕方やモジュールのインポート方法まで Python をコーディングする際に推奨されるお作法がまとめられています。有志で日本語に翻訳されたものが公開されています。

https://pep8-ja.readthedocs.io/ja/latest/

また、Python のチュートリアルでも簡潔にまとめられています。

https://docs.python.org/ja/3/tutorial/controlflow.html#intermezzo-coding-style

PEP8 に準拠しているかどうかをチェックしてくれるツールが [flake8](https://gitlab.com/pycqa/flake8) 、インポートのスタイルについてチェックしてくれるのが [isort](https://github.com/timothycrosley/isort) です。

## flake8

`PyFlakes` , `pycodestyle` , `Ned Batchelder's McCabe script` のラッパーで flake8 のコマンドを実行することですべてのツールが実行されます。

### 1. インストールする

```sh
$ pip install flake8
```

### 2. flake8 コマンドを実行する

ファイルまたはディレクトリを指定することができます。

```sh
$ flake8 path/to/code/to/check.py
# or
$ flake8 path/to/code/
```

また、チェックするルール、無視するルールを指定することもできます。

```sh
$ flake8 --select E123,W503 path/to/code/
```

```sh
$ flake8 --ignore E24,W504 path/to/code/
```

flake8 のオプションは`setup.cfg` という設定ファイルに設定することもできます。
以下はその例です。

```sh
[flake8]
exclude = .git,*migrations*,__init__.py
max-line-length = 100
```

## isort

インポートのスタイルは個人によって癖が出やすい部分ではないでしょうか。isort はインポートをアルファベット順にソートし、自動的にセクションに分割してくれるツールです。

### 1. インストールする

```sh
$ pip install isort
```

### 2. isort コマンドを実行する

```sh
$ isort -rc .
```

isort のオプションは flake8 と同じく `setup.cfg` で設定することができます。以下がその例です。

```sh
[isort]
line_length = 100
known_django = django
sections = FUTURE,STDLIB,THIRDPARTY,DJANGO,FIRSTPARTY,LOCALFOLDER
```

また、[flake8-isort](https://github.com/gforcada/flake8-isort) をインストールすると isort で設定したオプション通りにインポートがソートされているかチェックしてくれます。

# 型チェックする

変数の型が明示されているとプログラムの理解を助けてくれたり、デバックを容易にしてくれたりします。

Python3.5 以上であれば型アノテーション（[PEP484](https://www.python.org/dev/peps/pep-0484/)）の構文を利用することが可能です。この型アノテーションをチェックしてくれるのが [mypy](http://mypy-lang.org/) です。

## mypy

Python の型をチェックしてくれる静的ツールです。PEP484 の型アノテーションが適用されているプログラムをチェックすることができます。

### 1. インストールする

```sh
$ pip install mypy
```

### 2. mypy コマンドを実行する

```sh
$ mypy program.py
```

また、型アノテーションを自動生成してくれる [PyAnnotate](https://github.com/dropbox/pyannotate) というサードパーティもあるのでうまく活用すると Python での静的型付け開発を効率的に行えると思います。

https://mypy-lang.blogspot.com/2017/11/dropbox-releases-pyannotate-auto.html

# コードのメトリクスを計測する

コードのメトリクスとはプログラムの複雑度や行数のことです。これらを計測することでプログラムのどの部分に潜在的なリスクがあるかがわかり、リファクタリングの対象が明確にすることができます。

例えば、以下のようなコードはメトリクスを計測することで指摘されます。

- 行数が大きすぎる関数やメソッド、クラス
- 条件分岐が多すぎる制御文
- ネストが深すぎるコード

いずれもとても読みやすいと言えるようなコードではないことは直感的に理解していただけると思います。
コードのメトリクスを計測することでこういったコードをしっかりと把握できる状態にしておき、適切なリファクタリングをすることで読みやすくメンテナンスがしやすいコードにしましょう。

Python プログラムのメトリクスを計測してくれるのが [Radon](https://github.com/rubik/radon) や [Xenon](https://github.com/rubik/xenon) です。

## Radon

Radon はコメント行や空白行、複雑度などさまざまなコードメトリクスを計測することができるツールです。

### 1. インストールする

```sh
$ pip install radon
```

### 2. radon コマンドを実行する

```sh
$ radon cc sympy/solvers/solvers.py -a -nc
sympy/solvers/solvers.py
    F 346:0 solve - F
    F 1093:0 _solve - F
    F 1434:0 _solve_system - F
    F 2647:0 unrad - F
    F 110:0 checksol - F
    F 2238:0 _tsolve - F
    F 2482:0 _invert - F
    F 1862:0 solve_linear_system - E
    F 1781:0 minsolve_linear_system - D
    F 1636:0 solve_linear - D
    F 2382:0 nsolve - C

11 blocks (classes, functions, methods) analyzed.
Average complexity: F (61.0)
```

上記のように実行することで問題があるブロックと平均ランクを出力してくれます。

https://qiita.com/mima_ita/items/5010aaa6808b7290d68d

オプションやランクについては上記の記事がとてもよくまとまっています。

## Xenon

Xenon (キセノン) は Radon をベースとしているコードメトリクスの計測ツールですが、 要求に満たない結果だった場合に失敗（ `non-zero` を出力 ）します。これの何がよいかというと、CI で Xenon を実行するように設定することで強制的にリファクタリングの機会を提供することができます。

コードメトリクスによるより厳密なリファクタリング文化を醸成したいときには Xenon を利用するとよいかもしれません。

### 1. インストールする

```sh
$ pip install xenon
```

### 2. xenon コマンドを実行する

```sh
$ xenon --max-absolute B --max-modules A --max-average A
```

`--max-absolute` はブロックに対する絶対しきい値、 `--max-modules` はモジュールに対するしきい値、 `--max-average` は平均複雑度のしきい値を設定できるオプションです。指定されたランクより低い箇所がひとつでもあった場合、 `non-zero` を返却します。

# ロガーを設定する

障害が発生した際にその原因を素早く追及できることは保守性を高める上では欠かせない要素の一つです。
障害発生の調査をしやすくするためには適切な **ログ出力** と **エラー通知** がされていることが重要です。

エラー通知に関しては `Datadog`や、AWS であれば `CloudWatch` 、GCP であれば `Stackdriver` などの監視ツールを利用する必要があり、Python の責務からは離れているので今回は割愛させていただきます。

Python でログ出力をするために便利なのが [logging](https://docs.python.org/ja/3/howto/logging.html) という Python プログラム内で起きたイベントを記録するための標準ライブラリです。

## logging

logging はあるプログラムが実行されているときに起こったイベントを追跡するための手段です。イベントにはメッセージやロガーレベル、日時などを含めることができます。

logging を利用するにあたって理解しておきたいのは**ロガー**、**ハンドラ**、**フィルタ**、**フォーマッタ**の役割です。

> ロガーは、アプリケーションコードが直接使うインタフェースを公開します。  
> ハンドラは、(ロガーによって生成された) ログ記録を適切な送信先に送ります。  
> フィルタは、どのログ記録を出力するかを決定する、きめ細かい機能を提供します。  
> フォーマッタは、ログ記録が最終的に出力されるレイアウトを指定します。

参考: https://docs.python.org/ja/3/howto/logging.html#advanced-logging-tutorial

### 1. ロガーを作成する

```python
import logging

logger = logging.getLogger('simple_example')
```

### 2. ハンドラを作成する

```python
handler = logging.StreamHandler()
```

### 3. フィルタを作成する

```python
logger.setLevel(logging.INFO)
handler.setLevel(logging.INFO)
```

### 4. フォーマッタを作成してハンドラに追加する

```python
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
```

### 5. ハンドラをロガーに追加する

```python
logger.addHandler(handler)
```

### 6. ログを出力する

```python
logger.debug('debug message') # 何も出力されない
logger.info('info message') # 2019-01-08 12:18:45,965 - simple_example - INFO - info message
logger.warning('warning message') # 2019-01-08 12:18:45,967 - simple_example - WARNING - warning message
logger.error('error message') # 2019-01-08 12:18:45,968 - simple_example - ERROR - error message
logger.critical('critical message') # 2019-01-08 12:18:45,971 - simple_example - CRITICAL - critical message
```

上記の方法以外にもロガー、ハンドラ、フィルタ、フォーマッタの設定方法があります。今回は 1 の方法を紹介しています。

> 1. 上述の設定メソッドを呼び出す Python コードを明示的に使って、ロガー、ハンドラ、そしてフォーマッタを生成する。
> 2. ロギング設定ファイルを作り、それを fileConfig() 関数を使って読み込む。
> 3. 設定情報の辞書を作り、それを dictConfig() 関数に渡す。

参考：https://docs.python.org/ja/3/howto/logging.html#configuring-logging

Django を利用する場合は `LOGGING` という環境変数にロガー、ハンドラ、フィルタ、フォーマッタの設定ができます。以下はその例です。

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'simple': {
            'format': '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        }
    },
    'handlers': {
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
    },
    'loggers': {
        '': {
            'handlers': ['console'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}
```

参考：https://docs.djangoproject.com/ja/2.1/topics/logging/

上記の参考文献は Django 公式ドキュメントのロギングに関するページですが、「[**クイックロギング入門**](https://docs.djangoproject.com/ja/2.1/topics/logging/#a-quick-logging-primer)」という章はロギングに関してとてもわかりやすくまとまっています。Django が知らなくても読める章なのでぜひ一度目を通してみてください。

# おわりに

Coverage.py, pytest-cov, flake8, isort, mypy, Radon, Xenon, logging といったライブラリを活用して保守性・可読性が高い Python コードが実装できないか模索してみました。（Django 成分がやや多めになってしまいました。すみません。）

これらは決して保守性・可読性が高い Python コードを実装するためのベストプラクティスというわけではなく、およそ 2 年間 Python ともがき苦しんできた 1 プログラマの血と涙の結晶だと思っていただければ幸いです。

また、少し触れている章もありましたが、これらは CI で実行することでよりその効果を得ることができます。むしろ CI で実行しなければあまり意味がありません。継続的に実行し続ける仕組みが重要です。

「他にもこういう方法がある」とか「ここは違うんじゃない？」といった意見があればぜひご教授ください！

最後になりましたが、これらのノウハウが習得できたのは優秀なチームメンバーのおかげです。ありがとうございます！
