# Django学習レッスン: セッション - Lesson 2

## 目的

このレッスンでは、セッションにカスタムデータを保存し、それを取得して表示する方法を学びます。また、セッションの有効期限を制御する方法についても理解を深めます。

## 概要

セッションは、単純なカウンターだけでなく、ユーザー名、設定、カートの内容など、より複雑なデータを保存するためにも使われます。セッションデータはPythonの辞書（dictionary）のように扱うことができます。

*   **カスタムデータの保存**: `request.session['key'] = value` の形式でデータを保存します。
*   **カスタムデータの取得**: `request.session.get('key', default_value)` の形式でデータを取得します。
*   **セッションの有効期限**: `settings.py` の `SESSION_COOKIE_AGE` や `SESSION_EXPIRE_AT_BROWSER_CLOSE` で制御します。

## 環境構築

このレッスンは、`session_lesson/lesson-2` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `session_lesson/lesson-2` ディレクトリに移動します。
    ```bash
    cd session_lesson/lesson-2
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

## 課題1: お気に入りの色をセッションに保存・表示

ユーザーがお気に入りの色を入力し、それをセッションに保存して表示する機能を作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `session_project` プロジェクト内に、`user_pref_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp user_pref_app
    ```

2.  **`settings.py` の設定**
    `session_project/settings.py` を開き、`INSTALLED_APPS` リストに `'user_pref_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # session_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'user_pref_app' を追加
        # 'user_pref_app',
    ]
    ```

3.  **フォームの定義**
    `user_pref_app` ディレクトリ内に `forms.py` という新しいファイルを作成し、以下の内容を記述してください。

    ```python
    # user_pref_app/forms.py

    from django import forms

    class FavoriteColorForm(forms.Form):
        color = forms.CharField(label='お気に入りの色', max_length=50)
    ```

4.  **ビューの作成**
    `user_pref_app/views.py` を開き、`favorite_color_view` という名前の関数を作成してください。この関数は、フォームから色を受け取りセッションに保存し、セッションから色を取得してテンプレートに渡します。

    ```python
    # user_pref_app/views.py

    from django.shortcuts import render
    from django.http import HttpResponseRedirect
    from django.urls import reverse
    from .forms import FavoriteColorForm

    def favorite_color_view(request):
        if request.method == 'POST':
            form = FavoriteColorForm(request.POST)
            if form.is_valid():
                # バリデーションが成功したら、お気に入りの色をセッションに保存
                # request.session['favorite_color'] = form.cleaned_data['color']
                # request.session.modified = True # 変更を保存（セッションバックエンドによっては不要な場合もありますが、明示的に行うのが安全です）
                # return HttpResponseRedirect(reverse('favorite_color')) # 同じページにリダイレクトして、更新された色を表示
                pass
        else:
            form = FavoriteColorForm()
        
        # セッションからお気に入りの色を取得
        # favorite_color = request.session.get('favorite_color', '未設定')
        
        # context = {
        #     'form': form,
        #     'favorite_color': favorite_color,
        # }
        # return render(request, 'user_pref_app/favorite_color.html', context)
        pass
    ```

5.  **テンプレートファイルの作成**
    `user_pref_app/templates/user_pref_app/` ディレクトリを作成し、その中に `favorite_color.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>お気に入りの色</title>
    </head>
    <body>
        <h1>お気に入りの色を設定</h1>
        <p>現在のお気に入りの色: {{ favorite_color }}</p>
        
        <form action="" method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <input type="submit" value="設定">
        </form>
    </body>
    </html>
    ```

6.  **URL設定の追加**
    `user_pref_app/urls.py` を作成し、`favorite_color_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # user_pref_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('color/', views.favorite_color_view, name='favorite_color'),
    ]
    ```

    ```python
    # session_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('user_pref_app/', include('user_pref_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/user_pref_app/color/` にアクセスしてください。色を設定し、ページをリロードしても色が保持されることを確認してください。

## 課題2: セッションの有効期限の設定

セッションの有効期限を短く設定し、一定時間経過後にセッションが自動的にクリアされることを確認しましょう。

### 手順

1.  **`settings.py` の設定**
    `session_project/settings.py` を開き、セッションの有効期限を短く設定してください。例えば、`SESSION_COOKIE_AGE` を `60` 秒（1分）に設定します。

    ```python
    # session_project/settings.py

    # ... 既存の設定 ...

    # セッションの有効期限を秒単位で設定 (例: 60秒)
    # SESSION_COOKIE_AGE = 60

    # ブラウザを閉じてもセッションが期限切れにならないように設定（デフォルトはFalse）
    # SESSION_EXPIRE_AT_BROWSER_CLOSE = False
    ```

### 確認

すべての変更を保存したら、

*   `http://localhost:8000/user_pref_app/color/` でお気に入りの色を設定します。
*   **1分以上待ってから**、再度 `http://localhost:8000/user_pref_app/color/` にアクセスしてみてください。設定した色が「未設定」に戻っていることを確認します。これは `SESSION_COOKIE_AGE` の設定により、セッションが期限切れになったためです。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
