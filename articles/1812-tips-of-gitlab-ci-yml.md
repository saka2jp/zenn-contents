---
title: ".gitlab-ci.ymlの俺的Tips"
emoji: "🦊"
type: "tech"
topics: [GitLab, DevOps, CI/CD]
published: false
published_at: 2018-12-09 12:48
---

# はじめに

GitLab を使いはじめて 1 年半になりますが、`.gitlab-ci.yml` に関するノウハウがほどほどに溜まってきた気がするので Tips としてまとめてみました。

**注意**

- Web アプリケーションの CI/CD に関する Tips が多いです。
- `.gitlab-ci.yml` の例は Python/Django で書いていますが、特に難しいことは書いていないつもりです。適宜得意な言語に置き換えて読んでください。
- 本記事では[公式ドキュメント](https://docs.gitlab.com/ce/ci/yaml/)に倣い、 `.gitlab-ci.yml` の一つの実行単位を `ジョブ`、ジョブに定義する `image` や `stage`, `script` などのジョブのふるまいを `キーワード` と呼びます。

# Index

1. [Services](https://docs.gitlab.com/ee/ci/services/README.html)：コンテナイメージを複数扱う
2. [Anchors](https://docs.gitlab.com/ee/ci/yaml/README.html#anchors)：ジョブのテンプレートでスリムな YAML に
3. [Predefined environment variables](https://docs.gitlab.com/ee/ci/variables/README.html#predefined-environment-variables)：用意されている GitLab CI/CD 変数を活用する
4. [Only and Except](https://docs.gitlab.com/ce/ci/yaml/#onlyexcept-basic)：制限付きのジョブを作成する
5. [when:manual](https://docs.gitlab.com/ee/ci/yaml/README.html#whenmanual)：安心安全なデプロイ
6. [artifacts](https://docs.gitlab.com/ee/user/project/pipelines/job_artifacts.html)：カバレッジレポートを可視化する

# Tips

## 1. Services：コンテナイメージを複数扱う

https://docs.gitlab.com/ee/ci/services/README.html

`services` というキーワードを用いると、ベースイメージと接続可能なコンテナを定義できます。
Docker in Docker でコンテナレジストリに Docker イメージをプッシュしたり、単体テストでデータベースコンテナに接続したりするといったケースに有用です。

例えば、下記のように定義することで `postgres` というホスト名で PostgreSQL に接続できます。

```yaml
services:
  - postgres:latest
variables:
  POSTGRES_DB: custom_db
  POSTGRES_USER: custom_user
  POSTGRES_PASSWORD: custom_password
```

PostgreSQL の公式イメージで用意されている環境変数にデータベース名、ユーザ名、パスワードを設定します。

環境変数に関する詳細は PostgreSQL の DockerHub を参照してください。  
FYI: https://hub.docker.com/_/postgres/

`DATABASE_URL` で接続情報を表すと、 `postgres://$POSTGRES_USER:$POSTGRES_PASSWORD@postgres:5432/$POSTGRES_DB` となります。

## 2. Anchors：ジョブのテンプレートでスリムな YAML に

https://docs.gitlab.com/ee/ci/yaml/README.html#anchors

Anchors という機能を利用してジョブのテンプレートを作成することで、`.gitlab-ci.yml` をスリムに保つことができます。

例えば、test1, test2 というテストをそれぞれ実行したいとき、

```yaml
test1:
  stage: test
  image: python:latest
  before_script:
    - pipenv install --dev --system
  script:
    - pytest test1

test2:
  stage: test
  image: python:latest
  before_script:
    - pipenv install --dev --system
  script:
    - pytest test2
```

上記のように書くこともできますが、Anchors を利用すると下記のように書くことができます。

```yaml
.test_template: &test_definition
  stage: test
  image: python:latest
  before_script:
    - pipenv install --dev --system

test1:
  <<: *test_definition
  script:
    - pytest test1

test2:
  <<: *test_definition
  script:
    - pytest test2
```

今回の例では行数的な差があまり出ませんでしたが、ジョブが増えれば増えるほど Anchors の効果が高まります。

## 3. Predefined environment variables：用意されている GitLab CI/CD 変数を活用する

https://docs.gitlab.com/ee/ci/variables/README.html#predefined-environment-variables

`.gitlab-ci.yml` には GitLab があらかじめ用意している環境変数がいくつかあります。
それらを活用することで余計な環境変数定義をしなくて済んだり、別のプロジェクトでそのまま活用できたりします。

例えば、GitLab コンテナレジストリへのイメージのプッシュで Predefined environment variables を活用することができます。

```yaml
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY
  script:
    - docker build --pull -t $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
  only:
    - tags
```

`CI_JOB_TOKEN`, `CI_REGISTRY`, `CI_REGISTRY_IMAGE`, `CI_COMMIT_TAG` は Predefined variables です。

## 4. Only and Except：制限付きのジョブを作成する

https://docs.gitlab.com/ee/ci/yaml/README.html#onlyexcept-basic

`only` や `except` というキーワードを定義すると、ある条件の場合のみジョブを実行したり、ある条件を除く場合のみジョブを実行したりすることができます。

例えば、タグがプッシュされた時だけ実行するように定義したり、

```yaml
only:
  - tags
```

ある特定のブランチがプッシュされた時は実行をしないように定義したりすることができます。

```yaml
except:
  - master
```

また、正規表現にも対応しているので特定のプレフィックスがついたブランチがプッシュされた時だけ実行するというような定義も可能です。

```yaml
only:
  - /^release-.*$/
```

## 5. when:manual：安心安全なデプロイ

https://docs.gitlab.com/ee/ci/yaml/README.html#whenmanual

`when:manual` をジョブに定義すると、手動で実行ボタンをクリックしたらジョブが実行されるようになります。

<figure class="figure-image figure-image-fotolife" title="ジョブにwhen:manualを定義したときのパイプライン">[f:id:gyuuuutan:20181202012650p:plain]<figcaption>ジョブにwhen:manualを定義したときのパイプライン</figcaption></figure>

上記は `when:manual` をデプロイジョブに定義した際のパイプラインですが、プッシュしただけではデプロイジョブは実行されません。  
「▶️」の実行ボタンを押すことで初めてデプロイジョブが実行されます。

これは本番環境へのデプロイジョブに有用で、不慮の事故を防ぐことができます。

## 6. artifacts：カバレッジレポートを可視化する

https://docs.gitlab.com/ee/user/project/pipelines/job_artifacts.html

`artifacts` キーワードはジョブで生成された成果物をダウンロード可能にしたり、GitLab Pages で公開したりするときに有用です。

例えば、下記のように定義すると、ユニットテストで生成されるカバレッジレポートをパイプラインからダウンロードできるようになります。

```yaml
script:
  - pytest --cov-report=html
artifacts:
  paths:
    - htmlcov
```

<figure class="figure-image figure-image-fotolife" title="ジョブにartifactsを定義したときのパイプライン">[f:id:gyuuuutan:20181202012834p:plain]<figcaption>ジョブにartifactsを定義したときのパイプライン</figcaption></figure>

**〜補足〜**  
カバレッジレポートは GitLab Pages の機能を利用することで真価を発揮します。

GitLab Pages は GitHub Pages と同じような機能ですが、 GitLab のリポジトリから静的サイトを公開するための機能です。

無料で利用できたり、https に対応していたりといった特徴があります。

https://docs.gitlab.com/ce/user/project/pages/

GitLab Pages は以下の 3 ステップを行うだけで静的サイトを自動で生成してくれます。

1. `public` という名前のディレクトリに静的ファイルを配置する。
2. `.gitlab-ci.yml` にデプロイのジョブを定義する
3. 定義したジョブを実行する

下記の URL は上記の 3 ステップで自動生成された実際の URL です。  
カバレッジレポートを展開しています。

https://jumpyoshim.gitlab.io/django-polls

生成されたカバレッジレポートを `public` ディレクトリに配置し `artifacts` の `paths` に定義するだけです。

ここで注意しなければならないのが `deploy` ステージでないと GitLab Pages のドメインを生成してくれないことです。（どうしてもうまくいかなくて数時間悩まされました。）

```yaml
pages:
  stage: deploy
  image: alpine:latest
  script:
    - mv htmlcov/ public/
  artifacts:
    paths:
      - public
  only:
    - master
```

FYI: https://gitlab.com/help/user/project/pages/index.md

# 参考例

上記の Tips を活用した `.gitlab-ci.yml` の例です。
以下を実行しています。

- GitLab コンテナレジストリへの Docker イメージのプッシュ
- ユニットテスト
- 文法チェック
- コードメトリクス計測
- Heroku へのデプロイ
- GitLab Pages の公開設定

FYI: https://gitlab.com/jumpyoshim/django-polls/blob/master/.gitlab-ci.yml

```yaml
.build_template: &build_definition
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY

build_for_master:
  <<: *build_definition
  script:
    - docker build --pull -t $CI_REGISTRY_IMAGE:latest .
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - master

build_for_tags:
  <<: *build_definition
  script:
    - docker build --pull -t $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
  only:
    - tags

.test_template: &test_definition
  stage: test
  image: python:3.7
  before_script:
    - pip install pipenv
    - pipenv install --system --dev
  except:
    - master

pytest:
  <<: *test_definition
  services:
    - postgres:latest
  variables:
    POSTGRES_DB: custom_db
    POSTGRES_USER: custom_user
    POSTGRES_PASSWORD: custom_password
  script:
    - export DATABASE_URL=postgres://$POSTGRES_USER:$POSTGRES_PASSWORD@postgres:5432/$POSTGRES_DB
    - python3 manage.py collectstatic --noinput
    - pytest --cov-report=html --tb=line
  after_script:
    - mv htmlcov/ public/
  artifacts:
    paths:
      - public

flake8:
  <<: *test_definition
  script:
    - flake8

radon:
  <<: *test_definition
  script:
    - radon cc -n C .
    - radon mi -n B .

deploy:
  stage: deploy
  image: ruby:2.5
  script:
    - gem install dpl
    - dpl --provider=heroku --app=$HEROKU_APP --api-key=$HEROKU_API_KEY --skip-cleanup
  only:
    - tags
  when: manual

pages:
  stage: deploy
  image: alpine:latest
  script:
    - echo 'Upload the coverage report'
  artifacts:
    paths:
      - public
  only:
    - master
```

# おわりに

**俺的**と書いてありますがたくさんのドキュメントや記事、職場のエンジニア、GitLab コミュニティの方の助けで得ることができた Tips です。この場を借りてお礼申し上げます。  
OSS へのコントリビュートなど、何かしらの形で恩返しをしていきたいと思っています。

**ドキュメント**

- [Configuration of your jobs with .gitlab-ci.yml](https://docs.gitlab.com/ce/ci/yaml/)
- [GitLab Pages](https://gitlab.com/help/user/project/pages/index.md)

**記事**

- [.gitlab-ci.yml によるジョブの設定方法(日本語訳)](https://qiita.com/ynott/items/1ff698868ef85e50f5a1)

**GitLab コミュニティ**

- [Slack](https://gitlab-jp.herokuapp.com/)
- [Connpass](https://gitlab-jp.connpass.com/)

GitLab に関する機能や豆知識を 7 つ厳選してみました。  
もしよろしければのぞいてみてください。

https://qiita.com/saka2jp/items/d5a63bdd3681843866f8
