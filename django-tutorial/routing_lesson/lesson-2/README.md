# Django学習レッスン: ルーティング - Lesson 2

## 目的

このレッスンでは、DjangoのURLルーティングにおける正規表現を使ったURLパターンマッチングと、名前空間（Namespacing）を使ってURLを整理する方法を学びます。

## 概要

*   **正規表現 (Regular Expressions)**: `re_path()` 関数を使うことで、より複雑なURLパターンを定義できます。例えば、特定の形式のIDや日付を含むURLにマッチさせることができます。
*   **名前空間 (Namespacing)**: 複数のアプリケーションで同じ名前のURLパターン（例: `detail`）がある場合に、名前の衝突を避けるために使用します。これにより、`app_name:url_name` の形式でURLを逆引きできるようになります。

## 環境構築

このレッスンは、`routing_lesson/lesson-2` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `routing_lesson/lesson-2` ディレクトリに移動します。
    ```bash
    cd routing_lesson/lesson-2
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: 正規表現を使ったURLパターン

ユーザーが `/articles/2023/12/` のようにアクセスした際に、年と月をビュー関数に渡して表示するようにしましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `routing_project` プロジェクト内に、`blog` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp blog
    ```

2.  **`settings.py` の設定**
    `routing_project/settings.py` を開き、`INSTALLED_APPS` リストに `'blog'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # routing_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'blog' を追加
        # 'blog',
    ]
    ```

3.  **ビューの作成**
    `blog/views.py` を開き、`article_archive` という名前の関数を作成してください。この関数は、URLから渡される `year` と `month` パラメータを受け取り、それを使ってレスポンスを生成します。

    ```python
    # blog/views.py

    from django.http import HttpResponse

    def article_archive(request, year, month):
        # ここにコードを記述
        # return HttpResponse(f"{year}年{month}月の記事一覧")
        pass
    ```

4.  **アプリケーションのURL設定**
    `blog` ディレクトリ内に `urls.py` という新しいファイルを作成し、`article_archive` に対応するURLパターンを `re_path()` を使って追加してください。年（4桁の数字）と月（1桁または2桁の数字）をキャプチャするように正規表現を記述します。

    ```python
    # blog/urls.py

    from django.urls import re_path
    from . import views

    urlpatterns = [
        # ここに正規表現を使ったURLパターンを記述
        # re_path(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{1,2})/', views.article_archive, name='article_archive'),
    ]
    ```

5.  **プロジェクトのURL設定**
    `routing_project/urls.py` を開き、`blog` アプリケーションのURL設定をインクルードしてください。

    ```python
    # routing_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        # ここにblogのURL設定をインクルード
        # path('', include('blog.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/articles/2024/07/` にアクセスしてください。「2024年7月の記事一覧」と表示されれば成功です。

## 課題2: 名前空間 (Namespacing)

複数のアプリケーションで同じ名前のURLパターンがある場合に、名前空間を使って衝突を避ける方法を学びましょう。

### 手順

1.  **`blog/urls.py` に名前空間を設定**
    `blog/urls.py` を開き、`app_name` 変数を追加して名前空間を定義してください。

    ```python
    # blog/urls.py

    from django.urls import re_path
    from . import views

    app_name = 'blog' # ここに名前空間を定義

    urlpatterns = [
        re_path(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{1,2})/', views.article_archive, name='article_archive'),
    ]
    ```

2.  **プロジェクトのURL設定の修正**
    `routing_project/urls.py` を開き、`blog` アプリケーションをインクルードする際に、`namespace` を指定してください。

    ```python
    # routing_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        # ここで名前空間を指定してインクルード
        # path('', include(('blog.urls', 'blog'), namespace='blog')),
    ]
    ```

3.  **テンプレートでのURL逆引きの修正**
    （この課題ではテンプレートは作成しませんが、概念として）テンプレート内でURLを逆引きする際に、`{% url 'blog:article_archive' year=2024 month=7 %}` のように名前空間を指定して呼び出すようになります。

### 確認

ブラウザで `http://localhost:8000/articles/2024/07/` にアクセスして、引き続きページが表示されれば、名前空間の設定は正しく行われています。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
