# Django学習レッスン: セッション - Lesson 4

## 目的

このレッスンでは、セッションのセキュリティ対策と、セッションIDの管理について学びます。これにより、セッションハイジャックなどの攻撃からユーザーを保護し、安全なウェブアプリケーションを構築できるようになります。

## 概要

セッションはユーザーの状態を保持する便利な機能ですが、適切に管理しないとセキュリティ上の脆弱性につながる可能性があります。Djangoは、セッションのセキュリティを強化するための様々な設定を提供しています。

*   **`SESSION_COOKIE_SECURE`**: HTTPS接続でのみセッションクッキーを送信するかどうかを制御します。
*   **`SESSION_COOKIE_HTTPONLY`**: JavaScriptからのセッションクッキーへのアクセスを禁止します。
*   **`SESSION_COOKIE_SAMESITE`**: クロスサイトリクエストフォージェリ (CSRF) 攻撃に対する保護を強化します。
*   **セッションIDの再生成**: ユーザーがログインした際などにセッションIDを再生成することで、セッション固定攻撃を防ぎます。

## 環境構築

このレッスンは、`session_lesson/lesson-4` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `session_lesson/lesson-4` ディレクトリに移動します。
    ```bash
    cd session_lesson/lesson-4
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

## 課題1: セッションクッキーのセキュリティ設定

セッションクッキーをより安全にするための設定を `settings.py` に追加しましょう。

### 手順

1.  **`settings.py` の設定変更**
    `session_project/settings.py` を開き、以下のセッション関連の設定を追加または変更してください。

    ```python
    # session_project/settings.py

    # ... 既存の設定 ...

    # HTTPS接続でのみセッションクッキーを送信 (本番環境ではTrueに設定推奨)
    # SESSION_COOKIE_SECURE = False # 開発環境ではFalseでOK

    # JavaScriptからのセッションクッキーへのアクセスを禁止
    # SESSION_COOKIE_HTTPONLY = True

    # クロスサイトリクエストフォージェリ (CSRF) 攻撃に対する保護を強化
    # SESSION_COOKIE_SAMESITE = 'Lax' # または 'Strict'
    ```

### 確認

これらの設定は、ブラウザの開発者ツール（F12キーなどで開く）の「Application」タブの「Cookies」セクションで確認できます。`Secure` フラグや `HttpOnly` フラグ、`SameSite` 属性が設定されていることを確認してください。

## 課題2: ログイン時のセッションID再生成

ユーザーがログインした際に、セッションIDを再生成することでセッション固定攻撃を防ぐ機能を実装しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `session_project` プロジェクト内に、`auth_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp auth_app
    ```

2.  **`settings.py` の設定**
    `session_project/settings.py` を開き、`INSTALLED_APPS` リストに `'auth_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # session_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'auth_app' を追加
        # 'auth_app',
    ]
    ```

3.  **ビューの作成**
    `auth_app/views.py` を開き、`login_view` と `home_view` という名前の関数を作成してください。`login_view` では、ログイン成功時に `request.session.cycle_key()` を呼び出してセッションIDを再生成します。

    ```python
    # auth_app/views.py

    from django.shortcuts import render, redirect
    from django.http import HttpResponse
    from django.urls import reverse

    # 簡易的なログインフォーム（実際はDjangoの認証フォームを使用）
    class LoginForm(forms.Form):
        username = forms.CharField(max_length=100)
        password = forms.CharField(widget=forms.PasswordInput)

    def login_view(request):
        if request.method == 'POST':
            form = LoginForm(request.POST)
            if form.is_valid():
                # 実際にはここでユーザー認証を行う
                # 認証成功後、セッションIDを再生成
                # request.session.cycle_key()
                return redirect('home')
        else:
            form = LoginForm()
        return render(request, 'auth_app/login.html', {'form': form})

    def home_view(request):
        return HttpResponse("ログイン成功！ホーム画面です。")
    ```

4.  **テンプレートファイルの作成**
    `auth_app/templates/auth_app/` ディレクトリを作成し、その中に `login.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>ログイン</title>
    </head>
    <body>
        <h1>ログイン</h1>
        <form method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">ログイン</button>
        </form>
    </body>
    </html>
    ```

5.  **URL設定の追加**
    `auth_app/urls.py` を作成し、`login_view` と `home_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # auth_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('login/', views.login_view, name='login'),
        path('home/', views.home_view, name='home'),
    ]
    ```

    ```python
    # session_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('auth_app/', include('auth_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/auth_app/login/` にアクセスしてください。開発者ツールでセッションクッキー（`sessionid`）の値をメモします。フォームを送信してログイン成功ページにリダイレクトされた後、再度セッションクッキーの値を確認し、変更されていることを確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
