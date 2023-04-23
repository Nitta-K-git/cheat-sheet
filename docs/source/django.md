# Django

- [動かして学ぶ！Python Django開発入門 第2版（大高 隆）｜翔泳社の本](https://www.shoeisha.co.jp/book/detail/9784798174198)

## Install(Windows)

### 仮想環境作成

仮想環境作成
```sh
python -m venv venv_django
```

activate
```sh
venv_django\Scripts\activate.bat

# deactivateするときは
# venv_django\Scripts\deactivate.bat
```

install django
```sh
pip install django psycopg2-binary
```

### Install postgresql

- https://www.enterprisedb.com/downloads/postgres-postgresql-downloads
  - 適切なインストーラーをダウンロードして実行
  - 設定はデフォルトでOK
  - 途中でパスワードを入力する
  - インストール完了後の追加インストールのチェックは外してよい


### Create database

#### GUI

1. pgAdmin4をウィンドウズのメニューから起動
2. Servers > PostgreSQL 15 > Databases で右クリック > Create > Database
3. database名を入力する

#### CUI

- [postgreSQLとSQL Shellを使ったpsqlコマンドでデータのリレーションをする方法を詳しく解説](https://awesomecatsis.com/postgresql-mac-psql-relation/)

1. SQL shellを起動
2. 最初にログインするDBを決める
   1. 元々postgresという名前のdbが作成されているので，Enter連打とパスワード入力でOK
3. 新規でdbを作る場合は，`CREATE DATABASE mytestdb;` のように入力
4. `\l`でdb一覧を表示できる
5. `\c mytestdb` のようにすればカレントのdbを変更できる




## Install(linux)

### 仮想環境作成

```sh
poetry add django==3.2.7 psycopg2-binary==2.9.1
source .venv/bin/activate
```

### Install postgresql

- https://www.postgresql.org/download/linux/ubuntu/

```sh
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql
```

インストールするとpostgresという名前のアカウントがUbuntuのユーザーに追加されている
```sh
$ cat /etc/passwd | grep postgres
postgres:x:112:120:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash

# postgresユーザーのパスワードを設定しておく
sudo passwd postgres
```


### create database

- [PostgreSQL サーバの起動と停止方法まとめ - Qiita](https://qiita.com/domodomodomo/items/12fe7555513de6b078db)
- [Ubuntu 20.04へのPostgreSQLのインストールおよび使用方法 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-20-04-ja)
- [ユーザーとパスワードの設定 ｜ PostgreSQLではじめるDB入門](http://db-study.com/archives/121)
- [PostgreSQLにおけるユーザの生成、またパスワード変更の行い方 - Qiita](https://qiita.com/neco0128_/items/2c3366b62d257a192961)
- [PostgreSQL | データベースを作成する(CREATE DATABASE)](https://www.javadrive.jp/postgresql/database/index2.html)

```sh
# サービスを起動
sudo service postgresql start

# postgresユーザーでpsqlコマンドを実行する
sudo -u postgres psql

# postgresqlサーバーに入り，プロンプトがpostgres=# のようになる
# ログインパスワードの設定 (postgres8 をパスワードとして設定する場合)
alter role postgres with password 'postgres8';
# パスワードを変えた場合，一回postgresqlサーバーを再起動させる必要があるらしい

# 新しいユーザーアカウント(role)を作成する場合
CREATE ROLE user01 WITH PASSWORD 'pass01';

# ユーザー一覧表示
\du
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 user01    | Cannot login                                               | {}

# create roleだけではログイン権限がないので，追加する
ALTER ROLE user01 LOGIN;

# DBの作成
create database private_diary;
create database private_diary owner pascal; # ownerを別に指定

# DB一覧表示
\l

# コンソールを抜ける
\q

# サーバーを終了させる場合
sudo service postgresql stop
```

## create project

```sh
django-admin startproject private_diary
cd private_diary
python manage.py runserver
```

### 設定変更

#### 言語・タイムゾーン

必要なら言語とタイムゾーンをusから日本に修正

projectディレクトリの`settings.py`
```python
LANGUAGE_CODE = 'ja'
TIME_ZONE = 'Asia/Tokyo'
```

#### database

デフォルトではSQLite3なので，postgreを使う場合は修正

ユーザー名とパスワードは環境変数から取得する実装の方が安全

projectディレクトリの`settings.py`
```python
import os

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'private_diary',
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': '',
        'PORT': '',
    }
}
```

#### ロギング設定

- `loggers`: ログ設定の大元．複数のhandlerを指定できる．ログレベルはひとつだけ指定
- `handlers`: ログ出力先の設定．フォーマットはformatterで指定
- `formatters`: ログの出力形式を指定

以下の設定ではdjango本体とアプリで使うログを分ける想定．

```python
LOGGING = {
    'version': 1,  # 1固定
    'disable_existing_loggers': False,

    'loggers': {
        # Djangoが利用するロガー
        'django': {
            'handlers': ['console'],
            'level': 'INFO',
        },
        # diaryアプリケーションが利用するロガー
        'diary': {
            'handlers': ['console'],
            'level': 'DEBUG',
        },
    },

    'handlers': {
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'dev',
        },
    },

    'formatters': {
        'dev': {
            'format': '\t'.join([
                '%(asctime)s',
                '[%(levelname)s]',
                '%(pathname)s(Line:%(lineno)d)',
                '%(message)s',
            ])
        },
    }
}
```

## create app

```sh
python manage.py startapp diary
```

### INSTALLED_APPS追加

projectディレクトリの`settings.py`の`INSTALLED_APPS`の中に追加したappのconfig(`'diary.apps.DiaryConfig'`)を追加する．

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    'diary.apps.DiaryConfig',
]
```

### ビューの作成

appのディレクトリの`views.py`

```python
from django.views import generic

class IndexView(generic.TemplateView):
    template_name = "index.html"
```


### テンプレートの作成

ビューで参照するhtmlファイルを作成

app内に`templates`ディレクトリを作成し，その中に`index.html`を作成．

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>トップページ</title>
</head>
<body>
    <h1>Hello World</h1>
</body>
</html>
```


### ルーティング設定

projectディレクトリの`urls.py`

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('diary.urls')),
]
```

app内にも`urls.py`を作成し，以下を記載

```python
from django.urls import path
from . import views

app_name = 'diary'
urlpatterns = [
    path('', views.IndexView.as_view(), name="index"),
]
```

## Webページをデザインする

### Bootstrapの導入

CSSでのデザインが楽にできるようにbootstrapを導入する．

- https://startbootstrap.com/theme/one-page-wonder
  - Free Downloadからダウンロード

プロジェクト直下にstaticディレクトリを作成し，その中にダウンロードしたデータのassetsとcssディレクトリをコピーする．

cssの`styles.css`は別のテンプレートと名前が衝突しないように`one-page-wonder.css`にファイル名を変更する．

静的ファイルを置いた場所をdjangoに設定するために，プロジェクトの`settings.py`に以下を追加．

```python
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static'),
)
```

### ベーステンプレートの作成

ヘッダー，フッター，ナビゲーションバーなど共通の部品をテンプレートとして作成する．

one-page-wonderのindex.htmlファイルをベースに`base.html`をappのtemplatesディレクトリに作成する．

変更点
- load staticを先頭に書いてstaticタグを使えるようにする
  - `{% load static %}`
- タイトルをtitleブロックに置き換え
  - `<title>{% block title %}{% endblock %}</title>`
- 画像などはすべてstaticタグで記述する方式に変更(hrefの部分を書き換えるだけ)
  - `<link rel="icon" type="image/x-icon" href="{% static 'assets/favicon.ico' %}" />`
- cssの設定を上書きするためのmystyle.cssをlink
  - `<link rel="stylesheet" type="text/css" href="{% static 'css/mystyle.css' %}">`
- ヘッダー，コンテンツ部分などはそれぞれ丸ごとブロックに置き換え



## Formのサンプル

### 概要・追加手順

参照が効くように作りたいので，以下の順番で作成

1. フォームを定義
2. ビューを追加
3. ルーティングを追加
4. htmlと(必要なら)cssを作成


役割としては
- Form
  - django側のコントロールオブジェクト本体
  - 送信ボタンとか自動的に付けてくれる
- View
  - Formとhtmlファイルのつなぎ
  - Form動作時の処理を定義
- ルーティング: URLとViewのつなぎ
- HTML: Fromを呼び出して，HTMLに組み込む


### 問い合わせ(forms.Form)

フォーム
```python
import os
from django import forms
from django.core.mail import EmailMessage

class InquiryForm(forms.Form):
    name = forms.CharField(label='お名前', max_length=30)
    email = forms.EmailField(label='メールアドレス')
    title = forms.CharField(label='タイトル', max_length=30)
    message = forms.CharField(label='メッセージ', widget=forms.Textarea)

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self.fields['name'].widget.attrs['class'] = 'form-control'
        self.fields['name'].widget.attrs['placeholder'] = 'お名前をここに入力してください。'
        self.fields['email'].widget.attrs['class'] = 'form-control'
        self.fields['email'].widget.attrs['placeholder'] = 'メールアドレスをここに入力してください。'
        self.fields['title'].widget.attrs['class'] = 'form-control'
        self.fields['title'].widget.attrs['placeholder'] = 'タイトルをここに入力してください。'
        self.fields['message'].widget.attrs['class'] = 'form-control'
        self.fields['message'].widget.attrs['placeholder'] = 'メッセージをここに入力してください。'

    def send_email(self):
        name = self.cleaned_data['name']
        email = self.cleaned_data['email']
        title = self.cleaned_data['title']
        message = self.cleaned_data['message']

        subject = 'お問い合わせ {}'.format(title)
        message = '送信者名: {0}\nメールアドレス: {1}\nメッセージ:\n{2}'.format(name, email, message)
        from_email = os.environ.get('FROM_EMAIL')
        to_list = [
            os.environ.get('FROM_EMAIL')
        ]
        cc_list = [
            email
        ]

        message = EmailMessage(subject=subject, body=message, from_email=from_email, to=to_list, cc=cc_list)
        message.send()
```

ビュー
```python
from django.contrib import messages

class InquiryView(generic.FormView):
    template_name = "inquiry.html"
    form_class = InquiryForm
    success_url = reverse_lazy('diary:inquiry') # このメンバにurlを入れると成功時に自動的にリダイレクトする

    # 入力値に問題がなかったら自動的に実行される
    def form_valid(self, form):
        form.send_email() # formクラスで自前で定義
        messages.success(self.request, 'メッセージを送信しました。')
        logger.info('Inquiry sent by {}'.format(form.cleaned_data['name']))
        return super().form_valid(form)
```

ルーティング
```python
from django.urls import path
from . import views

app_name = 'diary'
urlpatterns = [
    path('', views.IndexView.as_view(), name="index"),
    path('inquiry/', views.InquiryView.as_view(), name="inquiry"),
]
```

html
```html
{% extends 'base.html' %}
{% block contents %}
<div class="container">
    <form method="post">
        {% csrf_token %}
        {{ form.non_field_errors }}
        {% for field in form %}
            <div class="mb-4 col-8">
                <label for="{{ field.id_for_label }}" class="form-label">
                    <strong>{{ field.label_tag }}</strong>
                </label>
                {{ field }}
                {{ field.errors }}
            </div>
        {% endfor %}
        <button class="btn btn-primary" type="submit">送信</button>
    </form>
</div>
{% endblock %}
```

### Update


## Modelのサンプル

```python
from accounts.models import CustomUser
from django.db import models


class Diary(models.Model):
    """日記モデル"""
    user = models.ForeignKey(CustomUser, verbose_name='ユーザー', on_delete=models.PROTECT)
    title = models.CharField(verbose_name='タイトル', max_length=40)
    content = models.TextField(verbose_name='本文', blank=True, null=True)
    photo1 = models.ImageField(verbose_name='写真1', blank=True, null=True)
    photo2 = models.ImageField(verbose_name='写真2', blank=True, null=True)
    photo3 = models.ImageField(verbose_name='写真3', blank=True, null=True)
    created_at = models.DateTimeField(verbose_name='作成日時', auto_now_add=True)
    updated_at = models.DateTimeField(verbose_name='更新日時', auto_now=True)

    class Meta:
        verbose_name_plural = 'Diary'

    def __str__(self):
        return self.title
```



## Tips

### テンプレート


htmlの中にDjangoテンプレート言語の記法で値を埋め込んだり，条件分岐などの制御ができる仕組みのこと．


`parent.html`
```html
{% load static %}

<html lang="ja">
    <head>
        <meta charset="utf-8">
        <link rel="stylesheet" href="{% static 'css/mystyle.css' %}" type="text/css">
    </head>
    <body>
        {% block contents %}{% endblock contents %}
    </body>
</html>
```

ポイント
- `{% load static %}`を宣言すると，`settings.py`で設定した`STATICFILES_DIRS`のディレクトリからのパス指定で静的なファイルを読めるようになる
  - `<link rel="icon" type="image/x-icon" href="{% static 'assets/favicon.ico' %}" />`のようにstaticタグを付けて，相対パスを指定
  - staticタグを使いたい場合は，どのhtmlファイルでも毎回宣言する必要あり



`child.html`
```html
{% extends 'parent.html' %}

{% block contents %}
    {% if user.is_authenticated %}
        {% for item in object_list %}
            <a href="{% url 'application:bar' item.pk %}">
                <p>
                    {{ item.content|truncatechars: 20 }}
                </p>
            </a>
        {% empty %}
            <p>空です．</p>
        {% endfor %}
    {% else %}
        <p>ログインしてください．</p>
    {% endif %}
{% endblock contents %}
```

ポイント
- userはデフォルトで使える変数．アクセスユーザー情報が入っている
- `{% extends 'parent.html' %}`とすることで，`parent.html`の中身に埋め込みができる
- `{% url 'application:bar' item.pk %}`で`urls.py`のurlを逆引きできる
  - urls.pyの方は`path('bar/<int:pk>/', Views.BarView.as_view(), name="bar")`のように定義されている
  - このurlはint:pkの引数を渡す必要があるため，item.pkの形で渡している
- `object_list`はデフォルトで利用可能な変数．ビューから渡されるモデルオブジェクトのリスト
- `{{ item.content|truncatechars: 20 }}`: itemの中身を取り出している．`|<フィルタ>`の形でフィルタを使える．ここでは20桁超えたら表示切り詰め


### テンプレートのデフォルト変数

- [【Django】template上で使用できる user について - Qiita](https://qiita.com/taki_21/items/bc53a45379db2e58eba6)
  - settings.pyのTEMPLATE内のcontext_processorsの`django.contrib.auth.context_processors.auth`で追加されている
    - user属性があれば，userを返し，なければAnonymousUserを返す



### テンプレートタグ・フィルタ一覧

- https://docs.djangoproject.com/en/4.2/ref/templates/builtins/


### filed error

- [Django、フォームの表示方法まとめ - Narito Blog](https://blog.narito.ninja/detail/98/)

`{{ form.non_field_errors }}`は通常は表示されないが，何か問題があったとき(入力必須項目が空とか)にエラーメッセージを表示する

### HTMLタグ

- https://www.htmq.com/html/indexm.shtml

- a: ハイパーリンク，他のURLへのリンクを表示する
- link: 関連文書を指定．見た目には関係ない．主にcssやjsの読み込みに使用

### emmet

- https://docs.emmet.io/cheat-sheet/

**ID and class**

```html
#header
<div id="header"></div>

.title
<div class="title"></div>

form#search.wide
<form id="search" class="wide"></form>

p.class1.class2.class3
<p class="class1 class2 class3"></p>
```

**custom attribute**

```html
p[title="Hello world"]
<p title="Hello world"></p>

td[rowspan=2 colspan=3 title]
<td rowspan="2" colspan="3" title=""></td>

[a='value1' b="value2"]
<div a="value1" b="value2"></div>
```

### run debugger

- [【VSCode】Django のデバッグ環境の構築手順 | だえうホームページ](https://daeudaeu.com/vscode-django/#launchjson)

VSCodeでやる場合
- Pythonの拡張機能をVSCodeに入れる
- Run and Debugの画面で設定ファイルを作成するときにDjangoを選択し，manage.pyのパスを入れる(後から変更もできる)
- launch.jsonファイルが作成される
- envの項目を追加し，環境変数を追加
  - ただし，パスワードを登録してしまうとgitなどにpushできない

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Django",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/private_diary/manage.py",
            "args": [
                "runserver"
            ],
            "env": {
                "DB_USER": "postgres",
                "DB_PASSWORD": "postgres8"
            },
            "django": true,
            "justMyCode": true
        }
    ]
}
```

デバッグのサイドバーで再生ボタンを押すと処理が走る．

デバッグモードを使わない場合
```sh
# 環境変数が必要な場合は予め設定しておく
python manage.py runserver
```

#### peer error対応

postgresqlはデフォルト設定では実行するubuntuユーザーとpostgresqlのユーザー名が一致していないとエラーになる
- [[PostgreSQL]psql: FATAL: Peer authentication failed for user "postgres"エラーが出たときの対処法 - Qiita](https://qiita.com/Jackson123/items/266ca4a6165881f53ae7)

```sh
FATAL:  Peer authentication failed for user "postgres"
```

peer認証をOffにする
```sh
# 設定ファイルの場所を探す
sudo find / -path "/mnt" -prune -o -type f -name pg_hba.conf

# ファイルを編集する．localの項目のpeerをtrustに変更(3か所くらいある？)
sudo vim /etc/postgresql/15/main/pg_hba.conf

# サーバーを再起動
sudo service postgresql restart

# 動作確認
psql -q -U postgres
```
