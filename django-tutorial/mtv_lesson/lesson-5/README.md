# Django学習レッスン: MTV (Model-Template-View) - Lesson 5

## 目的

このレッスンでは、Djangoのフォーム機能を使って、ユーザーからの入力を受け取り、それを処理してデータベースに保存する一連の流れを学びます。これにより、ユーザーとのインタラクションを伴うウェブアプリケーションの基本的な構築方法を理解します。

## 概要

Djangoのフォームは、HTMLフォームの生成、入力データのバリデーション、そしてクリーンなデータへの変換を効率的に行うための強力なツールです。フォームとビュー、モデルを連携させることで、ユーザーからの入力を安全かつ簡単に処理できます。

*   **`forms.Form`**: シンプルなHTMLフォームを扱うための基底クラスです。
*   **`forms.ModelForm`**: モデルと直接連携し、モデルのフィールドに基づいてフォームを自動生成するためのクラスです。データベースへの保存も簡単に行えます。

## 環境構築

このレッスンは、`mtv_lesson/lesson-5` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `mtv_lesson/lesson-5` ディレクトリに移動します。
    ```bash
    cd mtv_lesson/lesson-5
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: `ModelForm` を使ったデータの保存

ユーザーが書籍情報を入力し、それをデータベースに保存するフォームを作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `mtv_project` プロジェクト内に、`book_entry_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp book_entry_app
    ```

2.  **`settings.py` の設定**
    `mtv_project/settings.py` を開き、`INSTALLED_APPS` リストに `'book_entry_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # mtv_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'book_entry_app' を追加
        # 'book_entry_app',
    ]
    ```

3.  **モデルの定義**
    `book_entry_app/models.py` を開き、`Book` という名前のモデルを作成してください。このモデルには、`title` (CharField), `author` (CharField), `published_date` (DateField) のフィールドを持たせてください。

    ```python
    # book_entry_app/models.py

    from django.db import models

    class Book(models.Model):
        title = models.CharField(max_length=200)
        author = models.CharField(max_length=100)
        published_date = models.DateField()

        def __str__(self):
            return self.title
    ```

4.  **マイグレーションの作成と適用**
    モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成し、データベースに適用してください。

    ```bash
    docker compose exec web python manage.py makemigrations book_entry_app
    docker compose exec web python manage.py migrate
    ```

5.  **フォームの定義 (`ModelForm`)**
    `book_entry_app` ディレクトリ内に `forms.py` という新しいファイルを作成し、`Book` モデルに対応する `BookForm` を定義してください。

    ```python
    # book_entry_app/forms.py

    from django import forms
    from .models import Book

    class BookForm(forms.ModelForm):
        class Meta:
            model = Book
            fields = ['title', 'author', 'published_date']
    ```

6.  **ビューの作成**
    `book_entry_app/views.py` を開き、`book_create_view` という名前の関数を作成してください。この関数は、フォームの表示と処理、そしてデータベースへの保存を行います。

    ```python
    # book_entry_app/views.py

    from django.shortcuts import render, redirect
    from .forms import BookForm
    from .models import Book # 登録後のリスト表示のためにインポート

    def book_create_view(request):
        if request.method == 'POST':
            form = BookForm(request.POST)
            if form.is_valid():
                form.save() # フォームのデータをデータベースに保存
                return redirect('book_list') # 登録後、書籍リストページにリダイレクト
        else:
            form = BookForm()
        
        books = Book.objects.all() # 既存の書籍リストを取得
        return render(request, 'book_entry_app/book_form.html', {'form': form, 'books': books})
    ```

7.  **テンプレートファイルの作成**
    `book_entry_app/templates/book_entry_app/` ディレクトリを作成し、その中に `book_form.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>書籍登録</title>
    </head>
    <body>
        <h1>書籍登録</h1>
        <form method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">登録</button>
        </form>

        <h2>登録済み書籍</h2>
        <ul>
            {% for book in books %}
                <li>{{ book.title }} by {{ book.author }} ({{ book.published_date }})</li>
            {% empty %}
                <li>まだ書籍が登録されていません。</li>
            {% endfor %}
        </ul>
    </body>
    </html>
    ```

8.  **URL設定の追加**
    `book_entry_app/urls.py` を作成し、`book_create_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # book_entry_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('create/', views.book_create_view, name='book_create'),
        path('list/', views.book_create_view, name='book_list'), # 登録後のリダイレクト先として同じビューを使用
    ]
    ```

    ```python
    # mtv_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('books/', include('book_entry_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/books/create/` にアクセスしてください。フォームが表示され、書籍情報を入力して登録できることを確認してください。登録後、リストに表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
*   `docker compose exec web python manage.py shell` でシェルに入り、モデルのインポートやデータの確認を試してみてください。
