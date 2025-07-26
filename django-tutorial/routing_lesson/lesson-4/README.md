# Django学習レッスン: ルーティング - Lesson 4

## 目的

このレッスンでは、DjangoのURLルーティングにおける「カスタムパスコンバーター」の作成と使用方法を学びます。これにより、より柔軟で意味のあるURLパターンを定義できるようになります。

## 概要

Djangoの `path()` 関数は、`<int:pk>` や `<str:name>` のように、組み込みのパスコンバーター（`int`, `str`, `slug`, `uuid`, `path`）を提供しています。しかし、これらだけでは表現できない複雑なURLパターンに対応するために、独自のカスタムパスコンバーターを作成することができます。

カスタムパスコンバーターは、以下の2つのメソッドを持つクラスとして定義されます。

*   `regex`: URLパターンにマッチさせるための正規表現を定義します。
*   `to_python(self, value)`: URLからキャプチャした文字列を、Pythonの適切なデータ型に変換します。
*   `to_url(self, value)`: Pythonの値を、URLに含めるための文字列に変換します（URLの逆引き時に使用）。

## 環境構築

このレッスンは、`routing_lesson/lesson-4` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `routing_lesson/lesson-4` ディレクトリに移動します。
    ```bash
    cd routing_lesson/lesson-4
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: カスタムパスコンバーターの作成と使用

ユーザーが `/product/P12345/` のようにアクセスした際に、`P` で始まる5桁の数字を商品IDとしてビューに渡すカスタムパスコンバーターを作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `routing_project` プロジェクト内に、`shop_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp shop_app
    ```

2.  **`settings.py` の設定**
    `routing_project/settings.py` を開き、`INSTALLED_APPS` リストに `'shop_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # routing_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'shop_app' を追加
        # 'shop_app',
    ]
    ```

3.  **カスタムパスコンバーターの作成**
    `shop_app` ディレクトリ内に `converters.py` という新しいファイルを作成し、以下の内容を記述してください。

    ```python
    # shop_app/converters.py

    class ProductIDConverter:
        regex = 'P[0-9]{5}' # 'P'で始まり、5桁の数字

        def to_python(self, value):
            # URLからキャプチャした文字列をPythonのオブジェクトに変換
            return value # この例では文字列のまま

        def to_url(self, value):
            # PythonのオブジェクトをURLに含める文字列に変換
            return str(value)
    ```

4.  **カスタムパスコンバーターの登録**
    `routing_project/urls.py` を開き、作成したカスタムパスコンバーターを登録してください。

    ```python
    # routing_project/urls.py

    from django.contrib import admin
    from django.urls import path, include, register_converter # register_converterをインポート
    from shop_app import converters # カスタムコンバーターをインポート

    # カスタムコンバーターを登録
    # register_converter(converters.ProductIDConverter, 'product_id')

    urlpatterns = [
        path('admin/', admin.site.urls),
        # ...
    ]
    ```

5.  **ビューの作成**
    `shop_app/views.py` を開き、`product_detail` という名前の関数を作成してください。この関数は、カスタムパスコンバーターによって変換された商品IDを受け取ります。

    ```python
    # shop_app/views.py

    from django.http import HttpResponse

    def product_detail(request, product_id):
        return HttpResponse(f"商品ID: {product_id} の詳細ページ")
    ```

6.  **アプリケーションのURL設定**
    `shop_app` ディレクトリ内に `urls.py` という新しいファイルを作成し、`product_detail` ビューへのURLを設定してください。カスタムパスコンバーターを使用します。

    ```python
    # shop_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        # カスタムパスコンバーターを使用
        # path('product/<product_id:product_id>/', views.product_detail, name='product_detail'),
    ]
    ```

7.  **プロジェクトのURL設定**
    `routing_project/urls.py` を開き、`shop_app` のURL設定をインクルードしてください。

    ```python
    # routing_project/urls.py

    from django.contrib import admin
    from django.urls import path, include, register_converter
    from shop_app import converters

    register_converter(converters.ProductIDConverter, 'product_id')

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('shop_app/', include('shop_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/shop_app/product/P12345/` にアクセスしてください。「商品ID: P12345 の詳細ページ」と表示されれば成功です。`http://localhost:8000/shop_app/product/P123/` のように、正規表現にマッチしないURLにアクセスすると404エラーになることも確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
