# Django学習レッスン: セッション - Lesson 1

## 目的

このレッスンでは、Djangoにおけるセッションの概念と、ユーザーの状態を複数のリクエスト間で保持する方法を学びます。訪問回数のカウントや、カスタムデータの保存・取得、セッションのクリアと有効期限の設定について実践します。

## セッションの概要

セッションとは、ウェブサイトを訪れるユーザーの状態（情報）を、複数のリクエスト（ページの閲覧など）をまたいで保持するための仕組みです。HTTPは本来ステートレス（状態を持たない）なプロトコルなので、セッションを使うことで、ユーザーがログインしているか、カートに何が入っているか、といった情報をサーバー側で記憶しておくことができます。

Djangoは、セッション管理のための強力なフレームワークを標準で提供しています。デフォルトではデータベースにセッション情報を保存しますが、ファイルシステムやキャッシュなど、他のバックエンドも利用可能です。

## 環境構築

このレッスンは、`session_lesson/lesson-1` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `session_lesson/lesson-1` ディレクトリに移動します。
    ```bash
    cd session_lesson/lesson-1
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

## 課題1: 訪問回数カウンター

ユーザーがページを訪れるたびに訪問回数をカウントし、表示するシンプルなカウンターを作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `session_project` プロジェクト内に、`my_session_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp my_session_app
    ```

2.  **`settings.py` の設定**
    `session_project/settings.py` を開き、`INSTALLED_APPS` リストに `'my_session_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # session_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'my_session_app' を追加
        # 'my_session_app',
    ]
    ```

3.  **ビューの作成**
    `my_session_app/views.py` を開き、`session_counter_view` という名前の関数を作成してください。この関数は、セッションから `visits` カウンターを取得し、インクリメントして、テンプレートに渡します。

    ```python
    # my_session_app/views.py

    from django.shortcuts import render

    def session_counter_view(request):
        # セッションから'visits'キーの値を取得。なければ0
        # num_visits = request.session.get('visits', 0)
        
        # 訪問回数を1増やす
        # request.session['visits'] = num_visits + 1
        
        # テンプレートに渡すコンテキスト
        # context = {
        #     'visits': request.session['visits'],
        # }
        
        # テンプレートをレンダリング
        # return render(request, 'my_session_app/session_counter.html', context)
        pass
    ```

4.  **テンプレートファイルの作成**
    `my_session_app/templates/my_session_app/` ディレクトリを作成し、その中に `session_counter.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>セッションカウンター</title>
    </head>
    <body>
        <h1>セッションカウンター</h1>
        <p>このページへの訪問回数: {{ visits }}</p>
        <p>ブラウザを閉じたり、別のブラウザでアクセスするとカウントがリセットされることを確認してください。</p>
    </body>
    </html>
    ```

5.  **URL設定の追加**
    `my_session_app/urls.py` を作成し、`session_counter_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # my_session_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('counter/', views.session_counter_view, name='session_counter'),
    ]
    ```

    ```python
    # session_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('my_session_app/', include('my_session_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/my_session_app/counter/` にアクセスしてください。ページをリロードするたびに数字が増えていれば成功です。

## 課題2: カスタムデータの保存と取得

ユーザーのお気に入りの色をセッションに保存し、表示する機能を追加しましょう。

### 手順

1.  **フォームの定義**
    `my_session_app` ディレクトリ内に `forms.py` という新しいファイルを作成し、以下の内容を記述してください。

    ```python
    # my_session_app/forms.py

    from django import forms

    class FavoriteColorForm(forms.Form):
        color = forms.CharField(label='お気に入りの色', max_length=50)
    ```

