# Django学習レッスン: ルーティング - Lesson 5

## 目的

このレッスンでは、DjangoのURLルーティングにおける高度な機能である「ビューの引数としての辞書渡し」と「URLのプレフィックス」について学びます。これにより、より柔軟で再利用可能なURL設定を構築できるようになります。

## 概要

*   **ビューの引数としての辞書渡し**: `path()` 関数や `re_path()` 関数で、ビュー関数に追加の引数を辞書形式で渡すことができます。これは、同じビュー関数を異なる設定で再利用したい場合に便利です。
*   **URLのプレフィックス**: `include()` 関数を使って、特定のURLパスの下に別のURL設定をマウントする際に、共通のプレフィックスを付けることができます。これにより、URLの構造を整理し、管理しやすくなります。

## 環境構築

このレッスンは、`routing_lesson/lesson-5` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `routing_lesson/lesson-5` ディレクトリに移動します。
    ```bash
    cd routing_lesson/lesson-5
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: ビューへの追加引数渡し

同じビュー関数を使いながら、URLによって異なるメッセージを表示するようにしましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `routing_project` プロジェクト内に、`message_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp message_app
    ```

2.  **`settings.py` の設定**
    `routing_project/settings.py` を開き、`INSTALLED_APPS` リストに `'message_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # routing_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'message_app' を追加
        # 'message_app',
    ]
    ```

3.  **ビューの作成**
    `message_app/views.py` を開き、`display_message` という名前の関数を作成してください。この関数は、`message` という引数を受け取り、それを表示します。

    ```python
    # message_app/views.py

    from django.http import HttpResponse

    def display_message(request, message):
        # ここにコードを記述
        # return HttpResponse(f"表示されたメッセージ: {message}")
        pass
    ```

4.  **アプリケーションのURL設定**
    `message_app` ディレクトリ内に `urls.py` という新しいファイルを作成し、`display_message` ビューへのURLを複数設定してください。`kwargs` 引数を使って、異なるメッセージをビューに渡します。

    ```python
    # message_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        # /message_app/welcome/ にアクセスすると "ようこそ！" を表示
        # path('welcome/', views.display_message, {'message': 'ようこそ！'}, name='welcome_message'),
        # /message_app/goodbye/ にアクセスすると "さようなら！" を表示
        # path('goodbye/', views.display_message, {'message': 'さようなら！'}, name='goodbye_message'),
    ]
    ```

5.  **プロジェクトのURL設定**
    `routing_project/urls.py` を開き、`message_app` のURL設定をインクルードしてください。

    ```python
    # routing_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('message_app/', include('message_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/message_app/welcome/` と `http://localhost:8000/message_app/goodbye/` にアクセスしてください。それぞれ異なるメッセージが表示されれば成功です。

## 課題2: URLのプレフィックス

`include()` 関数を使って、共通のURLプレフィックスを持つURL設定を整理しましょう。

### 手順

1.  **新しいアプリケーションの作成**
    `routing_project` プロジェクト内に、`user_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp user_app
    ```

2.  **`settings.py` の設定**
    `routing_project/settings.py` を開き、`INSTALLED_APPS` リストに `'user_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # routing_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'user_app' を追加
        # 'user_app',
    ]
    ```

3.  **ビューの作成**
    `user_app/views.py` を開き、`profile_view` と `settings_view` という名前の関数を作成してください。

    ```python
    # user_app/views.py

    from django.http import HttpResponse

    def profile_view(request):
        return HttpResponse("ユーザープロフィールページ")

    def settings_view(request):
        return HttpResponse("ユーザー設定ページ")
    ```

4.  **アプリケーションのURL設定**
    `user_app` ディレクトリ内に `urls.py` という新しいファイルを作成し、`profile_view` と `settings_view` へのURLを設定してください。

    ```python
    # user_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('profile/', views.profile_view, name='profile'),
        path('settings/', views.settings_view, name='settings'),
    ]
    ```

5.  **プロジェクトのURL設定の修正**
    `routing_project/urls.py` を開き、`user_app` のURL設定を `/users/` というプレフィックスでインクルードしてください。

    ```python
    # routing_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('message_app/', include('message_app.urls')),
        # ここにuser_appのURL設定をプレフィックス付きでインクルード
        # path('users/', include('user_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/users/profile/` と `http://localhost:8000/users/settings/` にアクセスしてください。それぞれのページが表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。