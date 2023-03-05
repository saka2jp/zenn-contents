---
title: "libmysqlclient-devはdefault-libmysqlclient-devへ生まれ変わったのだ..."
emoji: "🐬"
type: "tech"
topics: [Python, Linux, Docker, Programming]
published: true
published_at: 2018-05-30 12:13
---

# はじめに

```Dockerfile
FROM python:latest

RUN apt -y update
RUN apt install -y build-essential libmysqlclient-dev libpcre3-dev tzdata

...
```

`python:latest` をベースイメージとした Dockerfile のビルドで下記のエラーでコケてしまった際の対処方法です。

```
E: Package 'libmysqlclient-dev' has no installation candidate
```

今まで `libmysqlclient-dev` は普通にインストールできていたのにどうしてでしょう。。。

# 原因

```Dockerfile
FROM buildpack-deps:stretch <- stretchになってる!!

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

...
```

[公式の `python:latest` の Dockerfile](https://github.com/docker-library/python/blob/b99b66406ebe728fb4da64548066ad0be6582e08/3.6/stretch/Dockerfile)のベースイメージが `jessie` から `stretch` にアップデートされており、 `libmysqlclient-dev` が `default-libmysqlclient-dev` に変更されていたことが原因でした。

https://packages.debian.org/stretch/default-libmysqlclient-dev

# 解決策

```Dockerfile
FROM python:latest

RUN apt -y update
RUN apt install -y build-essential libpcre3-dev tzdata

...
```

`libmysqlclient-dev` を削除します。  
名前から自明のように、 `default-libmysqlclient-dev` はデフォルトでインストールされているパッケージです。  
`jessie` から `stretch` でデフォルトのパッケージに繰り上がったらしいですね。
