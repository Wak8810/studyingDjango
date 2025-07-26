# Django学習レッスン: ミドルウェア - Lesson 4

## 目的

このレッスンでは、ミドルウェアがテンプレートレスポンスを処理する方法と、ビュー関数が呼び出される前後に特定の処理を挿入する方法を学びます。これにより、アプリケーション全体のエラーハンドリングや、リクエストごとの前処理・後処理を効率的に実装できるようになります。

## 概要

*   **`process_template_response`**: ビューが `TemplateResponse` オブジェクトを返した場合に呼び出されるミドルウェアメソッドです。テンプレートがレンダリングされる前に、コンテキストを変更したり、テンプレートを変更したりできます。
*   **`process_response`**: すべてのレスポンスがユーザーに返される直前に呼び出されるミドルウェアメソッドです。レスポンスヘッダーの追加、コンテンツの圧縮、キャッシュの制御などに使用できます。

## 環境構築

このレッスンは、`middleware_lesson/lesson-4` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `middleware_lesson/lesson-4` ディレクトリに移動します。
    ```bash
    cd middleware_lesson/lesson-4
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: `process_template_response` の使用

テンプレートがレンダリングされる前に、テンプレートコンテキストにカスタムデータを追加するミドルウェアを作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `middleware_project` プロジェクト内に、`template_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp template_app
    ```

2.  **`settings.py` の設定**
    `middleware_project/settings.py` を開き、`INSTALLED_APPS` リストに `'template_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # middleware_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'template_app' を追加
        # 'template_app',
    ]
    ```

3.  **カスタムミドルウェアの作成**
    `template_app` ディレクトリ直下に、`middleware.py` という新しいファイルを作成し、以下の内容を記述してください。`TemplateContextMiddleware` は、テンプレートコンテキストに `app_name` を追加します。

    ```python
    # template_app/middleware.py

    from django.template.response import TemplateResponse

    class TemplateContextMiddleware:
        def __init__(self, get_response):
            self.get_response = get_response

        def __call__(self, request):
            response = self.get_response(request)
            return response

        def process_template_response(self, request, response):
            # レスポンスがTemplateResponseインスタンスの場合にのみ実行される
            if isinstance(response, TemplateResponse):
                # コンテキストにデータを追加
                # response.context_data['app_name'] = 'My Template App'
            return response
    ```

4.  **カスタムミドルウェアの登録**
    `middleware_project/settings.py` を開き、`MIDDLEWARE` リストに `'template_app.middleware.TemplateContextMiddleware'` を追加してください。

    ```python
    # middleware_project/settings.py

    MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        # ここにTemplateContextMiddlewareを追加
        # 'template_app.middleware.TemplateContextMiddleware',
        # ... 既存のミドルウェア ...
    ]
    ```

5.  **テスト用のビューとテンプレートの作成**
    `template_app/views.py` にシンプルなビューを作成し、`template_app/urls.py` でそのビューへのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # template_app/views.py

    from django.shortcuts import render

    def home_view(request):
        return render(request, 'template_app/home.html')
    ```

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>テンプレートテスト</title>
    </head>
    <body>
        <h1>ようこそ、{{ app_name }}へ！</h1>
        <p>これはテンプレートから表示されています。</p>
    </body>
    </html>
    ```

    ```python
    # template_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('home/', views.home_view, name='home'),
    ]
    ```

    ```python
    # middleware_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('template_app/', include('template_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/template_app/home/` にアクセスしてください。「ようこそ、My Template Appへ！」と表示されれば成功です。

## 課題2: `process_response` の使用

すべてのレスポンスにカスタムヘッダーを追加するミドルウェアを作成しましょう。

### 手順

1.  **カスタムミドルウェアの作成**
    `template_app` ディレクトリ直下の `middleware.py` に、`AddCustomHeaderMiddleware` という新しいミドルウェアを追加してください。

    ```python
    # template_app/middleware.py

    class AddCustomHeaderMiddleware:
        def __init__(self, get_response):
            self.get_response = get_response

        def __call__(self, request):
            response = self.get_response(request)
            # レスポンスにカスタムヘッダーを追加
            # response['X-Powered-By'] = 'Django'
            return response
    ```

2.  **カスタムミドルウェアの登録**
    `middleware_project/settings.py` を開き、`MIDDLEWARE` リストに `'template_app.middleware.AddCustomHeaderMiddleware'` を追加してください。通常、`process_response` を持つミドルウェアはリストの最後の方に配置されます。

    ```python
    # middleware_project/settings.py

    MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        # ... 既存のミドルウェア ...
        # ここにAddCustomHeaderMiddlewareを追加
        # 'template_app.middleware.AddCustomHeaderMiddleware',
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/template_app/home/` にアクセスしてください。開発者ツール（F12キーなどで開く）のネットワークタブで、レスポンスヘッダーに `X-Powered-By: Django` が追加されていることを確認します。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
