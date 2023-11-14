---
title: "はじめてのPyPIを公開しました"
emoji: "🐍"
type: "tech"
topics: [Python, AWS, Programming]
published: false
published_at: 2018-10-01 08:53
---

# はじめに

PyPI デビューをしましたので、登録したライブラリと PyPI 登録方法を紹介させていただきます。

# aws-sm

https://github.com/saka2jp/aws-sm

Secrets Manager からデータベースの認証情報や APIKey などのアプリケーションの環境変数を安全に取得することができます。

CircleCI でテスト・文法チェック・コードメトリクス計測の CI 構築や、pyup でサードパーティ自動セキュリティアップデートの構築を行いました。

Secrets Manager の説明や利用するシチュエーションなどの詳細は別記事をご覧ください。

https://qiita.com/saka2jp/items/08f216a95509a24c58ec

## インストール

pip で簡単にインストールできます。

```sh
$ pip install aws-sm
```

## 使い方

`SecretsManager` クラスをインポートして、シークレット名と取得したい環境変数名を適切に設定することで環境変数を取得できます。

```python
from aws_sm import SecretsManager

AWS_ACCESS_KEY_ID = ***************
AWS_SECRET_ACCESS_KEY = ***************

secretsmanager = SecretsManager('ap-northeast-1', AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)
secrets = secretsmanager.get_secret_values('tutorials/MyFristTutorialSecret')

USER_NAME = secretsmanager.get_secret_value('USER_NAME', secrets)
PASSWORD = secretsmanager.get_secret_value('PASSWORD', secrets)
```

# PyPI 登録方法

https://packaging.python.org/tutorials/packaging-projects/

公式ドキュメントに PyPI の登録方法がまとまっているのでこの通りに行いましたが、私の環境では一点だけうまくいきませんでした。

それは、TestPyPI や PyPI へのアップロードで利用する twine コマンドを利用する時です。

```sh
$ twine upload dist/*
-bash: twine: command not found
```

ドキュメント通りにコマンドを叩いたのですが、 `twine: command not found` と怒られてしまいました。
検索してみると同じような現象に遭遇している方がいらっしゃいました。

```sh
$ python3 -m twine upload dist/*
```

とすることで正常に動作しました。ちなみに、 `-m` は `-m mod : run library module as a script (terminates option list)` というオプションです。

# おわりに

OSS への貢献は身につく力が大きいなと改めて感じました。はじめてのこととなると失敗も多かったですが、そのトライアンドエラーの工程が自分を大きく成長させてくれたと思います。
PyPI への登録自体はドキュメントのおかげで比較的簡単にできましたので興味のある方はぜひ試してみてはいかがでしょうか。
