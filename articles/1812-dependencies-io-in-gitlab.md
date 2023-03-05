---
title: "Dependencies.ioによるGitLabプロジェクトのマニフェスト自動アップデート"
emoji: "🦊"
type: "tech"
topics: [GitLab, SRE]
published: true
published_at: 2018-12-19 02:43
---

# はじめに

[GitLab Advent Calendar 2018](https://qiita.com/advent-calendar/2018/gitlab)の 15 日目の記事です。

https://qiita.com/advent-calendar/2018/gitlab

みなさんマニフェスト（ `package.json`, `composer.json`, `requirements.txt`, etc.）のアップデートはどうされていますか？

気まぐれなタイミングで行なっていたり、手作業で行なっていたりしないでしょうか？

> 通常運用中のシステムに人手が必要なら、それはバグだ。

引用：[SRE サイトリライアビリティエンジニアリング ―Google の信頼性を支えるエンジニアリングチーム](https://amzn.to/3ZkQYQt)

５章の「トイル（労苦）の撲滅」での一節です。

- 手作業であること
- 繰り返されること
- 自動化できること
- 戦術的であること
- 長期的な価値を持たないこと
- サービスの成長に対して O(n) であること

上記の項目に当てはまるようなタスクは可能な限り自動化していこうというのが SRE の考え方です。  
マニフェストのアップデートはまさにこれに当てはまるのではないかと思います。

本記事は、GitLab プロジェクトにおけるマニフェストのアップデートというトイルを [Dependencies.io](https://www.dependencies.io/) というサービスを利用して撲滅していこうという内容です。

https://www.dependencies.io/

# 導入方法

導入はお手軽 2 ステップです。とても簡単に導入することができます。

1. `dependencies.yml` をリポジトリに追加する
2. 対象リポジトリを [Dependencies.io](https://app.dependencies.io/projects) に追加する

だけです。

# 実践

下記の `requirements.txt` のアップデートの設定をしてみます。

```python
requests==2.20.0
Django==2.1
```

アップデートの MR が作成されることを確認するためにわざと最新ではないバージョンを指定しています。

## 1. dependencies.yml をリポジトリに追加する

![image](/images/hatena/20181219022459.png)

## 2. GitLab の Personal Access Token を発行する

スコープは `api`, `read_repository` を選択します。

![image](/images/hatena/20181219021222.png)

## 3. GitLab プロジェクトを選択する

![image](/images/hatena/20181219021048.png)

## 4. 対象リポジトリの URL と発行した Personal Access Token を入力する

![image](/images/hatena/20181219021627.png)

上記で設定は完了です。GitLab の MR を確認してみると...

![image](/images/hatena/20181219022757.png)

確かにアップデートの MR が自動生成されています！

ちなみに Dependencies.io のプロジェクトページはこんな感じです。

![image](/images/hatena/20181219022741.png)

# Dependencies.io の何がいいのか

Dependencies.io の特徴をまとめてみました。

## ワークフローにあったアップデート

**複数言語のアップデートが可能**

1 つのリポジトリで複数の開発言語を利用しているケースがあるかもしれません。そういった場合でも 1 つの言語だけでなく、複数の開発言語をアップデートするように設定することができます。2018 年 12 月時点で `JavaScript`, `Python`, `PHP`, `Docker`, `Git` に対応しており, `Ruby`, `Java`, `Go`, `Rust`, `iOS`, `Android`, `.NET` に関しても順次追加予定らしいです。

**カスタムスクリプト実行後のアップデートが可能**

リポジトリによってはカスタムスクリプトやあるコマンドを実行した後にパッケージのアップデートを行いたいといったケースがあるかもしれません。そういった場合でもアップデート前に任意のコマンドを実行するように設定することができます。

上記のように様々なワークフローに対応することができます。

## アップデートのカスタマイズ

**パッケージ管理ファイルの場所に依存しない**

パッケージ管理ファイルのパスを指定できるため、必ずしもトップディレクトリに配置する必要はありません。

**バージョンのフィルタリングが可能**

フィルタリングという機能を利用すると、必要な更新と依存関係を正確に指定することができます。これにより、いくつかの依存関係をパッチやバグ修正だけに制限することができます。例えば、 `L.Y.Y` と指定するとマイナーとパッチのバージョンだけ受け入れられて、メジャーバージョンはロックされます。また、 バージョンといってもセマンティックバージョンが全てではありません。正規表現に対応しているため `nighly` のように指定することも可能です。

**パイプラインが自動で走る**

GitLab CI/CD のパイプラインを実行してくれるため、テストジョブが設定されている場合、アップデートされたバージョンで確かにテストが通ったことを確認することができます。

**ロックファイルの自動アップデート**

マニフェストだけではなく、ロックファイルの依存関係もアップデートするように設定することが可能です。

## スケジューリング

**アップデートの頻度・時間帯の指定が可能**

毎日、毎週、毎月のスケジュールオプションにより、プロジェクトや開発チームにあったタイミングでアップデートを受け取ることができます。時間帯の指定も可能です。

# おわりに

Free プランはパブリックリポジトリのみなので、プライベートリポジトリで Dependencies.io を利用する場合は有料プランに加入する必要がありますが、それにしても無料でここまでできてしまうのはなかなかのものではないでしょうか。

導入前の調査で[Greenkeeper](https://greenkeeper.io/)や[Dependabot](https://dependabot.com/), [Renovate](https://renovatebot.com/)などの類似サービスも候補に上がっていたのですが、GitLab に対応していることがホームページのトップで確認できたのが Dependencies.io だけだったので今回は Dependencies.io を試してみました。

https://www.slideshare.net/teppeis/automated-dependency-updates-with-renovate-102769685

記事を書いたあとに[Teppei Sato](https://www.slideshare.net/teppeis)さんの SlideShare をみて Renovate が GitLab に対応しているらしいことを知り、確認してみたところ確かに GitLab に対応していました。
オートマージなど、機能的には Renovate の方が充実してそうな印象を持ったのでこちらもいつか試してみたいです。

GitLab に関する機能や豆知識を 7 つ厳選してみました。  
もしよろしければのぞいてみてください。

https://qiita.com/jumpyoshim/items/d5a63bdd3681843866f8
