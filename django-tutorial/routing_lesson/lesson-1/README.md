# Django学習レッスン: ルーティング - Lesson 1

## 目的

このレッスンでは、DjangoにおけるURLルーティングの仕組みを深く理解します。URLパターンとビュー関数のマッピング、アプリケーションごとのURL設定の分離、動的なURLの扱い方について学びます。

## ルーティングの概要

Djangoのルーティングは、ユーザーがアクセスしたURLをどのビュー関数（またはクラスベースビュー）で処理するかを決定するプロセスです。これは主に `urls.py` ファイルで定義されます。

*   **プロジェクトの `urls.py`**: プロジェクト全体のURL設定を管理し、各アプリケーションのURL設定をインクルードします。
*   **アプリケーションの `urls.py`**: 各アプリケーション固有のURLパターンを定義します。これにより、プロジェクトの規模が大きくなってもURL設定を整理しやすくなります。
*   **`path()` 関数**: URLパターン、対応するビュー関数、そしてそのURLに名前を付けるために使用されます。

## 環境構築

このレッスンは、`routing_lesson/lesson-1` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `routing_lesson/lesson-1` ディレクトリに移動します。
    ```bash
    cd routing_lesson/lesson-1
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: 基本的なルーティングの設定

`routing_project` プロジェクト内に `pages` というアプリケーションを作成し、`/about/` というURLで「このサイトについて」というテキストを表示するページを作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `routing_project` プロジェクト内に、`pages` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp pages
    ```

2.  **`settings.py` の設定**
    `routing_project/settings.py` を開き、`INSTALLED_APPS` リストに `'pages'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # routing_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'pages' を追加
        # 'pages',
    ]
    ```

3.  **ビューの作成**
    `pages/views.py` を開き、`about_view` という名前の関数を作成してください。この関数は、`HttpResponse` を使って「このサイトについて」という文字列を返します。

    ```python
    # pages/views.py

    from django.http import HttpResponse

    def about_view(request):
        # ここにコードを記述
        # return HttpResponse("このサイトについて")
        pass
    ```

4.  **アプリケーションのURL設定**
    `pages` ディレクトリ内に `urls.py` という新しいファイルを作成し、以下の内容を記述してください。

    ```python
    # pages/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        # ここにURLパターンを記述
        # path('about/', views.about_view, name='about'),
    ]
    ```

5.  **プロジェクトのURL設定**
    `routing_project/urls.py` を開き、`pages` アプリケーションのURL設定をインクルードしてください。

    ```python
    # routing_project/urls.py

    from django.contrib import admin
    from django.urls import path, include # includeをインポート

    urlpatterns = [
        path('admin/', admin.site.urls),
        # ここにpagesのURL設定をインクルード
        # path('', include('pages.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/about/` にアクセスしてください。「このサイトについて」と表示されれば成功です。

## 課題2: 動的なURLとパスパラメータ

ユーザーが `/greet/<name>/` のようにアクセスした際に、`<name>` の部分をビュー関数に渡して「こんにちは、<name>さん！」と表示するようにしましょう。

### 手順

1.  **ビューの修正**
    `pages/views.py` を開き、`greet_view` という名前の関数を作成してください。この関数は、URLから渡される `name` パラメータを受け取り、それを使ってレスポンスを生成します。

    ```python
    # pages/views.py

    from django.http import HttpResponse

    def greet_view(request, name):
        # ここにコードを記述
        # return HttpResponse(f"こんにちは、{name}さん！")
        pass
    ```

2.  **アプリケーションのURL設定の修正**
    `pages/urls.py` を開き、`greet_view` に対応するURLパターンを追加してください。動的な部分をキャプチャするために `<str:name>` を使用します。

    ```python
    # pages/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('about/', views.about_view, name='about'),
        # ここに動的なURLパターンを記述
        # path('greet/<str:name>/', views.greet_view, name='greet'),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/greet/DjangoUser/` にアクセスしてください。「こんにちは、DjangoUserさん！」と表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
