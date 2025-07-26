# Django学習レッスン: MTV (Model-Template-View) - Lesson 1

## 目的

このレッスンでは、Djangoの基本的なアーキテクチャであるMTV (Model-Template-View) について学びます。特に、ViewとTemplateの連携に焦点を当て、シンプルなウェブページを表示する手順を理解します。

## MTVアーキテクチャの概要

Djangoは、伝統的なMVC (Model-View-Controller) パターンを独自に解釈したMTVパターンを採用しています。

*   **Model (モデル)**: データの定義を担当します。データベースのテーブル構成や、データ同士の関係性をPythonのクラスとして表現します。アプリケーションの「データ」そのものを扱います。

*   **Template (テンプレート)**: ユーザーに見せる部分、つまりプレゼンテーションを担当します。HTMLを主体とし、そこにどのようにデータを表示するかを定義します。

*   **View (ビュー)**: ModelとTemplateの橋渡し役です。ユーザーからのリクエストを受け取り、必要に応じてModelからデータを取得し、そのデータをTemplateに渡して、最終的なHTMLレスポンスを生成します。MVCの「Controller」に相当するロジック部分です。

## 環境構築

このレッスンは、`mtv_lesson/lesson-1` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `mtv_lesson/lesson-1` ディレクトリに移動します。
    ```bash
    cd mtv_lesson/lesson-1
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: シンプルなビューとテンプレートの作成

ユーザーが `/hello/` にアクセスした際に、「Hello, MTV Lesson!」というメッセージを表示するページを作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `mtv_project` プロジェクト内に、`hello_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp hello_app
    ```

2.  **`settings.py` の設定**
    `mtv_project/settings.py` を開き、`INSTALLED_APPS` リストに `'hello_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # mtv_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'hello_app' を追加
        # 'hello_app',
    ]
    ```

3.  **ビューの作成**
    `hello_app/views.py` を開き、`hello_world` という名前の関数を作成してください。この関数は、`HttpResponse` を使って「Hello, MTV Lesson!」という文字列を直接返します。

    ```python
    # hello_app/views.py

    from django.http import HttpResponse

    def hello_world(request):
        # ここにコードを記述
        # return HttpResponse("Hello, MTV Lesson!")
        pass
    ```

4.  **アプリケーションのURL設定**
    `hello_app` ディレクトリ内に `urls.py` という新しいファイルを作成し、以下の内容を記述してください。

    ```python
    # hello_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        # ここにURLパターンを記述
        # path('hello/', views.hello_world, name='hello_world'),
    ]
    ```

5.  **プロジェクトのURL設定**
    `mtv_project/urls.py` を開き、`hello_app` のURL設定をインクルードしてください。

    ```python
    # mtv_project/urls.py

    from django.contrib import admin
    from django.urls import path, include # includeをインポート

    urlpatterns = [
        path('admin/', admin.site.urls),
        # ここにhello_appのURL設定をインクルード
        # path('hello_app/', include('hello_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/hello_app/hello/` にアクセスしてください。「Hello, MTV Lesson!」と表示されれば成功です。

## 課題2: テンプレートの導入

次に、直接文字列を返すのではなく、HTMLテンプレートを使ってメッセージを表示するように変更しましょう。

### 手順

1.  **`templates` ディレクトリの作成**
    `hello_app` ディレクトリ内に、`templates/hello_app/` というディレクトリを作成してください。
    ```bash
    mkdir -p hello_app/templates/hello_app
    ```

2.  **テンプレートファイルの作成**
    `hello_app/templates/hello_app/hello.html` という名前で新しいHTMLファイルを作成し、以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>MTV Lesson</title>
    </head>
    <body>
        <h1>{{ message }}</h1>
        <p>これはテンプレートから表示されています。</p>
    </body>
    </html>
    ```

3.  **ビューの修正**
    `hello_app/views.py` を開き、`hello_world` 関数を修正して、`render` 関数を使って `hello.html` テンプレートをレンダリングするように変更してください。`message` というコンテキスト変数に「Hello, MTV Lesson!」という文字列を渡してください。

    ```python
    # hello_app/views.py

    from django.shortcuts import render # renderをインポート
    # from django.http import HttpResponse # 不要になるのでコメントアウトまたは削除

    def hello_world(request):
        context = {
            # ここにコンテキスト変数を定義
            # 'message': 'Hello, MTV Lesson!',
        }
        # ここでテンプレートをレンダリング
        # return render(request, 'hello_app/hello.html', context)
        pass
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/hello_app/hello/` に再度アクセスしてください。今度は、HTMLでレンダリングされた「Hello, MTV Lesson!」という見出しが表示されれば成功です。

## 課題3: モデルの作成とデータの表示

最後に、Modelを作成し、データベースからデータを取得してテンプレートに表示する流れを学びましょう。

### 手順

1.  **モデルの定義**
    `hello_app/models.py` を開き、`Greeting` という名前のモデルを作成してください。このモデルには、`message` という `CharField` を持たせてください。

    ```python
    # hello_app/models.py

    from django.db import models

    class Greeting(models.Model):
        # ここにmessageフィールドを定義
        # message = models.CharField(max_length=200)

        def __str__(self):
            return self.message
    ```

2.  **マイグレーションの作成と適用**
    モデルを変更したら、データベースにその変更を反映させる必要があります。

    ```bash
    # マイグレーションファイルを作成
    docker compose exec web python manage.py makemigrations hello_app
    # マイグレーションをデータベースに適用
    docker compose exec web python manage.py migrate
    ```

3.  **初期データの投入**
    Djangoシェルを使って、`Greeting` モデルにデータを投入してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from hello_app.models import Greeting
    Greeting.objects.create(message="Hello from Model!")
    exit()
    ```

4.  **ビューの修正**
    `hello_app/views.py` を開き、`hello_world` 関数を修正して、`Greeting` モデルからデータを取得し、そのデータをテンプレートに渡すように変更してください。

    ```python
    # hello_app/views.py

    from django.shortcuts import render
    from .models import Greeting # Greetingモデルをインポート

    def hello_world(request):
        # データベースから最初のGreetingオブジェクトを取得
        # greeting = Greeting.objects.first()
        
        context = {
            # 取得したメッセージをテンプレートに渡す
            # 'message': greeting.message if greeting else "No message from Model.",
        }
        return render(request, 'hello_app/hello.html', context)
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/hello_app/hello/` に再度アクセスしてください。「Hello from Model!」というメッセージが表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
*   `docker compose exec web python manage.py shell` でシェルに入り、モデルのインポートやデータの確認を試してみてください。
