# Django学習レッスン: ルーティング - Lesson 3

## 目的

このレッスンでは、DjangoのURLルーティングにおけるリダイレクトと、ビュー関数に引数を渡す方法を学びます。また、URLの逆引き（URL Reversing）をより実践的に活用する方法を理解します。

## 概要

*   **リダイレクト**: あるURLへのアクセスを別のURLに転送する機能です。例えば、古いURLから新しいURLへユーザーを誘導したり、フォーム送信後に別のページへ遷移させたりする際に使用します。
*   **URLの逆引き (URL Reversing)**: URLパターンに付けた名前を使って、実際のURLを動的に生成する機能です。これにより、URLの変更に強く、コードの可読性も向上します。

## 環境構築

このレッスンは、`routing_lesson/lesson-3` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `routing_lesson/lesson-3` ディレクトリに移動します。
    ```bash
    cd routing_lesson/lesson-3
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: リダイレクトの実装

`/old-path/` にアクセスした際に、`/new-path/` にリダイレクトされるように設定しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `routing_project` プロジェクト内に、`redirect_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp redirect_app
    ```

2.  **`settings.py` の設定**
    `routing_project/settings.py` を開き、`INSTALLED_APPS` リストに `'redirect_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # routing_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'redirect_app' を追加
        # 'redirect_app',
    ]
    ```

3.  **ビューの作成**
    `redirect_app/views.py` を開き、`new_path_view` という名前の関数を作成してください。この関数は、シンプルなHTTPレスポンスを返します。

    ```python
    # redirect_app/views.py

    from django.http import HttpResponse

    def new_path_view(request):
        return HttpResponse("新しいパスのページです！")
    ```

4.  **アプリケーションのURL設定**
    `redirect_app` ディレクトリ内に `urls.py` という新しいファイルを作成し、`new_path_view` へのURLと、リダイレクト元のURLを設定してください。

    ```python
    # redirect_app/urls.py

    from django.urls import path
    from django.views.generic.base import RedirectView # RedirectViewをインポート
    from . import views

    urlpatterns = [
        path('new-path/', views.new_path_view, name='new_path'),
        # /old-path/ にアクセスしたら /new-path/ にリダイレクト
        # path('old-path/', RedirectView.as_view(url='/redirect_app/new-path/'), name='old_path'),
    ]
    ```

5.  **プロジェクトのURL設定**
    `routing_project/urls.py` を開き、`redirect_app` のURL設定をインクルードしてください。

    ```python
    # routing_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('redirect_app/', include('redirect_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/redirect_app/old-path/` にアクセスしてください。自動的に `http://localhost:8000/redirect_app/new-path/` にリダイレクトされ、「新しいパスのページです！」と表示されれば成功です。

## 課題2: URLの逆引きとビューへの引数渡し

テンプレート内でURLの逆引きを行い、ビュー関数に引数を渡す方法を学びましょう。

### 手順

1.  **ビューの修正**
    `redirect_app/views.py` を開き、`show_number_view` という名前の関数を作成してください。この関数は、URLから渡される `number` パラメータを受け取り、それを表示します。

    ```python
    # redirect_app/views.py

    from django.http import HttpResponse

    def show_number_view(request, number):
        return HttpResponse(f"表示された数字: {number}")
    ```

2.  **アプリケーションのURL設定の修正**
    `redirect_app/urls.py` を開き、`show_number_view` に対応するURLパターンを追加してください。動的な部分をキャプチャするために `<int:number>` を使用します。

    ```python
    # redirect_app/urls.py

    from django.urls import path
    from django.views.generic.base import RedirectView
    from . import views

    urlpatterns = [
        path('new-path/', views.new_path_view, name='new_path'),
        path('old-path/', RedirectView.as_view(url='/redirect_app/new-path/'), name='old_path'),
        # ここにshow_number_viewへのURLを追加
        # path('show/<int:number>/', views.show_number_view, name='show_number'),
    ]
    ```

3.  **テンプレートの作成**
    `redirect_app/templates/redirect_app/` ディレクトリを作成し、その中に `number_link.html` という名前で以下の内容を記述してください。`show_number` URLを逆引きしてリンクを生成します。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>数字表示リンク</title>
    </head>
    <body>
        <h1>数字表示リンク</h1>
        <p>
            {% comment %} ここにURLの逆引きを記述 {% endcomment %}
            {# 例: <a href="{% url 'show_number' number=123 %}">数字123を表示</a> #}
        </p>
    </body>
    </html>
    ```

4.  **ビューの追加**
    `redirect_app/views.py` に、`number_link_view` という名前の関数を作成してください。この関数は、`number_link.html` テンプレートをレンダリングします。

    ```python
    # redirect_app/views.py

    from django.shortcuts import render

    def number_link_view(request):
        return render(request, 'redirect_app/number_link.html')
    ```

5.  **URL設定の追加**
    `redirect_app/urls.py` に、`number_link_view` へのURLを追加してください。

    ```python
    # redirect_app/urls.py

    from django.urls import path
    from django.views.generic.base import RedirectView
    from . import views

    urlpatterns = [
        path('new-path/', views.new_path_view, name='new_path'),
        path('old-path/', RedirectView.as_view(url='/redirect_app/new-path/'), name='old_path'),
        path('show/<int:number>/', views.show_number_view, name='show_number'),
        # ここにnumber_link_viewへのURLを追加
        # path('link/', views.number_link_view, name='number_link'),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/redirect_app/link/` にアクセスしてください。「数字123を表示」というリンクが表示され、クリックすると「表示された数字: 123」と表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
