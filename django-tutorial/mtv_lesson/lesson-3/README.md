# Django学習レッスン: MTV (Model-Template-View) - Lesson 3

## 目的

このレッスンでは、Djangoテンプレートのより高度な機能である「テンプレートフィルター」を使ってデータを整形する方法と、ウェブサイトのデザインに不可欠な「静的ファイル」（CSS、JavaScript、画像など）をDjangoで扱う方法を学びます。

## 概要

*   **テンプレートフィルター**: テンプレート内で変数の値を表示する際に、その値を変換したり整形したりするための機能です。例えば、日付のフォーマット変更、文字列の大文字/小文字変換、リストの要素の結合などに使われます。
*   **静的ファイル**: CSS、JavaScript、画像ファイルなど、サーバー側で動的に生成されないファイルのことです。Djangoでは、これらの静的ファイルを効率的に提供するための仕組みが用意されています。

## 環境構築

このレッスンは、`mtv_lesson/lesson-3` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `mtv_lesson/lesson-3` ディレクトリに移動します。
    ```bash
    cd mtv_lesson/lesson-3
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: テンプレートフィルターの使用

ビューから渡された日付や文字列を、テンプレートフィルターを使って整形して表示しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `mtv_project` プロジェクト内に、`filter_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp filter_app
    ```

2.  **`settings.py` の設定**
    `mtv_project/settings.py` を開き、`INSTALLED_APPS` リストに `'filter_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # mtv_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'filter_app' を追加
        # 'filter_app',
    ]
    ```

3.  **ビューの作成**
    `filter_app/views.py` を開き、`filter_example` という名前の関数を作成してください。この関数内で、日付オブジェクトと文字列を定義し、それをテンプレートに渡してください。

    ```python
    # filter_app/views.py

    from django.shortcuts import render
    from datetime import datetime

    def filter_example(request):
        current_date = datetime.now()
        message = "hello django filters"
        context = {
            # ここにcurrent_dateとmessageを渡す
            # 'current_date': current_date,
            # 'message': message,
        }
        # ここでテンプレートをレンダリング
        # return render(request, 'filter_app/filter_example.html', context)
        pass
    ```

4.  **テンプレートファイルの作成**
    `filter_app/templates/filter_app/` ディレクトリを作成し、その中に `filter_example.html` という名前で以下の内容を記述してください。`current_date` を `"Y年m月d日 H時i分s秒"` の形式で表示し、`message` をすべて大文字と小文字で表示するようにテンプレートフィルターを適用してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>テンプレートフィルター</title>
    </head>
    <body>
        <h1>テンプレートフィルターの例</h1>
        <p>現在の日付: {% comment %} ここにdateフィルターを適用 {% endcomment %}</p>
        <p>メッセージ（大文字）: {% comment %} ここにupperフィルターを適用 {% endcomment %}</p>
        <p>メッセージ（小文字）: {% comment %} ここにlowerフィルターを適用 {% endcomment %}</p>
    </body>
    </html>
    ```

5.  **アプリケーションのURL設定**
    `filter_app` ディレクトリ内に `urls.py` という新しいファイルを作成し、`filter_example` ビューへのURLを設定してください。

    ```python
    # filter_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('filters/', views.filter_example, name='filter_example'),
    ]
    ```

6.  **プロジェクトのURL設定**
    `mtv_project/urls.py` を開き、`filter_app` のURL設定をインクルードしてください。

    ```python
    # mtv_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        # ここにfilter_appのURL設定をインクルード
        # path('filter_app/', include('filter_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/filter_app/filters/` にアクセスしてください。日付が指定された形式で、メッセージが大文字と小文字で表示されれば成功です。

## 課題2: 静的ファイルの提供

ウェブページにCSSファイルを適用し、スタイルを適用しましょう。

### 手順

1.  **静的ファイルディレクトリの設定**
    `mtv_project/settings.py` を開き、静的ファイルがどこにあるかをDjangoに教えるために `STATIC_URL` と `STATICFILES_DIRS` を設定してください。

    ```python
    # mtv_project/settings.py

    STATIC_URL = '/static/'

    # プロジェクトレベルの静的ファイルディレクトリ
    STATICFILES_DIRS = [
        BASE_DIR / 'static',
    ]
    ```

2.  **静的ファイルディレクトリの作成**
    `mtv_project` プロジェクトのルートディレクトリ（`manage.py` がある場所）に `static` ディレクトリを作成し、その中に `css` ディレクトリを作成してください。
    ```bash
    mkdir -p static/css
    ```

3.  **CSSファイルの作成**
    `static/css/style.css` という名前で新しいCSSファイルを作成し、以下の内容を記述してください。

    ```css
    /* static/css/style.css */
    body {
        font-family: sans-serif;
        background-color: #f0f0f0;
        color: #333;
    }
    h1 {
        color: #0056b3;
    }
    ```

4.  **テンプレートファイルの修正**
    `filter_app/templates/filter_app/filter_example.html` を開き、作成したCSSファイルを読み込むように `<head>` セクションに `<link>` タグを追加してください。`{% load static %}` テンプレートタグをファイルの先頭に追加するのを忘れないでください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>テンプレートフィルター</title>
        {% comment %} ここに静的ファイルをロードするタグとCSSリンクを記述 {% endcomment %}
        {# 例: {% load static %}
        <link rel="stylesheet" href="{% static 'css/style.css' %}">
        #}
    </head>
    <body>
        <h1>テンプレートフィルターの例</h1>
        <p>現在の日付: {{ current_date|date:"Y年m月d日 H時i分s秒" }}</p>
        <p>メッセージ（大文字）: {{ message|upper }}</p>
        <p>メッセージ（小文字）: {{ message|lower }}</p>
    </body>
    </html>
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/filter_app/filters/` に再度アクセスしてください。ページの背景色や文字の色、フォントがCSSによって変更されていれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
