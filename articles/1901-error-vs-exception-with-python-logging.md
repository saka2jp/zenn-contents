---
title: "【Python】loggingのerror()とexception()の違い"
emoji: "🐍"
type: "tech"
topics: [Python]
published: true
published_at: 2019-01-24 07:30
---

logging の error()と exception()は同じ `ERROR` レベルのロギング関数なのですが、どういった違いがあるのでしょうか？

結論は Python 公式ドキュメントの「Logging HOWTO」に記載されています。

> Logger.exception() は Logger.error() と似たログメッセージを作成します。違いは Logger.exception() がスタックトレースを一緒にダンプすることです。例外ハンドラでだけ使うようにしてください。

https://docs.python.org/ja/3/howto/logging.html#loggers

実際にコーディングして確かめてみます。

例外処理をまずは error()でログ出力してみます。
`ZeroDivisionError` の例外が投げられるようなコーディングをします。

```python
import logging

logger = logging.getLogger('example')
handler = logging.StreamHandler()
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)

try:
    result = 1 / 0
except ZeroDivisionError as e:
    logger.error(f'{e}')
```

これの出力は以下のようになりました。

```sh
2019-01-24 14:27:39,163 - example - ERROR - division by zero
```

続いて、exception()です。

```python
try:
    result = 1 / 0
except ZeroDivisionError as e:
    logger.exception(f'{e}')
```

この出力は以下のようになりました。

```sh
2019-01-24 14:27:39,175 - example - ERROR - division by zero
Traceback (most recent call last):
  File "<ipython-input-2-6a7fc51b65bc>", line 2, in <module>
    result = 1 / 0
ZeroDivisionError: division by zero
```

上記のように、exception() のログにはスタックトレースが出力されているのに対し、error() のログには確認できません。

実際に試してみると非常にシンプルな違いですね。

例外処理のとき、exception() でログを出力するとスタックトレースとともに詳細なログが出力されます。
よって、例外処理のときは exception() を、それ以外のときは error() を使うと良いということでした。

error()の場合
![image](/images/hatena/20190124145333.png)

exception()の場合
![image](/images/hatena/20190124145402.png)
