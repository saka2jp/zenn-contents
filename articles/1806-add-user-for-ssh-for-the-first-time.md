---
title: "はじめてのSSH用ユーザー追加"
emoji: "🐧"
type: "tech"
topics: [Linux]
published: true
published_at: 2018-06-16 06:13
---

# はじめに

SSH 用の Linux ユーザの作成の作業依頼を初めて担当することになったため、その作業内容と目的についてまとめます。Linux ユーザの作成は定期的かつ頻繁に発生する作業であるためしっかりと理解しておく必要があります。

# Linux のユーザ作成

**なにをするのか**

- リモートホスト(AWS EC2)に SSH でログインするためのユーザーを作成する。

**なぜするのか**

- 例えば、新入社員のためのユーザ作成や新しく加わるデベロッパ用のユーザ作成など、アプリケーションやシステムの開発・運用のためには欠かせない作業です。

# サーバ側

## ユーザ追加

```console
$ useradd -m username
$ passwd username
```

ランダムなパスワードを設定し、後で本人に設定し直してもらう

## 権限変更

```console

$ cd /home/username
$ mkdir .ssh
$ chmod 700 .ssh
$ touch .ssh/authorized_keys
$ chmod 600 .ssh/authorized_keys
```

sudo 権限付与

```console
$ gpasswd -a username sudo
```

## 所有権の確認

```console
$ ls -la /home/username/.ssh
$ ls -la /home/username/.ssh/authorized_keys
```

所有者 `username` 、グループ `group` であることを確認する。  
所有者、グループが異なっていた場合は正しく設定し直す。

```console
$ chown username:group .ssh
$ chown username:group .ssh/authorized_keys
```

# クライアント側

## 鍵の作成

```console
$ cd ~
$ mkdir .ssh
$ cd .ssh
$ ssh-keygen -t rsa
```

## config ファイルの作成

~/.ssh/config

```:config
## 踏み台サーバ
Host <<Domain>>
     User <<username>>
     Hostname <<IP Adress>>

## 接続先サーバ
Host <<Domain>>
     User <<username>>
     Hostname <<IP Adress>>
     ProxyCommand ssh <<踏み台サーバのDomain>> nc %h %p
```

https://tech.recruit-mp.co.jp/infrastructure/retry-aws-bastion-host-vpc/

## 公開鍵の登録

```console
$ vi ~/.ssh/authorized_keys
```

`authorized_keys` に公開鍵をペーストする。

## サーバ接続確認

```
$ ssh-add ~/.ssh/id_rsa
$ ssh -A app1.production-jp.gu-mania.vpc
```

# 参考記事

- https://eng-entrance.com/linux-user-what
- https://eng-entrance.com/linux-user-add
