---
title: "Django管理画面のカスタマイズ方法【デザイン】"
emoji: "🎸"
type: "tech"
topics: [Django, Python]
published: false
published_at: 2018-07-31 05:51
---

# はじめに

Django 管理画面のデザイン変更方法を調べてみると 3 つほど確認できました。

## 1. style 属性を利用する

Django 管理画面のテンプレートに HTML の style タグや style 属性を追加します。

※ 参考：[【Django 入門】admin サイトの作り方からカスタマイズまで！](https://www.sejuku.net/blog/27082)

少ない変更で簡単にデザインを変更することができますが、この方法は望ましくありません。文書構造とデザインを分離するために、XHTML1.1 では style 属性は非推奨とされているためです。style 属性は将来的に廃止される可能性もあるので利用するのは避けた方が良さそうです。

※ 参考：[style 属性](https://w3g.jp/xhtml/dic/style)

## 2. django/django の css をオーバーライドする

[django/django](https://github.com/django/django)の管理画面のスタイルシートをオーバーライドします。

任意のディレクトリにオーバーライドする css ファイルを設置します。このとき、css のディレクトリ全てをコピーしても良いですし、オーバーライドしたいファイルのみコピーしてきても良いです。

※ 参考：[[Python] Django 管理サイトのカスタマイズ（表示面）](https://qiita.com/okoppe8/items/702dab51e4db5d0ed677#%EF%BC%92css%E3%82%92%E5%A4%89%E6%9B%B4%E3%81%99%E3%82%8B)

こちらは管理画面のデザインを変更する方法として最も一般的であると思いますが、できれば避けたい方法です。Django は日々盛んにアップデートされているので css も変更されないとはいえません。実際、GitHub の History を確認してみると数ヶ月単位ではありますが、css もアップデートされています。もしコピーしてきたものと Django の本体で差分が発生した場合、追従する必要性が出てくるため、保守が面倒なことになりそうです。

## 3. Media class を利用する

https://docs.djangoproject.com/en/2.1/topics/forms/media/

`admin.py` をカスタマイズする方法です。変更したいデザインがモデル固有のものであれば、以下のように css や javascript を追加することで、その内容を適用することができます。

```python
class MyModelAdmin(admin.ModelAdmin):
    class Media:
      js = ('admin/css/myadmin.js',)
      css = {
        'all': ('admin/css/myadmin.css')
      }
```

こちらの方法は、Django 本体の css をオーバーライドするわけではないのでメンテナンスが楽です。変更したいデザインがモデル固有のものであれば積極的に採用したい方法です。

※ 参考: [Overriding admin css in django](https://stackoverflow.com/questions/7357057/overriding-admin-css-in-django)
