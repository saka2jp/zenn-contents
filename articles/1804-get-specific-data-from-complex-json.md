---
title: "複雑なJSONから特定のデータを再帰で取り出せるようになるための4ステップ"
emoji: "🐍"
type: "tech"
topics: [Python, Programming]
published: true
published_at: 2018-04-03 01:58
---

# はじめに

http://qiita.com/JumpeiYoshimura/items/3b138d3ebbdaa873eaa5

こちらの記事の「複雑な JSON から特定のデータを取り出す」実装するにあたり、段階的に考えることで徐々に正解に近づけていきました。もし、上記の記事だけではわかりにくかったり、もう少し詳しい説明が読みたかったりする場合はぜひ参考にしてみてください。

# Step1. 配列から Str 型の要素を取得する

配列の要素が Str 型であれば res に追加を配列の要素個数回繰り返します。

**サンプルデータ**

```python
sample = ["a", "b", 1]
```

**取得したい値**

```python
["a", "b"]
```

**実装**

```python
res = []
for v in sample:
    if isinstance(v, str):
        res.append(v)

print(res) # ["a", "b"]
```

for を用いた反復法です。Step2 以降との比較のためにリスト内包表記では実装していません。
非常にシンプルな例なので解説は不要かと思います。

# Step2. 2 階層構造の配列からすべての Str 型の要素を取得する

次は、データが 2 階層、つまり、配列の要素に配列がある場合を考えてみましょう。

**サンプルデータ**

```python
sample = [["a", 1], "b"]
```

**取得したい値**

```python
["a", "b"]
```

**実装**

```python
res = []
for arg in sample:
    if isinstance(arg, str):
        res.append(arg)
    if isinstance(arg, list):
        for v in arg:
            if isinstance(v, str):
                res.append(v)

print(res) # ["a", "b"]
```

Step1 の実装を元に 2 階層の for 文で実装しました。

見て分かる通り、期待の結果は得られますが、Str 型かどうかの条件分岐を 2 回行なっており冗長であることがわかります。

この実装だと配列が 3 階層、4 階層、、、と増えた場合、その分 for 文の階層を増やすことになり、さらに冗長なコードになってしまいます。

そこで再帰関数の登場です。

再帰は、**大きな問題を小さな問題に分割して解決する分割統治法** であることが特徴で、Step2 で実装したコードを再帰関数で実装すると冗長な部分を簡潔に実装することができます。Step3 でやってみましょう。

# Step3. n 階層構造の配列から Str 型の要素を取得する

Step2 の実装を再帰法で実装してみます。

**サンプルデータ**

```python
sample = ["a",["b", 1, [[["c", 2], 3], 4], "d"], ["e"]]
```

**取得したい値**

```python
["a", "b", "c", "d", "e"]
```

**実装**

```python
def get_str(arg):
    res = []
    if isinstance(arg, str):
        res.append(arg)
    elif isinstance(arg, list):
        for v in arg:
            return res += get_str(v)

print(get_str(sample)) #
```

処理を順番に追いながら解説していきます。
`sample` はリスト型なので、最初の処理は、

```python
for v in arg:
    res += get_str(v)
```

となります。この時、`arg` は 3 要素なので、上記の処理は下記の処理をそれぞれ行うことになります。

```python
①res += get_str("a")
②res += get_str(["b", 1, [[["c", 2], 3], 4], "d"])
③res += get_str(["e"])
```

それぞれの処理をさらに追ってみましょう。

```python
①res += get_str("a")
```

これは、`res` に `"a"` を追加して終了です。

```python
②res += get_str(["b", 1, [[["c", 2], 3], 4], "d"])
```

こちらは、下記のそれぞれの処理を行います。

```python
res += get_str("b")
res += get_str(1)
res += get_str([[["c", 2], 3], 4])
res += get_str("d")
```

これらもそれぞれ処理を追うことができます。

```python
③res += get_str(["e"])
```

そしてこちらは、下記の処理を行います。

```python
res += get_str("e")
```

同じようにどんどん処理を追ってくことができます。

```python
①res += get_str("a")
>> res.append("a") *
②res += get_str(["b", 1, [[["c", 2], 3], 4], "d"])
>> res += get_str("b")
   >> res.append("b") *
   res += get_str(1)
   res += get_str([[["c", 2], 3], 4])
   >> res += get_str([["c", 2], 3)
      >> res += get_str(["c", 2])
         >> res += get_str("c")
            >> res.append("c") *
            res += get_str(2)
      res += get_str(4)
   res += get_str("d")
   >> res.append("d") *
③res += get_str(["e"])
>> res += get_str("e")
   >> res.append("e") *
```

\*印のところが res に最終的に追加されている部分です。そして、最終的に `get_str` は `res` を return するため、Str 型の値のみが格納された配列を取得することができます。

このように `get_str` の引数が Str 型であれば配列に追加され、List 型であればさらにその要素に対して `get_str` （関数自身）を呼ぶことで、再帰的に配列の要素を処理してくれる関数が実装できました。

Step2 のような反復法だと、n 階層分 for を書く必要があるのに比べ、コードの量が少なく簡潔に実装できていることがわかります。

# Step4. Int,Str,List,Dict の混合オブジェクト（JSON）から Str 型の value を取得する

ついに目的の実装ですが、Step3 まで終えた皆さんは簡単に実装できると思います。

Step3 の実装では List 型か Str 型かの判定しか行っていませんでしたが、そこに Dict 型の判定が加わるだけです。

**サンプルデータ**

```python
sample = {
    "a": [{
        "b": "y",
        "c": [{
            "d": [2,3]
        }],
        "e": {"g": "z"}
        }],
        "f": ["x"],
}
```

**取得したい値**

```python
["x", "y", "z"]
```

**実装**

```python
def get_str(arg):
    res =[]
    if isinstance(arg, str):
        res.append(arg)
    elif isinstance(arg, list):
        for item in arg:
            res += get_str(item)
    elif isinstance(arg, dict):
        for value in arg.values():
            res += get_str(value)
    return res

print(get_str(sample)) # ["x", "y", "z"]
```

Step3 に Dict 型が来た場合の処理を追加します。
Dict 型の場合は、キーバリューのバリューを `get_str` の引数に与えてあげるだけですね。

これで JSON から特定のデータを取り出せることができました！

# リファクタ

Step4 の実装だと他のデータを取り出したい時に関数を書き換えなければいけないため、データの判定は外出しすることにします。
こうすることで、他のデータを取り出したい時にそのデータを判定する関数を実装するだけで済みます。

```python
def search(arg, cond):
    res =[]
    if cond(arg):
        res.append(arg)
    elif isinstance(arg, list):
        for item in arg:
            res += search(item, cond)
    elif isinstance(arg, dict):
        for value in arg.values():
            res += search(value, cond)
    return res

def has_str(arg):
    return isinstance(arg, str)

def get_str(arg):
    return search(arg, has_str)
```
