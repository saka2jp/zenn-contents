---
title: "【Django】フォームのテストの書き方"
emoji: "🎸"
type: "tech"
topics: [Django, Python, テスト]
published: false
published_at: 2019-03-11 09:30
---

```python
from django import forms

class NameForm(forms.Form):
    name = forms.CharField(label='Your name', max_length=100)
```

上記の `NameForm` という Form のテストを pytest で書いてみます。

```python
import pytest

from .forms import NameForm


@pytest.mark.parametrize('name,is_valid', [
    ('name', True),
    (1, False),
    (None, False),
])
def test_nameform(name, is_valid):
    form_data = {
        'name': name
    }
    form = NameForm(data=form_data)
    assert form.is_valid() is is_valid
```

[is_valid()](https://docs.djangoproject.com/en/2.2/ref/forms/api/#django.forms.Form.is_valid) という有効なデータかどうかを検証するメソッドを利用すれば簡単にテストがかけるという紹介でした。
