---
title: "DjangoのINSTALLED_APPSの順番がめちゃくちゃ重要だった話"
emoji: "🎸"
type: "tech"
topics: [Django, Python]
published: true
published_at: 2019-01-18 06:06
---

# TL;DR

- `INSTALLED_APPS` は最初にリストされているアプリケーションのソースコードが優先される
- `INSTALLED_APPS` は **ローカルアプリケーション**・**サードパーティアプリケーション**・**Django アプリケーション** という順番にしたほうがよい

```python
INSTALLED_APPS = [
    # Local apps
    'polls.apps.PollsConfig',

    # Third party apps
    'rest_framework',

    # Django apps
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
]
```

- 上記の順番にするとテンプレートや静的ファイル、マネジメントコマンドを上書きする変更が反映される

# INSTALLED_APPS

Django の環境変数に `INSTALLED_APPS` という環境変数があります。

これは Django インスタンスの中で有効化されているすべての Django アプリケーションの名前を保持するリストの環境変数です。

`django-admin startproject` で作成された Django アプリケーションの `INSTALLED_APPS` はデフォルトで以下のアプリケーションを保持しています。

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

この `INSTALLED_APPS` を編集するタイミングが主に 2 パターンあると思います。ひとつは `django-admin startapp` でローカルアプリケーションを追加した場合。もうひとつはサードパーティのライブラリをインストールした場合です。

`django-admin startapp` でローカルアプリケーションを作成したら、そのアプリケーションおよびモデルを有効化するために `INSTALLED_APPS` に追加する必要があります。例えば、 `polls` というローカルアプリケーションを作成した場合以下のように追加します。

```python
INSTALLED_APPS = [
    ...
    'polls.apps.PollsConfig',
    ...
]
```

ちなみに、単に `polls` と追加するのはタブーです。

> Your code should never access INSTALLED_APPS directly. Use django.apps.apps instead.

FYI: https://docs.djangoproject.com/en/2.1/ref/settings/#installed-apps

また、サードパーティのライブラリをインストールしたら `INSTALLED_APPS` にアプリケーションを追加する必要がある場合があります。例えば [Django REST Framework](https://github.com/encode/django-rest-framework)は以下のように `rest_framework` を追加する必要があります。

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    ...
]
```

# INSTALLED_APPS の順番

`INSTALLED_APPS` に関する順番に関しては特に公式ドキュメントには記載されていません。  
また、さまざまなサードパーティの `README` やドキュメント、Django に関する記事を見てきましたが本当にバラバラで多種多様です。

私自身も 1 週間前までは特に順番を意識することなく、Django アプリケーション・サードパーティアプリケーション・ローカルアプリケーションの順番でなんとなくまとめているという状況でした。

順番が重要だと気付いたのは、Django 管理画面の既存のテンプレートや静的ファイルをカスタマイズしたい・上書きしたいという状況の時でした。ドキュメント通りに変更を加えたつもりが一切変更は反映されなかったのです。

このときはじめて `INSTALLED_APPS` に関するドキュメントを読んで、**最初にリストされるアプリケーションのリソース（テンプレートや静的ファイル、マネジメントコマンド、翻訳）が優先される** ということを知りました。

> When several applications provide different versions of the same resource (template, static file, management command, translation), the application listed first in INSTALLED_APPS has precedence.

FYI: https://docs.djangoproject.com/en/2.1/ref/settings/#installed-apps

上記の仕様があるため、サードパーティや Django の既存のリソースを上書きするような場合、上書きしたリソースがきちんと反映されるためにはそのリソースが含まれるアプリケーション（ローカルアプリケーションであることがほとんどだと思いますが）が上書きしたいリソースのアプリケーション（サードパーティや Django）より優先されていなければなりません。

つまり、サードパーティや Django の既存のリソースを上書きするような場合、以下のような順番にする必要があります。

```python
INSTALLED_APPS = [
    # Local apps
    'polls.apps.PollsConfig',

    # Third party apps
    'rest_framework',

    # Django apps
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
]
```

最も反映させたいのは言わずもがなローカルアプリケーションです。次にサードパーティアプリケーションです。サードパーティが Django のリソースを上書きする場合があるためです。

カスタマイズは必要なくて上書きもしない場合は特にこれを意識する必要はありませんが、私のようにいざそのシチュエーションがきたとき、 `INSTALLED_APPS` が適切な順番でなかったら変更が反映されず混乱することになります。エラーが何か出力されるわけではないので原因の特定にもかなり苦労しました。

そんなことがないように `INSTALLED_APPS` の順番には意味があることをぜひ知っておいていただきたいです。余裕があれば自分のプロジェクトの `INSTALLED_APPS` の順番を見直してみてください。
