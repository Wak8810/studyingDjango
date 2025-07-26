# Django学習レッスン: ミドルウェア - Lesson 3

## 目的

このレッスンでは、ミドルウェアにおける例外処理と、ビュー関数が呼び出される前後に特定の処理を挿入する方法を学びます。これにより、アプリケーション全体のエラーハンドリングや、リクエストごとの前処理・後処理を効率的に実装できるようになります。

## 概要

*   **`process_exception`**: ビュー関数が例外を発生させた場合に呼び出されるミドルウェアメソッドです。カスタムエラーページを表示したり、エラーをログに記録したりするのに使用できます。
*   **`process_view`**: ビュー関数が呼び出される直前に呼び出されるミドルウェアメソッドです。ビュー関数に渡される引数を変更したり、ビュー関数の実行を中断したりできます。

## 環境構築

このレッスンは、`middleware_lesson/lesson-3` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `middleware_lesson/lesson-3` ディレクトリに移動します。
    ```bash
    cd middleware_lesson/lesson-3
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: 例外処理ミドルウェアの作成

ビュー関数で例外が発生した場合に、カスタムエラーメッセージを表示するミドルウェアを作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `middleware_project` プロジェクト内に、`error_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp error_app
    ```

2.  **`settings.py` の設定**
    `middleware_project/settings.py` を開き、`INSTALLED_APPS` リストに `'error_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # middleware_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'error_app' を追加
        # 'error_app',
    ]
    ```

3.  **カスタムミドルウェアの作成**
    `error_app` ディレクトリ直下に、`middleware.py` という新しいファイルを作成し、以下の内容を記述してください。`ExceptionHandlingMiddleware` は、ビューで発生した例外をキャッチし、カスタムレスポンスを返します。

    ```python
    # error_app/middleware.py

    from django.http import HttpResponse

    class ExceptionHandlingMiddleware:
        def __init__(self, get_response):
            self.get_response = get_response

        def __call__(self, request):
            response = self.get_response(request)
            return response

        def process_exception(self, request, exception):
            # ビューで例外が発生した場合に呼び出される
            # print(f"例外が発生しました: {exception}")
            # return HttpResponse("カスタムエラーページ: 何か問題が発生しました。", status=500)
            pass
    ```

4.  **カスタムミドルウェアの登録**
    `middleware_project/settings.py` を開き、`MIDDLEWARE` リストに `'error_app.middleware.ExceptionHandlingMiddleware'` を追加してください。通常、例外処理ミドルウェアはリストの先頭に近い位置に配置されます。

    ```python
    # middleware_project/settings.py

    MIDDLEWARE = [
        # ここにExceptionHandlingMiddlewareを追加
        # 'error_app.middleware.ExceptionHandlingMiddleware',
        'django.middleware.security.SecurityMiddleware',
        # ... 既存のミドルウェア ...
    ]
    ```

5.  **テスト用のビューとURLの作成**
    `error_app/views.py` に例外を発生させるビューを作成し、`error_app/urls.py` でそのビューへのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # error_app/views.py

    from django.http import HttpResponse

    def raise_exception_view(request):
        raise ValueError("これはテスト用の例外です")
        return HttpResponse("このメッセージは表示されません")
    ```

    ```python
    # error_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('raise-error/', views.raise_exception_view, name='raise_error'),
    ]
    ```

    ```python
    # middleware_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('error_app/', include('error_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/error_app/raise-error/` にアクセスしてください。「カスタムエラーページ: 何か問題が発生しました。」と表示されれば成功です。また、Dockerコンテナのログにも例外メッセージが表示されていることを確認してください。

## 課題2: `process_view` の使用

ビュー関数が呼び出される直前に、リクエストにカスタムデータを追加するミドルウェアを作成しましょう。

### 手順

1.  **カスタムミドルウェアの作成**
    `error_app` ディレクトリ直下の `middleware.py` に、`ProcessViewMiddleware` という新しいミドルウェアを追加してください。

    ```python
    # error_app/middleware.py

    from django.http import HttpResponse

    class ProcessViewMiddleware:
        def __init__(self, get_response):
            self.get_response = get_response

        def __call__(self, request):
            response = self.get_response(request)
            return response

        def process_view(self, request, view_func, view_args, view_kwargs):
            # ビュー関数が呼び出される直前に実行される
            # request.custom_data = "これはミドルウェアから追加されたデータです。"
            # print("process_viewが実行されました")
            return None # Noneを返すと、通常のビュー処理が続行される
    ```

2.  **カスタムミドルウェアの登録**
    `middleware_project/settings.py` を開き、`MIDDLEWARE` リストに `'error_app.middleware.ProcessViewMiddleware'` を追加してください。

    ```python
    # middleware_project/settings.py

    MIDDLEWARE = [
        'error_app.middleware.ExceptionHandlingMiddleware',
        # ここにProcessViewMiddlewareを追加
        # 'error_app.middleware.ProcessViewMiddleware',
        'django.middleware.security.SecurityMiddleware',
        # ... 既存のミドルウェア ...
    ]
    ```

3.  **テスト用のビューの修正**
    `error_app/views.py` を開き、`test_process_view` という新しいビューを作成し、ミドルウェアから追加されたデータにアクセスするように修正してください。

    ```python
    # error_app/views.py

    from django.http import HttpResponse

    def test_process_view(request):
        # ミドルウェアから追加されたデータにアクセス
        # custom_data = getattr(request, 'custom_data', 'データなし')
        # return HttpResponse(f"ビューで受け取ったデータ: {custom_data}")
        pass
    ```

4.  **URL設定の追加**
    `error_app/urls.py` に、`test_process_view` へのURLを追加してください。

    ```python
    # error_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('raise-error/', views.raise_exception_view, name='raise_error'),
        # ここにtest_process_viewへのURLを追加
        # path('test-process-view/', views.test_process_view, name='test_process_view'),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/error_app/test-process-view/` にアクセスしてください。「ビューで受け取ったデータ: これはミドルウェアから追加されたデータです。」と表示されれば成功です。また、Dockerコンテナのログに「process_viewが実行されました」と表示されていることも確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
