# Django学習レッスン: ミドルウェア - Lesson 1

## 目的

このレッスンでは、Djangoのミドルウェアの概念と、それがリクエスト/レスポンスサイクルにどのように介入するかを学びます。カスタムミドルウェアを作成し、その動作を確認します。

## ミドルウェアの概要

ミドルウェアは、Djangoのリクエスト/レスポンス処理の途中で、グローバルな処理を実行するためのフック（割り込み点）を提供します。例えるなら、ウェブサイトの「警備員」や「受付係」のようなものです。

*   **リクエスト処理**: ユーザーからのリクエストがビューに届く前に、認証チェック、セッション管理、セキュリティ対策、ログ記録などの処理を行うことができます。
*   **レスポンス処理**: ビューが生成したレスポンスがユーザーに送られる前に、圧縮、キャッシュ、ヘッダーの追加などの処理を行うことができます。

Djangoには、セッション管理やCSRF保護など、多くの標準ミドルウェアが組み込まれており、これらは `settings.py` の `MIDDLEWARE` 設定で管理されます。

## 環境構築

このレッスンは、`middleware_lesson/lesson-1` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `middleware_lesson/lesson-1` ディレクトリに移動します。
    ```bash
    cd middleware_lesson/lesson-1
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: カスタムミドルウェアの作成と登録

リクエストが来るたびにコンソールにメッセージを表示する、シンプルなカスタムミドルウェアを作成し、それをDjangoプロジェクトに登録しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `middleware_project` プロジェクト内に、`my_middleware_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp my_middleware_app
    ```

2.  **`settings.py` の設定**
    `middleware_project/settings.py` を開き、`INSTALLED_APPS` リストに `'my_middleware_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # middleware_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'my_middleware_app' を追加
        # 'my_middleware_app',
    ]
    ```

3.  **カスタムミドルウェアの作成**
    `my_middleware_app` ディレクトリ直下に、`middleware.py` という新しいファイルを作成し、以下の内容を記述してください。

    ```python
    # my_middleware_app/middleware.py

    class SimpleLogMiddleware:
        def __init__(self, get_response):
            self.get_response = get_response

        def __call__(self, request):
            # リクエストがビューに到達する前に実行されるコード
            # print("リクエストがビューに到達する前です！")

            response = self.get_response(request)

            # レスポンスがユーザーに返される前に実行されるコード
            # print("レスポンスがユーザーに返される前です！")

            return response
    ```

4.  **カスタムミドルウェアの登録**
    `middleware_project/settings.py` を開き、`MIDDLEWARE` リストに `'my_middleware_app.middleware.SimpleLogMiddleware'` を追加してください。リストのどこに追加するかによって、実行される順番が変わります。今回は、既存のミドルウェアの後に追記してみましょう。

    ```python
    # middleware_project/settings.py

    MIDDLEWARE = [
        # ... 既存のミドルウェア ...
        # ここにカスタムミドルウェアを追加
        # 'my_middleware_app.middleware.SimpleLogMiddleware',
    ]
    ```

5.  **テスト用のビューとURLの作成**
    `my_middleware_app/views.py` にシンプルなビューを作成し、`my_middleware_app/urls.py` でそのビューへのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # my_middleware_app/views.py

    from django.http import HttpResponse

    def test_view(request):
        return HttpResponse("ミドルウェアテストページ")
    ```

    ```python
    # my_middleware_app/urls.py

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
        path('my_middleware_app/', include('my_middleware_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/my_middleware_app/test/` にアクセスしてください。その後、Dockerコンテナのログを確認します。

```bash
docker compose logs web
```

ログに「リクエストがビューに到達する前です！」と「レスポンスがユーザーに返される前です！」のメッセージが表示されていれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
