---
title: "DBテーブル設計入門に参加してきました"
emoji: "🏃‍♂️"
type: "tech"
topics: [勉強会]
published: false
published_at: 2018-06-12 08:37
---

# 感想

https://supporterzcolab.com/event/407/

サポーターズ CoLab の「DB テーブル設計入門」に参加してきました。

**DB 設計ムズイ。**

データの種類によってテーブルを分割したり、ミドルウェアを変更したりと考慮するべき点が多いなと改めて感じました。
アンチパターンは存在するが、正解があってないような世界だと思うので、勉強し続けることと良質な経験を重ねることが大事なのかなという所感です。

ゲームアプリの DB テーブル設計を実例にどのような設計にするべきなのかということを学べました。

# スライド

共有はありませんでした。

# メモ書き

## 自己紹介

- 梶川 琢馬さん
- サーバーサイドエンジニア 3 年目

## DB テーブル設計の重要性

- データの蓄積や検索を楽にする
- データはシステムより長く残り続ける
- 後から変更しづらい
  - 影響範囲が広いため
  - メンテナンスがしづらいため

## DB テーブル設計の流れ

- 論理（概念）設計：人間のための設計
  - ビジネス要件をデータモデルとして表現する
  - エンティティやリレーションの定義
  - 正規化
- 物理設計：システムのための設計
  - セキュリティやパフォーマンスを考慮
  - 水平/垂直分割、Master/Slave 構成
  - 正規化を崩す
  - ミドルウェアを変更する

## オンラインゲーム

- ユーザ数が多い
- リリース後に継続して改修

## データの分類

- マスタ
  - 一度登録すると頻繁に更新されないデータ
- トランザクション
  - 頻繁に更新されるデータ
- サマリ
  - 集計結果を保存する。主に分析用。
- キャッシュ
  - 頻繁に参照されるデータの置き場所。
- ログ
  - 分析・デバッグ用

## テクニック

- 1 対 1 テーブル
  - 基本的に 1 つにできるがトランザクション頻度によって分ける
- クラス継承
  - 単一デーブル継承（STI）
  - テーブル構造を変更しやすい
  - SQL 原則から外れる
  - クラステーブル継承（CTI）
  - 後からテーブル構造を変更しにくい
  - SQL 原則にのっとった方法
  - 具象クラス継承（CCI）
  - 全体から検索しにくい

https://qiita.com/bmf_san/items/a03820b14a72db618d15

- ビジネス要件を把握する
- チームメンバーと議論する
  - 既存のテーブル設計について
  - コードレビューよりも早めにテーブル設計レビューを依頼する

## データ特性に応じたミドルウェア選択

- RDB
- KVS
- DWH

## おすすめ書籍

- [SQL アンチパターン](https://amzn.to/3SThSge)
- [現場で役立つシステム設計の原則](https://amzn.to/3STjiXR)
