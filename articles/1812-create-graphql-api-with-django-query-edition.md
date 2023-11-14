---
title: "DjangoでGraphQLを実装する【Query編】"
emoji: "🎸"
type: "tech"
topics: [GraphQL, Django, Python, Programming]
published: false
published_at: 2018-12-02 11:55
---

# はじめに

[Django Advent Calendar 2018](https://qiita.com/advent-calendar/2018/django) の 2 日目の記事です。

https://qiita.com/advent-calendar/2018/django

REST の次のパラダイムとして注目されている GraphQL。  
本記事は Django で GraphQL を実装する方法を紹介します。

# GraphQL の動向

[GraphQL](https://graphql.org/)は Facebook によって開発された OSS で、Web API のクエリ言語です。

https://graphql.org/

GitHub の API に採用されたり、AWS のフルマネージド GraphQL サービス [AppSync](https://aws.amazon.com/jp/appsync/)が発表されたりなど徐々に盛り上がりを見せているように思います。

参考：[2018 年の API 動向 - ポスト REST 時代の到来？](http://gihyo.jp/dev/column/newyear/2018/api-prospect) / gihyo.jp

また、今年で 3 年目になる[GraphQL Summit 2018](https://summit.graphql.com/)では 850 人以上のエンジニアが集まり盛り上がりを見せたようです。

参考：[GraphQL Summit 2018 に参加してきました](https://tech.mercari.com/entry/2018/11/30/100332) / Mercari Engineering Blog

# Python / Django 界隈における GraphQL の動向

- [PyCon 2018]() /「[Win back lovely API: GraphQL in Python](https://us.pycon.org/2018/schedule/presentation/142/)」
- [PyConJP 2018](https://pycon.jp/2018/) /「[REST API に疲れたあなたへ贈る GraphQL 入門](https://www.slideshare.net/keisuketsukagoshi/rest-api-graphql)」
- [DjangoCon US 2018](https://2018.djangocon.us/) /「[Introduction to Django and GraphQL](https://speakerdeck.com/patrick91/introduction-to-django-and-graphql-djangocon-us-2018)」
- [DjangoCon US 2018](https://2018.djangocon.us/) /「[Build a GraphQL API powered by Django](https://2018.djangocon.us/tutorial/build-a-graphql-api-powered-by-django/)」

上記のように GraphQL に関連したトークやチュートリアルが多く確認できました。

このような注目度や GraphQL の理解を深めるといった目的から自分の得意としている言語とフレームワークで GraphQL を実装してみようと思ったのが動機です。

# Graphene

[Graphene](https://graphene-python.org/)は GraphQL フレームワークで PyPI に[graphene](https://github.com/graphql-python/graphene)として登録されています。

https://graphene-python.org/

また、同じ組織が Django で Graphene を利用するためのライブラリ[graphene-django](https://github.com/graphql-python/graphene-django) を公開しています。

今回、graphene-django を利用して Django で GraphQL を実装してみたいと思います。

# GraphQL API を作成する

[Graphene-Django](http://docs.graphene-python.org/projects/django/en/latest/) のチュートリアルをベースに GraphQL API を作成してみます。  
作成する API は食材の名前やカテゴリに関する API です。

Graphene-Django のチュートリアルをやる前に

- GraphQL の[イントロダクション](https://graphql.org/learn/)
- Graphene の[チュートリアル](https://docs.graphene-python.org/en/latest/)

に目を通しておくといいみたいです。

説明に必要な部分だけ抜粋しますので詳細は Graphene-Django のチュートリアルをご確認ください。

## モデル

材料モデルとそれにリーレーションを持つカテゴリモデルを定義します。  
ここでは Graphene 特有の書き方はありません。

```python
from django.db import models


class Category(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name


class Ingredient(models.Model):
    name = models.CharField(max_length=100)
    notes = models.TextField()
    category = models.ForeignKey(Category, related_name='ingredients', on_delete=models.CASCADE)

    def __str__(self):
        return self.name
```

## ルーティング

GraphQL のエンドポイントを定義します。エンドポイントは基本的に 1 つです。

`graphiql=True` とすると[GraphiQL](https://github.com/graphql/graphiql/)という Visual Editor が使えるようになります。

```python
from django.conf.urls import path
from django.contrib import admin

from graphene_django.views import GraphQLView

from cookbook.schema import schema

urlpatterns = [
    path('admin/', admin.site.urls),
    path('graphql/', GraphQLView.as_view(graphiql=True, schema=schema)),
]
```

## GraphQL スキーマ

> A GraphQL schema describes your data model, and provides a GraphQL server with an associated set of resolve methods that know how to fetch data.

FYI: https://docs.graphene-python.org/en/latest/quickstart/#creating-a-basic-schema

GraphQL スキーマはデータのフィールドや型などデータを取得するための情報を定義するデータセットです。

クエリが入力されると定義したスキーマに対してクエリが検証され、実行されます。  
REST におけるスキーマとほぼ同等の役割と認識しています。

GraphQL ではデータに対する操作は `Query` と `Mutation` の二つに分類されるみたいです。  
`Query` はデータの取得で、`Mutation` はデータの登録や更新です。

### Query

データの取得を実装します。  
アプリケーション下のディレクトリに作成した `schema.py` に Query を定義します。

#### 全件取得

```python
import graphene

from graphene_django.types import DjangoObjectType

from .models import Category, Ingredient


class CategoryType(DjangoObjectType):
    class Meta:
        model = Category


class IngredientType(DjangoObjectType):
    class Meta:
        model = Ingredient


class Query:
    all_categories = graphene.List(CategoryType)
    all_ingredients = graphene.List(IngredientType)

    def resolve_all_categories(self, info, **kwargs):
        return Category.objects.all()

    def resolve_all_ingredients(self, info, **kwargs):
        return Ingredient.objects.select_related('category').all()
```

GraphiQL で `Ingredient` のデータを全件取得するクエリを実際に叩いてみます。

※ 検証前にフィクスチャでデータを事前にロードしておく必要があります。

<figure class="figure-image figure-image-fotolife" title="全件取得のクエリ">[f:id:gyuuuutan:20181202215724p:plain]<figcaption>全件取得のクエリ</figcaption></figure>

リレーションをもつ全件取得もできます。

<figure class="figure-image figure-image-fotolife" title="リレーションをもつ全件取得">[f:id:gyuuuutan:20181202221239p:plain]<figcaption>リレーションをもつ全件取得</figcaption></figure>

#### 1 件取得

```python
# ...

class Query(object):
    category = graphene.Field(CategoryType,
                              id=graphene.Int(),
                              name=graphene.String())
    all_categories = graphene.List(CategoryType)


    ingredient = graphene.Field(IngredientType,
                                id=graphene.Int(),
                                name=graphene.String())
    all_ingredients = graphene.List(IngredientType)

    def resolve_all_categories(self, info, **kwargs):
        return Category.objects.all()

    def resolve_all_ingredients(self, info, **kwargs):
        return Ingredient.objects.all()

    def resolve_category(self, info, **kwargs):
        id = kwargs.get('id')
        name = kwargs.get('name')

        if id is not None:
            return Category.objects.get(pk=id)

        if name is not None:
            return Category.objects.get(name=name)

        return None

    def resolve_ingredient(self, info, **kwargs):
        id = kwargs.get('id')
        name = kwargs.get('name')

        if id is not None:
            return Ingredient.objects.get(pk=id)

        if name is not None:
            return Ingredient.objects.get(name=name)

        return None
```

GraphiQL でクエリを実際に叩いてみます。

<figure class="figure-image figure-image-fotolife" title="1件取得のクエリ">[f:id:gyuuuutan:20181202223201p:plain]<figcaption>1件取得のクエリ</figcaption></figure>

指定した `id` や `name` でオブジェクトが取得できていることがわかります。

#### 複雑な取得

[Relay](https://docs.graphene-python.org/en/latest/relay/) や [django-filter](https://github.com/carltongibson/django-filter) を活用することでより複雑なクエリでオブジェクトを取得できたり、実装をシンプルにしたりすることができるようです。

```python
from graphene import relay
from graphene_django import DjangoObjectType
from graphene_django.filter import DjangoFilterConnectionField

from .models import Category, Ingredient


class CategoryNode(DjangoObjectType):
    class Meta:
        model = Category
        filter_fields = ['name', 'ingredients']
        interfaces = (relay.Node, )


class IngredientNode(DjangoObjectType):
    class Meta:
        model = Ingredient
        filter_fields = {
            'name': ['exact', 'icontains', 'istartswith'],
            'notes': ['exact', 'icontains'],
            'category': ['exact'],
            'category__name': ['exact'],
        }
        interfaces = (relay.Node, )


class Query(object):
    category = relay.Node.Field(CategoryNode)
    all_categories = DjangoFilterConnectionField(CategoryNode)

    ingredient = relay.Node.Field(IngredientNode)
    all_ingredients = DjangoFilterConnectionField(IngredientNode)
```

GraphiQL で実際にクエリを叩いてみます。

<figure class="figure-image figure-image-fotolife" title="複雑な取得のクエリ">[f:id:gyuuuutan:20181202224048p:plain]<figcaption>複雑な取得のクエリ</figcaption></figure>

# 所感

エンドポイントを一つ用意するだけでデータの取得ができてしまうのは色々恩恵がありそうです。  
リクエスト数を削減できたり、必要なフィールドのみを取得できたり柔軟性が高いのも GraphQL の特徴だと実感することができました。

クエリの書き方は若干癖がありますが、かなり直感的ではあるので一度書き方を覚えてしまえばそこまで苦労はしなそうです。

ただ、サーバー側が楽になった分クライアント側の実装者の負担が増えるのかなと感じました。

REST の場合はドキュメント通りに叩けばドキュメント通りに結果を返してくれますが（そうではない API もありますが）、GraphQL の場合はクライアントの実装者が適切なクエリを考えなければならないので、データベースやクエリ言語に対する一定の理解が必要になりそうです。

https://k0kubun.hatenablog.com/entry/graphql

https://workplus.feedforce.jp/entry/2018/03/29/133000

# おわりに

1 つの記事でまとめたかったのですがかなりのボリュームになってしまうので、3 部構成で書きたいと思います。

現時点では以下の構成で書く予定です。

1. Django で GraphQL を実装する【Query 編】：本記事
2. Django で GraphQL を実装する【Mutation 編】：公開予定
3. Django で GraphQL を実装する【認証と認可編】：公開予定

次のステップとして考えていることは[Apollo](https://www.apollographql.com/)で GraphQL を実装することです。

今回は Python/Django で GraphQL を実装しましたが、GraphQL の特性上、サーバーとクライアントを一緒に実装するのがよいと思っています。

Apollo はクライアント側のライブラリが充実しているようです。

どこかの機会で挑戦してみたいです。

https://www.kabuku.co.jp/developers/develop-web-service-with-apollo-graphql
