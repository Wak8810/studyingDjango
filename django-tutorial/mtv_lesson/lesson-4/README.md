# Django学習レッスン: MTV (Model-Template-View) - Lesson 4

## 目的

このレッスンでは、Djangoテンプレートの「継承」機能を使って、共通のレイアウトを再利用する方法を学びます。これにより、ウェブサイト全体のデザインの一貫性を保ちつつ、効率的にテンプレートを管理できるようになります。

## 概要

テンプレート継承は、Djangoテンプレートシステムの最も強力な機能の一つです。これにより、共通のHTML構造（ヘッダー、フッター、ナビゲーションなど）を定義した「ベーステンプレート」を作成し、個々のページテンプレートでそのベーステンプレートを拡張（継承）することができます。

*   **`{% extends 'base.html' %}`**: ベーステンプレートを継承することを示します。
*   **`{% block content %}` ... `{% endblock %}`**: ベーステンプレートで定義された「ブロック」を、子テンプレートで上書き（または追加）するためのマークです。

## 環境構築

このレッスンは、`mtv_lesson/lesson-4` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `mtv_lesson/lesson-4` ディレクトリに移動します。
    ```bash
    cd mtv_lesson/lesson-4
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: ベーステンプレートの作成と継承

共通のヘッダーとフッターを持つベーステンプレートを作成し、それを継承した子テンプレートでコンテンツを表示しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `mtv_project` プロジェクト内に、`layout_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp layout_app
    ```

2.  **`settings.py` の設定**
    `mtv_project/settings.py` を開き、`INSTALLED_APPS` リストに `'layout_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # mtv_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'layout_app' を追加
        # 'layout_app',
    ]
    ```

3.  **ベーステンプレートディレクトリの設定**
    `mtv_project/settings.py` を開き、プロジェクトレベルのテンプレートディレクトリを設定してください。これにより、どのアプリケーションからも共通のベーステンプレートを参照できるようになります。

    ```python
    # mtv_project/settings.py

    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [
                BASE_DIR / 'templates', # ここを追加
            ],
            'APP_DIRS': True,
            # ... 既存の設定 ...
        },
    ]
    ```

4.  **ベーステンプレートの作成**
    `mtv_project` プロジェクトのルートディレクトリ（`manage.py` がある場所）に `templates` ディレクトリを作成し、その中に `base.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}My Site{% endblock %}</title>
        <style>
            body { font-family: sans-serif; margin: 0; padding: 0; }
            header { background-color: #333; color: white; padding: 1em; text-align: center; }
            footer { background-color: #eee; color: #333; padding: 1em; text-align: center; position: fixed; bottom: 0; width: 100%; }
            .content { padding: 20px; min-height: calc(100vh - 120px); }
        </style>
    </head>
    <body>
        <header>
            <h1>{% block header_title %}共通ヘッダー{% endblock %}</h1>
        </header>

        <div class="content">
            {% block content %}
                <!-- 子テンプレートのコンテンツがここに入る -->
            {% endblock %}
        </div>

        <footer>
            <p>&copy; 2025 My Django Site</p>
        </footer>
    </body>
    </html>
    ```

5.  **ビューの作成**
    `layout_app/views.py` を開き、`home_view` という名前の関数を作成してください。この関数は、`base.html` を継承するテンプレートをレンダリングします。

    ```python
    # layout_app/views.py

    from django.shortcuts import render

    def home_view(request):
        return render(request, 'layout_app/home.html')
    ```

6.  **子テンプレートの作成**
    `layout_app/templates/layout_app/` ディレクトリを作成し、その中に `home.html` という名前で以下の内容を記述してください。`base.html` を継承し、`content` ブロックを上書きします。

    ```html
    {% comment %} ここでbase.htmlを継承 {% endcomment %}
    {# 例: {% extends 'base.html' %} #}

    {% comment %} ここでtitleブロックを上書き {% endcomment %}
    {# 例: {% block title %}ホーム{% endblock %} #}

    {% comment %} ここでheader_titleブロックを上書き {% endcomment %}
    {# 例: {% block header_title %}ようこそ！{% endblock %} #}

    {% comment %} ここでcontentブロックを上書き {% endcomment %}
    {# 例: {% block content %}
        <h2>これはホームコンテンツです。</h2>
        <p>テンプレート継承の例です。</p>
    {% endblock %} #}
    ```

7.  **URL設定の追加**
    `layout_app/urls.py` を作成し、`home_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # layout_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('', views.home_view, name='home'),
    ]
    ```

    ```python
    # mtv_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('layout_app/', include('layout_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/layout_app/` にアクセスしてください。共通のヘッダーとフッターが表示され、その中に `home.html` で定義したコンテンツが表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
