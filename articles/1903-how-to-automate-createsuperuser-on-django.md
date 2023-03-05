---
title: "【Django】ワンライナーでスーパーユーザーを作成する方法"
emoji: "🎸"
type: "tech"
topics: [Django, Python]
published: true
published_at: 2019-03-10 09:37
---

Django Admin のスーパーユーザーをワンライナーで作成する方法を紹介します。

Django Admin のスーパーユーザーの作成は本来、[createsuperuser](https://docs.djangoproject.com/ja/2.1/ref/django-admin/#createsuperuser) コマンドで作成できますが、インタラクティブ（対話的）に実行されます。

```sh
$ python manage.py createsuperuser
Username (leave blank to use 'root'): admin
Email address: admin@example.com
Password:
Password (again):
Superuser created successfully.
```

命令的に実行する場合はこれでもいいのですが、宣言的に実行したいケースがあります。

例えば、Docker Compose による開発環境構築で、環境を立ち上げるたびにスーパーユーザーを作成するのは手間ですしスマートではありません。

環境を立ち上げるときにスーパーユーザーも作成されるのが理想です。

宣言的に実行するケースではパスワードを設定できないのが現在の Django の仕様です。パスワードを設定しないままスーパーユーザーを作成することも可能ですが、パスワードを設定するまでログインはできません。

そのためワンライナーで実行できるようにするためには createsuperuser コマンドを `--password` のようなオプションでパスワードを指定できるようにカスタマイズしてあげる必要があります。

`management/commands/` 配下に `custom_createsuperuser.py` のようなファイルを作成します。

```python
from django.contrib.auth.management.commands import createsuperuser
from django.core.management import CommandError


class Command(createsuperuser.Command):
    help = 'Create a superuser with a password non-interactively'

    def add_arguments(self, parser):
        super(Command, self).add_arguments(parser)
        parser.add_argument(
            '--password', dest='password', default=None,
            help='Specifies the password for the superuser.',
        )

    def handle(self, *args, **options):
        options.setdefault('interactive', False)
        username = options.get('username')
        email = options.get('email')
        password = options.get('password')
        database = options.get('database')

        if not (username and email and password):
            raise CommandError('--username, --email and --password are required options')

        user_data = {
            'username': username
            'email': email,
            'password': password,
        }

        exists = self.UserModel._default_manager.db_manager(database).filter(username=username).exists()
        if not exists:
            self.UserModel._default_manager.db_manager(database).create_superuser(**user_data)
```

これで以下のようにワンランナーでスーパーユーザーを作成することができます。

```sh
$ python manage.py custom_createsuperuser --username admin --email admin@example.com --password admin
```

ついでに pytest で書いたユニットテストも掲載しておきます。

```python
import pytest

from django.core.management import CommandError, call_command
from django.test.client import Client


@pytest.mark.django_db
@pytest.mark.parametrize('username,email,password', [
    ('admin', 'admin@example.com', 'admin')
])
def test_success_case(username, email, password):
    call_command(
        'createsuperuser_with_password',
        '--username', username,
        '--email', email,
        '--password', password
    )
    client = Client()
    response = client.login(username=username, password=password)
    assert response is True


@pytest.mark.django_db
@pytest.mark.parametrize('username,email', [
    ('admin', 'admin@example.com')
])
def test_CommandError_case(username, email):
    with pytest.raises(CommandError):
        call_command(
            'createsuperuser_with_password',
            '--username', username,
            '--email', email
        )
```

参考：
https://stackoverflow.com/questions/6244382/how-to-automate-createsuperuser-on-django