2.  **ビューの修正**
    `my_session_app/views.py` を開き、`favorite_color_view` という名前の関数を作成してください。この関数は、フォームから色を受け取りセッションに保存し、セッションから色を取得してテンプレートに渡します。

    ```python
    # my_session_app/views.py

    from django.shortcuts import render
    from django.http import HttpResponseRedirect
    from django.urls import reverse
    from .forms import FavoriteColorForm # FavoriteColorFormをインポート

    # ... 既存のsession_counter_view ...

    def favorite_color_view(request):
        if request.method == 'POST':
            form = FavoriteColorForm(request.POST)
            if form.is_valid():
                # お気に入りの色をセッションに保存
                # request.session['favorite_color'] = form.cleaned_data['color']
                # request.session.modified = True # 変更を保存
                return HttpResponseRedirect(reverse('favorite_color'))
        else:
            form = FavoriteColorForm()
        
        # セッションからお気に入りの色を取得
        # favorite_color = request.session.get('favorite_color', '未設定')
        
        # context = {
        #     'form': form,
        #     'favorite_color': favorite_color,
        # }
        # return render(request, 'my_session_app/favorite_color.html', context)
        pass
    ```

3.  **テンプレートファイルの作成**
    `my_session_app/templates/my_session_app/` ディレクトリ内に `favorite_color.html` という名前で以下の内容を記述してください。

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

4.  **URL設定の追加**
    `my_session_app/urls.py` に、`favorite_color_view` へのURLを追加してください。

    ```python
    # my_session_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('counter/', views.session_counter_view, name='session_counter'),
        # ここにfavorite_color_viewへのURLを追加
        # path('color/', views.favorite_color_view, name='favorite_color'),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/my_session_app/color/` にアクセスしてください。色を設定し、ページをリロードしても色が保持されることを確認してください。

## 課題3: セッションのクリアと有効期限

セッションデータをクリアする機能と、セッションの有効期限を設定する方法を学びましょう。

### 手順

1.  **ビューの修正**
    `my_session_app/views.py` を開き、`clear_session_view` という名前の関数を作成してください。この関数は、`request.session.clear()` を使ってセッションデータをクリアします。

    ```python
    # my_session_app/views.py

    from django.shortcuts import render
    # ... 既存のビュー ...

    def clear_session_view(request):
        # セッションデータをすべて削除
        # request.session.clear()
        
        # テンプレートに渡すメッセージ
        # context = {
        #     'message': 'セッションデータがクリアされました。',
        # }
        # return render(request, 'my_session_app/session_cleared.html', context)
        pass
    ```

2.  **テンプレートファイルの作成**
    `my_session_app/templates/my_session_app/` ディレクトリ内に `session_cleared.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>セッションクリア</title>
    </head>
    <body>
        <h1>セッションクリア</h1>
        <p>{{ message }}</p>
        <p><a href="{% url 'session_counter' %}">セッションカウンターに戻る</a></p>
        <p><a href="{% url 'favorite_color' %}">お気に入りの色設定に戻る</a></p>
    </body>
    </html>
    ```

3.  **URL設定の追加**
    `my_session_app/urls.py` に、`clear_session_view` へのURLを追加してください。

    ```python
    # my_session_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('counter/', views.session_counter_view, name='session_counter'),
        path('color/', views.favorite_color_view, name='favorite_color'),
        # ここにclear_session_viewへのURLを追加
        # path('clear/', views.clear_session_view, name='clear_session'),
    ]
    ```

4.  **セッションの有効期限の設定**
    `session_project/settings.py` を開き、セッションの有効期限を設定してください。例えば、`SESSION_COOKIE_AGE = 60` と設定すると、セッションは60秒で期限切れになります。

    ```python
    # session_project/settings.py

    # ... 既存の設定 ...

    # セッションの有効期限を秒単位で設定 (例: 60秒)
    # SESSION_COOKIE_AGE = 60

    # ブラウザを閉じるとセッションが期限切れになるか (True/False)
    # SESSION_EXPIRE_AT_BROWSER_CLOSE = False
    ```

### 確認

すべての変更を保存したら、

*   `http://localhost:8000/my_session_app/counter/` や `http://localhost:8000/my_session_app/color/` でセッションデータを設定します。
*   `http://localhost:8000/my_session_app/clear/` にアクセスし、セッションがクリアされることを確認します。
*   `SESSION_COOKIE_AGE` を設定後、セッションデータを設定し、設定した時間（例: 60秒）以上待ってから再度アクセスし、セッションが期限切れになっていることを確認します。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
*   `docker compose exec web python manage.py shell` でシェルに入り、`request.session` の内容を確認してみてください。
