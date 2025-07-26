# Django学習レッスン: セッション - Lesson 3

## 目的

このレッスンでは、セッションのクリアと、セッションのバックエンド（保存場所）の変更について学びます。これにより、セッション管理の柔軟性を高め、アプリケーションの要件に合わせて最適化できるようになります。

## 概要

*   **セッションのクリア**: ユーザーがログアウトした際など、セッションに保存されているすべてのデータを削除したい場合があります。`request.session.clear()` や `request.session.flush()` を使用します。
*   **セッションバックエンド**: Djangoは、セッションデータを保存するための複数のバックエンドを提供しています。デフォルトはデータベースですが、ファイルシステム、キャッシュ、クッキーなどがあります。バックエンドを変更することで、パフォーマンスやスケーラビリティを向上させることができます。

## 環境構築

このレッスンは、`session_lesson/lesson-3` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `session_lesson/lesson-3` ディレクトリに移動します。
    ```bash
    cd session_lesson/lesson-3
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

## 課題1: セッションのクリア

ユーザーが「セッションをクリア」ボタンをクリックすると、セッションに保存されているすべてのデータが削除される機能を作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `session_project` プロジェクト内に、`session_control_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp session_control_app
    ```

2.  **`settings.py` の設定**
    `session_project/settings.py` を開き、`INSTALLED_APPS` リストに `'session_control_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # session_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'session_control_app' を追加
        # 'session_control_app',
    ]
    ```

3.  **ビューの作成**
    `session_control_app/views.py` を開き、`set_session_view` と `clear_session_view` という名前の関数を作成してください。`set_session_view` はセッションにデータを保存し、`clear_session_view` はセッションをクリアします。

    ```python
    # session_control_app/views.py

    from django.shortcuts import render, redirect
    from django.urls import reverse

    def set_session_view(request):
        # セッションにデータを保存
        # request.session['my_data'] = 'これはセッションに保存されたデータです。'
        # request.session['counter'] = request.session.get('counter', 0) + 1
        
        # context = {
        #     'my_data': request.session.get('my_data'),
        #     'counter': request.session.get('counter'),
        # }
        # return render(request, 'session_control_app/session_status.html', context)
        pass

    def clear_session_view(request):
        # セッションデータをすべてクリア
        # request.session.clear()
        # return redirect('set_session') # セッション設定ページにリダイレクト
        pass
    ```

4.  **テンプレートファイルの作成**
    `session_control_app/templates/session_control_app/` ディレクトリを作成し、その中に `session_status.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>セッションステータス</title>
    </head>
    <body>
        <h1>セッションステータス</h1>
        <p>My Data: {{ my_data }}</p>
        <p>Counter: {{ counter }}</p>
        <p>
            <a href="{% url 'clear_session' %}">セッションをクリア</a>
        </p>
    </body>
    </html>
    ```

5.  **URL設定の追加**
    `session_control_app/urls.py` を作成し、`set_session_view` と `clear_session_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # session_control_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('set/', views.set_session_view, name='set_session'),
        path('clear/', views.clear_session_view, name='clear_session'),
    ]
    ```

    ```python
    # session_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('session_control_app/', include('session_control_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/session_control_app/set/` にアクセスしてください。セッションデータが設定され、カウンターが増えることを確認します。その後、「セッションをクリア」リンクをクリックし、データがクリアされることを確認してください。

## 課題2: セッションバックエンドの変更

セッションデータをデータベースではなく、ファイルシステムに保存するように設定を変更しましょう。

### 手順

1.  **`settings.py` の設定変更**
    `session_project/settings.py` を開き、`SESSION_ENGINE` を `'django.contrib.sessions.backends.file'` に変更してください。また、セッションファイルを保存するディレクトリを指定するために `SESSION_FILE_PATH` を設定してください。

    ```python
    # session_project/settings.py

    # ... 既存の設定 ...

    # セッションバックエンドをファイルシステムに変更
    # SESSION_ENGINE = 'django.contrib.sessions.backends.file'
    # セッションファイルを保存するディレクトリ (プロジェクトルートからの相対パス)
    # SESSION_FILE_PATH = BASE_DIR / 'sessions'
    ```

2.  **セッションファイル保存ディレクトリの作成**
    `session_project` プロジェクトのルートディレクトリ（`manage.py` がある場所）に `sessions` ディレクトリを作成してください。
    ```bash
    mkdir sessions
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/session_control_app/set/` にアクセスし、セッションデータを設定します。その後、`sessions` ディレクトリ内にセッションファイルが作成されていることを確認してください。Dockerコンテナを停止・起動してもセッションが保持されることを確認します。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
