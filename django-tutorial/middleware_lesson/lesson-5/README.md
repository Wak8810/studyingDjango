# Django学習レッスン: ミドルウェア - Lesson 5

## 目的

このレッスンでは、ミドルウェアの実行順序の重要性と、複数のミドルウェアが連携して動作する例を学びます。また、ミドルウェアのデバッグ方法についても触れます。

## 概要

`settings.py` の `MIDDLEWARE` リストに記述されたミドルウェアは、上から順にリクエストを処理し、下から順にレスポンスを処理します。この順序は非常に重要で、ミドルウェアの機能によっては配置場所が厳密に定められているものもあります。

*   **リクエスト処理**: リストの上から下へ実行されます。
*   **レスポンス処理**: リストの下から上へ実行されます。

## 環境構築

このレッスンは、`middleware_lesson/lesson-5` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `middleware_lesson/lesson-5` ディレクトリに移動します。
    ```bash
    cd middleware_lesson/lesson-5
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: ミドルウェアの実行順序の確認

複数のカスタムミドルウェアを作成し、それらの実行順序がどのようにリクエスト/レスポンスサイクルに影響するかを確認しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `middleware_project` プロジェクト内に、`order_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp order_app
    ```

2.  **`settings.py` の設定**
    `middleware_project/settings.py` を開き、`INSTALLED_APPS` リストに `'order_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # middleware_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'order_app' を追加
        # 'order_app',
    ]
    ```

3.  **カスタムミドルウェアの作成**
    `order_app` ディレクトリ直下に、`middleware.py` という新しいファイルを作成し、以下の内容を記述してください。`MiddlewareA` と `MiddlewareB` は、それぞれリクエストとレスポンスの処理時にメッセージを出力します。

    ```python
    # order_app/middleware.py

    class MiddlewareA:
        def __init__(self, get_response):
            self.get_response = get_response

        def __call__(self, request):
            print("MiddlewareA: リクエスト処理開始")
            response = self.get_response(request)
            print("MiddlewareA: レスポンス処理終了")
            return response

    class MiddlewareB:
        def __init__(self, get_response):
            self.get_response = get_response

        def __call__(self, request):
            print("MiddlewareB: リクエスト処理開始")
            response = self.get_response(request)
            print("MiddlewareB: レスポンス処理終了")
            return response
    ```

4.  **カスタムミドルウェアの登録**
    `middleware_project/settings.py` を開き、`MIDDLEWARE` リストに `'order_app.middleware.MiddlewareA'` と `'order_app.middleware.MiddlewareB'` を追加してください。それぞれのミドルウェアの配置順序を変えて、ログの出力順序がどのように変わるかを確認しましょう。

    ```python
    # middleware_project/settings.py

    MIDDLEWARE = [
        # ... 既存のミドルウェア ...
        # ここにMiddlewareAとMiddlewareBを追加し、順序を変えて試す
        # 'order_app.middleware.MiddlewareA',
        # 'order_app.middleware.MiddlewareB',
    ]
    ```

5.  **テスト用のビューとURLの作成**
    `order_app/views.py` にシンプルなビューを作成し、`order_app/urls.py` でそのビューへのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # order_app/views.py

    from django.http import HttpResponse

    def test_view(request):
        print("ビュー関数が実行されました")
        return HttpResponse("ミドルウェア順序テストページ")
    ```

    ```python
    # order_app/urls.py

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
        path('order_app/', include('order_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/order_app/test/` にアクセスしてください。その後、Dockerコンテナのログを確認します。

```bash
docker compose logs web
```

`MIDDLEWARE` リストでのミドルウェアの配置順序と、ログに出力されるメッセージの順序を比較し、リクエスト処理とレスポンス処理で逆順に実行されることを確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
