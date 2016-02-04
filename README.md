study-django
=====

# ドキュメント
- [Python Django入門 (1) - Qiita](http://qiita.com/kaki_k/items/511611cadac1d0c69c54)
- https://docs.djangoproject.com/en/1.9/

# MacでPythonの環境構築

## python2, 3をそれぞれinstall
```
$ python -V
Python 2.7.10

$ brew install python3
Python 3.5.1
```

## python2でvirtualenvとvirtualenvwrapperを入れる

```
$ sudo easy_install pip

# update
$ sudo pip install --upgrade pip

$ sudo -H pip install virtualenv virtualenvwrapper
$ mkdir ~/.virtualenvs
```

## direnvでvirtualenvwrappeerを走らせるようにする

```
$ cd /path/to/study-django
$ direnv edit .

export WORKON_HOME=$HOME/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh

:wq

$ direnv allow
```

## mkvirtualenvで仮想環境を作る

- env1という名前
- --no-site-packageは、ベースとなるpythonのsite-packageを受け継がない
- --python オプションで使用するpythonを指定
- (env1)とターミナルに出てればOK
- workon コマンドで仮想環境の一覧が出る
- workon 環境名 で仮想環境を選択
- deactive コマンドで仮想環境を無効にしてグローバル環境に戻る
- rmvitualenv で仮想環境に入れたパッケージごと削除
- freezeで仮想環境にインストールされたパッケージを確認

```
$ mkvirtualenv --no-site-package --python /usr/local/bin/python3 env1

$ workon
env1

$ workon env1
$ deactive
$ rmvirtualenv env1
$ pip freeze -l
```

# Djangoをインストール

```
$ workon env1
$ pip install django
```


# プロジェクトを作成

```
$ django-admin.py startproject mysite
$ cd mysite

$ python manage.py migrate
```

- django-admin.py を実行してみると、いろいろサブコマンドがある


## とりあえずサーバ起動まで

- localhost:8000
- Ctrl-Cで停止

```
$ python manage.py migrate
$ python manage.py runserver
```

- ここまでやるとディレクトリ構造はこんな感じになる
- sqliteのファイルができた

```
$ tree -F .
.
└── mysite/
    ├── db.sqlite3
    ├── manage.py*
    └── mysite/
        ├── __init__.py
        ├── __pycache__/
        │   ├── __init__.cpython-35.pyc
        │   ├── settings.cpython-35.pyc
        │   ├── urls.cpython-35.pyc
        │   └── wsgi.cpython-35.pyc
        ├── settings.py
        ├── urls.py
        └── wsgi.py
```


# Djangoでつくってみる

## データベースのセットアップ

- mysite/settings.py にすでにsqlite3の設定がされている

```
$ python manage.py migrate
$ python manage.py createsuperuser
```

## 言語とタイムゾーンの設定

- mysite/setings.py

```python
# LANGUAGE_CODE = 'en-us'
LANGUAGE_CODE = 'ja'

# TIME_ZONE = 'UTC'
TIME_ZONE = 'Asia/Tokyo'
```

## アプリケーションの作成

- 先のプロジェクト作成、`$ django-admin.py startproject mysite` との違いがよくわからないけど

```
$ python manage.py startapp cms
```

## 作ったアプリケーションをインストールしたことをプロジェクトにしらせる

- mysite/setings.py

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'cms',
]
```

## モデル

### 作成

- mysite/cms/models.py
- 複数のモデルを書いているけど、モデルごとにファイルをわけない？

### マイグレーション

- マイグレーションファイルの作成

```
$ python manage.py makemigrations cms
```

- マイグレーション内容の確認

```
$ python manage.py sqlmigrate cms 0001
BEGIN;
--
-- Create model Book
--
CREATE TABLE "cms_book" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "name" varchar(255) NOT NULL, "publisher" varchar(255) NOT NULL, "page" integer NOT NULL);
--
-- Create model Impression
--
CREATE TABLE "cms_impression" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "comment" text NOT NULL, "book_id" integer NOT NULL REFERENCES "cms_book" ("id"));
CREATE INDEX "cms_impression_0a4572cc" ON "cms_impression" ("book_id");

COMMIT;
```

- まだ適用していないマイグレーションファイルをDBへ適用

```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: sessions, admin, auth, contenttypes, cms
Running migrations:
  Rendering model states... DONE
```


## 管理サイト

- サーバ起動
- http://localhost:8000/admin/
- createsuperuserでつくったユーザでログイン

```
$ python manage.py runserver
```


### さっき作ったモデルをadmin上で編集できるようにする

- mysite/cms/admin.py

```python
from django.contrib import admin
from cms.models import Book, Impression

admin.site.register(Book)
admin.site.register(Impression)
```



### 管理サイトの一覧に表示する項目を増やす

- mysite/cms/admin.py
```
from django.contrib import admin
from cms.models import Book, Impression

# admin.site.register(Book)
# admin.site.register(Impression)

class BookAdmin(admin.ModelAdmin):
    list_display = ('id', 'name', 'publisher', 'page',)  # 一覧に出したい項目
    list_display_links = ('id', 'name',)  # 修正リンクでクリックできる項目
admin.site.register(Book, BookAdmin)

class ImpressionAdmin(admin.ModelAdmin):
    list_display = ('id', 'comment',)
    list_display_links = ('id', 'comment',)
admin.site.register(Impression, ImpressionAdmin)
```


## css と js

### 静的ファイルの置き場所

- static ディレクトリを作る
- Bootstrap とjQueryをダウンロードしてstaticディレクトリへ配置

```
$ mkdir mysite/static
```

- static というURLが静的ファイルを置くディレクトリであること、mysite/mysite/settings.py に定義されている
- static/ は、 mysite/cms/static -> mysite/static の順に見に行く
- mysite/mysite は全体のアプリケーション共通部分、mysite/cmsはモジュール的な役割なのかも

### django-bootstrap-form の利用

```
$ pip install django-bootstrap-form
$ pip freeze -l
Django==1.9.2
django-bootstrap-form==3.2
wheel==0.26.0
```

- INSTALLED_APPSへ追加

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'bootstrapform', # django-bootstrap-form
    'cms',
]
```

