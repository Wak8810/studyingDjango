# Django学習レッスン: セッション - Lesson 5

## 目的

このレッスンでは、セッションのバックエンドとしてキャッシュを使用する方法と、セッションデータを永続化しない一時的なセッションの利用について学びます。これにより、セッション管理のパフォーマンスと柔軟性をさらに向上させることができます。

## 概要

*   **キャッシュバックエンド**: セッションデータをデータベースではなく、キャッシュ（例: Redis, Memcached）に保存することで、I/O負荷を軽減し、パフォーマンスを向上させることができます。Djangoは、様々なキャッシュバックエンドをサポートしています。
*   **一時的なセッション**: ユーザーがブラウザを閉じると自動的に破棄されるセッションです。ログイン状態を保持する必要がない、一時的な情報（例: フォームの入力途中データ）を保存するのに適しています。

## 環境構築

このレッスンは、`session_lesson/lesson-5` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `session_lesson/lesson-5` ディレクトリに移動します。
    ```bash
    cd session_lesson/lesson-5
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

3.  **データベースマイグレーションの実行**
    セッション機能はデータベーステーブルを使用するため、以下のコマンドでマイグレーションを実行し、必要なテーブルを作成してください。
    ```bash
    docker compose exec web python manage.py migrate
    ```

## 課題1: キャッシュバックエンドへの変更

セッションデータをデータベースではなく、ローカルメモリキャッシュに保存するように設定を変更しましょう。

### 手順

1.  **`settings.py` の設定変更**
    `session_project/settings.py` を開き、`SESSION_ENGINE` を `'django.contrib.sessions.backends.cache'` に変更してください。また、`CACHES` 設定を追加して、ローカルメモリキャッシュを定義します。

    ```python
    # session_project/settings.py

    # ... 既存の設定 ...

    # セッションバックエンドをキャッシュに変更
    # SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

    # キャッシュ設定
    # CACHES = {
    #     'default': {
    #         'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    #         'LOCATION': 'unique-snowflake',
    #     }
    # }
    ```

2.  **セッションデータを設定するビューの作成**
    `session_project` プロジェクト内に、`cache_session_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp cache_session_app
    ```

3.  **`settings.py` の設定**
    `session_project/settings.py` を開き、`INSTALLED_APPS` リストに `'cache_session_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # session_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'cache_session_app' を追加
        # 'cache_session_app',
    ]
    ```

4.  **ビューの作成**
    `cache_session_app/views.py` を開き、`set_cache_session_view` という名前の関数を作成してください。この関数は、セッションにデータを保存し、表示します。

    ```python
    # cache_session_app/views.py

    from django.shortcuts import render

    def set_cache_session_view(request):
        # セッションにデータを保存
        # request.session['cache_data'] = 'これはキャッシュに保存されたデータです。'
        # request.session['cache_counter'] = request.session.get('cache_counter', 0) + 1
        
        # context = {
        #     'cache_data': request.session.get('cache_data'),
        #     'cache_counter': request.session.get('cache_counter'),
        # }
        # return render(request, 'cache_session_app/cache_session_status.html', context)
        pass
    ```

5.  **テンプレートファイルの作成**
    `cache_session_app/templates/cache_session_app/` ディレクトリを作成し、その中に `cache_session_status.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>キャッシュセッション</title>
    </head>
    <body>
        <h1>キャッシュセッションステータス</h1>
        <p>Cache Data: {{ cache_data }}</p>
        <p>Cache Counter: {{ cache_counter }}</p>
    </body>
    </html>
    ```

6.  **URL設定の追加**
    `cache_session_app/urls.py` を作成し、`set_cache_session_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # cache_session_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('cache/', views.set_cache_session_view, name='set_cache_session'),
    ]
    ```

    ```python
    # session_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('cache_session_app/', include('cache_session_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/cache_session_app/cache/` にアクセスしてください。ページをリロードするたびにカウンターが増えることを確認します。Dockerコンテナを再起動すると、ローカルメモリキャッシュはクリアされるため、カウンターがリセットされることを確認してください。

## 課題2: 一時的なセッションの利用

ユーザーがブラウザを閉じると自動的に破棄される一時的なセッションを利用しましょう。

### 手順

1.  **ビューの修正**
    `cache_session_app/views.py` を開き、`set_temp_session_view` という名前の関数を作成してください。この関数は、`request.session.set_expiry(0)` を使ってセッションを一時的なものとして設定します。

    ```python
    # cache_session_app/views.py

    from django.shortcuts import render

    def set_temp_session_view(request):
        # セッションを一時的なものとして設定（ブラウザを閉じると期限切れ）
        # request.session.set_expiry(0)
        # request.session['temp_data'] = 'これは一時的なセッションデータです。'
        
        # context = {
        #     'temp_data': request.session.get('temp_data'),
        # }
        # return render(request, 'cache_session_app/temp_session_status.html', context)
        pass
    ```

2.  **テンプレートファイルの作成**
    `cache_session_app/templates/cache_session_app/` ディレクトリを作成し、その中に `temp_session_status.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>一時セッション</title>
    </head>
    <body>
        <h1>一時セッションステータス</h1>
        <p>Temp Data: {{ temp_data }}</p>
        <p>ブラウザを閉じるとこのデータは失われます。</p>
    </body>
    </html>
    ```

3.  **URL設定の追加**
    `cache_session_app/urls.py` に、`set_temp_session_view` へのURLを追加してください。

    ```python
    # cache_session_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('cache/', views.set_cache_session_view, name='set_cache_session'),
        path('temp/', views.set_temp_session_view, name='set_temp_session'),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/cache_session_app/temp/` にアクセスしてください。データが表示されることを確認します。その後、ブラウザを完全に閉じてから再度アクセスし、データが失われていることを確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
