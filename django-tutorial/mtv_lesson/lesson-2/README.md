# Django学習レッスン: MTV (Model-Template-View) - Lesson 2

## 目的

このレッスンでは、ビューからテンプレートへデータを渡し、テンプレートタグ（`for`ループや`if`文）を使って動的にコンテンツを表示する方法を学びます。

## 概要

Djangoのテンプレートは、単なる静的なHTMLファイルではありません。ビューから渡されたデータ（コンテキスト変数）を受け取り、`{{ variable }}` のようなテンプレート変数や、`{% tag %}` のようなテンプレートタグを使って、動的にHTMLを生成できます。

*   **コンテキスト変数**: ビューの `render()` 関数に辞書形式で渡され、テンプレート内で `{{ key }}` の形式でアクセスできます。
*   **`for` ループ**: リストやクエリセットなどの反復可能なオブジェクトをループ処理し、各要素を表示します。
*   **`if` 文**: 条件に基づいてコンテンツの表示/非表示を切り替えます。

## 環境構築

このレッスンは、`mtv_lesson/lesson-2` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `mtv_lesson/lesson-2` ディレクトリに移動します。
    ```bash
    cd mtv_lesson/lesson-2
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: ビューからテンプレートへのデータ渡しと表示

ビューで定義したリストデータをテンプレートに渡し、`for`ループを使って表示しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `mtv_project` プロジェクト内に、`data_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp data_app
    ```

2.  **`settings.py` の設定**
    `mtv_project/settings.py` を開き、`INSTALLED_APPS` リストに `'data_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # mtv_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'data_app' を追加
        # 'data_app',
    ]
    ```

3.  **ビューの作成**
    `data_app/views.py` を開き、`item_list` という名前の関数を作成してください。この関数内で、いくつかのアイテムを含むリストを定義し、それをテンプレートに渡してください。

    ```python
    # data_app/views.py

    from django.shortcuts import render

    def item_list(request):
        items = [
            {'name': 'りんご', 'price': 100},
            {'name': 'バナナ', 'price': 150},
            {'name': 'みかん', 'price': 80},
        ]
        context = {
            # ここにitemsを渡す
            # 'items': items,
        }
        # ここでテンプレートをレンダリング
        # return render(request, 'data_app/item_list.html', context)
        pass
    ```

4.  **テンプレートファイルの作成**
    `data_app/templates/data_app/` ディレクトリを作成し、その中に `item_list.html` という名前で以下の内容を記述してください。`items` リストをループして、各アイテムの名前と価格を表示するようにします。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>商品リスト</title>
    </head>
    <body>
        <h1>商品リスト</h1>
        <ul>
            {% comment %} ここにforループを記述 {% endcomment %}
            {# 例: {% for item in items %}
            <li>{{ item.name }} - {{ item.price }}円</li>
            {% endfor %} #}
        </ul>
    </body>
    </html>
    ```

5.  **アプリケーションのURL設定**
    `data_app` ディレクトリ内に `urls.py` という新しいファイルを作成し、`item_list` ビューへのURLを設定してください。

    ```python
    # data_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('items/', views.item_list, name='item_list'),
    ]
    ```

6.  **プロジェクトのURL設定**
    `mtv_project/urls.py` を開き、`data_app` のURL設定をインクルードしてください。

    ```python
    # mtv_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        # ここにdata_appのURL設定をインクルード
        # path('data_app/', include('data_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/data_app/items/` にアクセスしてください。定義した商品リストがHTMLで表示されれば成功です。

## 課題2: `if` 文を使った条件分岐

商品リストに在庫切れの商品がある場合、その商品名の横に「(在庫切れ)」と表示するように変更しましょう。

### 手順

1.  **ビューの修正**
    `data_app/views.py` を開き、`items` リストに `stock` というキーを追加し、一部の商品を `stock: 0` に設定してください。

    ```python
    # data_app/views.py

    from django.shortcuts import render

    def item_list(request):
        items = [
            {'name': 'りんご', 'price': 100, 'stock': 10},
            {'name': 'バナナ', 'price': 150, 'stock': 0},
            {'name': 'みかん', 'price': 80, 'stock': 5},
        ]
        context = {
            'items': items,
        }
        return render(request, 'data_app/item_list.html', context)
    ```

2.  **テンプレートファイルの修正**
    `data_app/templates/data_app/item_list.html` を開き、`for`ループ内で `if` 文を使って `stock` が0の場合に「(在庫切れ)」と表示するように修正してください。

    ```html
    {# 例: {% for item in items %}
    <li>{{ item.name }} - {{ item.price }}円
        {% if item.stock == 0 %}
            (在庫切れ)
        {% endif %}
    </li>
    {% endfor %} #}
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/data_app/items/` に再度アクセスしてください。在庫が0の商品名の横に「(在庫切れ)」と表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
