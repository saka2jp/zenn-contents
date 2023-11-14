---
title: "コテコテのGitLabユーザがCircleCIに入門してみた"
emoji: "🦊"
type: "tech"
topics: [GitLab, CircleCI]
published: false
published_at: 2018-09-26 02:02
---

# はじめに

GitLab ユーザが GitHub で CI 構築するために CircleCI に入門する物語です。

GitLab 10.6 で[GitLab CI/CD for Github](https://about.gitlab.com/features/github/)がリリースされているのにもかかわらず、CircleCI を選定した理由は以下です。

- GitHub.com で利用されている CI サービス第 2 位(2017 年時点)

https://blog.github.com/2017-11-07-github-welcomes-all-ci-tools/

- Forrester に Leader という最上位のランクに認定されている

https://www2.circleci.com/circleci-forrester-wave-leader-2017.html

- 単純にほかの CI サービスを触ってみたかった
- CircleCI 2.0 がいい感じらしい

ちなみに、GitLab CI/CD 以外に他の CI サービスを触るのは初めてです。なんだがいけないことをしているような気分です。

まずは、GitLab CI/CD、CircleCI がそれぞれどんなサービスで何を売りとしているのかを理解してみます。

# GitLab CI/CD って何？

https://about.gitlab.com/features/gitlab-ci-cd/

> GitLab has integrated CI/CD pipelines to build, test, deploy, and monitor your code
> Rated #1 in the Forrester CI Wave™

Forrester に認められた No.1CI サービス。

https://about.gitlab.com/features/gitlab-ci-cd/

> GitLab supports development teams with a well-documented installation and configuration processes, an easy-to-follow UI, and a flexible per-seat pricing model that supports self service. GitLab’s vision is to serve enterprise-scale, integrated software development teams that want to spend more time writing code and less time maintaining their tool chain

Forrester のレポートによると、整備されたドキュメントや使いやすい UI、GitLab のビジョンが賞賛されています。

GitLab への移行で他の CI サービス使っているというケースは考えられますが、~~よっぽどの変態でもない限り~~、GitLab を使っているなら GitLab CI/CD を選択するのがおそらく一般的なのではないでしょうか。

# CircleCI って何？

https://circleci.com/product/

> Power, Flexibility, and Control
> CircleCI gives your team more speed and configurability than ever before.

他サービスとの柔軟な連携や容易な設定が売りのようです。
Forrester には GitLab と同じく Leader に認定されています。

> Organizations striving to be lean and stay lean will appreciate the simplified, nearly zero-maintenance approach CircleCI provides. Existing customers will appreciate the new features of CircleCI 2.0 that appear to completely address all the top asks that customers are looking for.

Forrester のレポートによると、ほぼ保守が不要な点と 2.0 の新機能が賞賛されています。

GitHub であれば Travis CI に続いて 2 番目のシェア率と利用者も多いようです。

それぞれ何となくどんなサービスなのかはわかりましたが、まだボヤけているので GitLab CI/CD と CircleCI の違いを調べてみることにしました。

# GitLab CI/CD vs CircleCI

なんと、それぞれサービスであの CI サービスとはここが違うという主張がまとまっているページがありました。
それぞれの主張を列挙してみます。

## GitLab CI/CD の主張

https://about.gitlab.com/comparison/gitlab-vs-circleci.html

- CLI, API, Webhook を提供しており、これらを活用することで他サービスとの連携が可能である。
- GitLab CI/CD にあって CircleCI にない機能が 29 個ある。
  - Application performance monitoring and alerts
  - Preview your changes with Review Apps
  - Code Quality
  - Show code coverage rate for your pipelines
  - Auto DevOps
  - Easy integration of existing Kubernetes clusters
  - Easy creation of Kubernetes clusters on GKE
  - etc...

## CircleCI の主張

https://circleci.com/gitlab-versus-circleci/

- チームの成長段階に合った最適なツールとの統合ができる。
  - GitHub, BitBucket, Slack, AWS など、サービス切り替えが容易である。
  - GitLab CI は GitLab のユーザしか使えない。
- CI/CD にのみ焦点を当てたサービスである。
- builders の管理が容易。
  - GitLab Runner は独自で管理する必要がある。

CircleCI の GitLab はこうだという主張に誤りがあるように感じますが、まとめると、

**GitLab CI/CD**「GitLab は、DevOps を実現するための統合ツールであり、我々はその一つのサービスである。DevOps を実現するための機能をたくさん提供している。」

**CircleCI**「私たちは何かのツールに依存したサービスではない。CI/CD にのみ焦点を当てており、他サービスとの連携・切り替えが容易である。」

みたいなイメージかなと思います。

# CircleCI を使ってみた

実際のリポジトリはこちらです。

https://github.com/saka2jp/aws-sm

```yaml
version: 2
jobs:
  build:
    working_directory: ~/aws-sm
    docker:
      - image: circleci/python:3.7
    steps:
      - checkout
      - run: sudo chown -R circleci:circleci /usr/local/bin
      - run: sudo chown -R circleci:circleci /usr/local/lib/python3.7/site-packages
      - restore_cache:
          key: deps1-{{ .Branch }}-{{ checksum "Pipfile.lock" }}
      - run:
          command: |
            pipenv install --system --dev
      - save_cache:
          key: deps1-{{ .Branch }}-{{ checksum "Pipfile.lock" }}
          paths:
            - ".venv"
            - "/usr/local/bin"
            - "/usr/local/lib/python3.7/site-packages"
      - run:
          command: |
            py.test --tb=line --cov-report=html
      - store_artifacts:
          path: htmlcov/
      - run:
          command: |
            flake8
      - run:
          command: |
            radon cc -n C .
            radon mi -n B .
```

テスト、文法チェック、コードメトリクス計測を走らせてみました。
以下の `config.yml` を参考にさせていただきました。

https://github.com/CircleCI-Public/circleci-demo-python-django/blob/master/.circleci/config.yml

# 感想

## いい感じ

- GitHub のアカウントを持っていれば簡単に利用を始められる
- OS と言語を選択することで YAML の雛形を生成してくれる
- GitLab CI/CD に似た構文である（CI サービス自体がそこまで異なる構文になるとは思っていませんでしたが）

## なんだかなぁ

- プルリクを作成するまで CI の実行結果を GitHub 上で確認できない（Fail した時はメールで通知される）
- ジョブが失敗していてもマージできてしまう
- coverage のバッジを生成できない

「なんだかなぁ」に関してはこれを解消するための方法があるかもしれません。入門者が「あれ？」と思った点です。

# おわりに

使っている時間が全く違うのでかなり GitLab CI/CD に偏った意見になりますが、統合ツールとしての GitLab の使いやすさを実感する時間になりました。

GitLab は GitLab 上で CI/CD の実行結果の確認ができ、CI が失敗した際にはマージボタンが押せません。また、coverage のバッジを自動生成してくれるような設定が可能です。

しかし、かなりの短時間でテストの自動化が実現できた CircleCI は素晴らしい CI ツールだと思います。GitHub では CircleCI 使っていきたいです。

今回試したのはテストのみだったのでビルド、デプロイも実装してみると CircleCI の便利さがより見えてくるかもしれません。
今後機会があれば試してみたいです。
