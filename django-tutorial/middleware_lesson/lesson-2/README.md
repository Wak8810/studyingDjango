# Django学習レッスン: ミドルウェア - Lesson 2

## 目的

このレッスンでは、ミドルウェアがリクエストとレスポンスのオブジェクトにアクセスし、それらを変更する方法を学びます。また、特定の条件に基づいてリクエストをブロックするミドルウェアを作成します。

## 概要

ミドルウェアは、`process_request`, `process_view`, `process_exception`, `process_template_response`, `process_response` といったフックメソッドを通じて、リクエスト/レスポンスサイクルに深く介入できます。これにより、リクエストヘッダーの追加、ユーザーエージェントのチェック、エラーハンドリングのカスタマイズなどが可能になります。

## 環境構築

このレッスンは、`middleware_lesson/lesson-2` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `middleware_lesson/lesson-2` ディレクトリに移動します。
    ```bash
    cd middleware_lesson/lesson-2
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: リクエスト/レスポンスの変更とブロック

特定のユーザーエージェント（例: "BadBot"）からのアクセスをブロックするミドルウェアを作成しましょう。また、すべてのリクエストにカスタムヘッダーを追加するミドルウェアも作成します。

### 手順

1.  **Djangoアプリケーションの作成**
    `middleware_project` プロジェクト内に、`security_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp security_app
    ```

2.  **`settings.py` の設定**
    `middleware_project/settings.py` を開き、`INSTALLED_APPS` リストに `'security_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # middleware_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'security_app' を追加
        # 'security_app',
    ]
    ```

3.  **カスタムミドルウェアの作成**
    `security_app` ディレクトリ直下に、`middleware.py` という新しいファイルを作成し、以下の内容を記述してください。`UserAgentBlockMiddleware` は特定のユーザーエージェントをブロックし、`CustomHeaderMiddleware` はカスタムヘッダーを追加します。

    ```python
    # security_app/middleware.py

    from django.http import HttpResponseForbidden

    class UserAgentBlockMiddleware:
        def __init__(self, get_response):
            self.get_response = get_response

        def __call__(self, request):
            # リクエストのユーザーエージェントを取得
            user_agent = request.META.get('HTTP_USER_AGENT', '')
            # もしユーザーエージェントが"BadBot"だったらアクセスを拒否
            # if "BadBot" in user_agent:
            #     return HttpResponseForbidden("アクセスが拒否されました。")

            response = self.get_response(request)
            return response

    class CustomHeaderMiddleware:
        def __init__(self, get_response):
            self.get_response = get_response

        def __call__(self, request):
            response = self.get_response(request)
            # レスポンスにカスタムヘッダーを追加
            # response['X-Custom-Header'] = 'Django-Powered'
            return response
    ```

4.  **カスタムミドルウェアの登録**
    `middleware_project/settings.py` を開き、`MIDDLEWARE` リストに `'security_app.middleware.UserAgentBlockMiddleware'` と `'security_app.middleware.CustomHeaderMiddleware'` を追加してください。`UserAgentBlockMiddleware` は他のミドルウェアよりも前に配置すると良いでしょう。

    ```python
    # middleware_project/settings.py

    MIDDLEWARE = [
        # ここにUserAgentBlockMiddlewareを追加
        # 'security_app.middleware.UserAgentBlockMiddleware',
        'django.middleware.security.SecurityMiddleware',
        # ... 既存のミドルウェア ...
        # ここにCustomHeaderMiddlewareを追加
        # 'security_app.middleware.CustomHeaderMiddleware',
    ]
    ```

5.  **テスト用のビューとURLの作成**
    `security_app/views.py` にシンプルなビューを作成し、`security_app/urls.py` でそのビューへのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # security_app/views.py

    from django.http import HttpResponse

    def test_view(request):
        return HttpResponse("セキュリティテストページ")
    ```

    ```python
    # security_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('test/', views.test_view, name='test_page'),
    ]
    ```

    ```python
    # middleware_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('security_app/', include('security_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、

*   ブラウザで `http://localhost:8000/security_app/test/` にアクセスし、ページが表示されることを確認します。
*   開発者ツール（F12キーなどで開く）のネットワークタブで、レスポンスヘッダーに `X-Custom-Header: Django-Powered` が追加されていることを確認します。
*   `UserAgentBlockMiddleware` を有効にした後、`curl` コマンドなどでユーザーエージェントを偽装してアクセスし、アクセスが拒否されることを確認します。
    ```bash
    curl -A "BadBot" http://localhost:8000/security_app/test/
    ```

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
