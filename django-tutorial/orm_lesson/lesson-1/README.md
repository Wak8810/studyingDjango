# Django学習レッスン: ORM (Object-Relational Mapper) - Lesson 1

## 目的

このレッスンでは、DjangoのORM (Object-Relational Mapper) について学びます。SQLを直接書く代わりに、Pythonのコードを使ってデータベースを操作する方法を習得します。データの作成、読み取り、更新、削除 (CRUD) の基本的な操作を実践します。

## ORMの概要

ORMは、データベースのテーブルとPythonのオブジェクト（クラスのインスタンス）をマッピングする技術です。これにより、開発者はSQLクエリを直接書く代わりに、Pythonのオブジェクト指向の概念を使ってデータベースを操作できるようになります。

DjangoのORMは非常に強力で、以下のようなメリットがあります。

*   **生産性の向上**: SQLを直接書く手間が省け、Pythonコードに集中できます。
*   **可読性の向上**: データベース操作がPythonのオブジェクトとして表現されるため、コードが読みやすくなります。
*   **データベースの抽象化**: データベースの種類（SQLite, PostgreSQL, MySQLなど）を意識せずにコードを書くことができます。
*   **セキュリティ**: SQLインジェクションなどのセキュリティリスクを軽減します。

## 環境構築

このレッスンは、`orm_lesson/lesson-1` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `orm_lesson/lesson-1` ディレクトリに移動します。
    ```bash
    cd or_lesson/lesson-1
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

3.  **データベースマイグレーションの実行**
    ORMでデータベースを操作する前に、必要なテーブルを作成するためにマイグレーションを実行してください。
    ```bash
    docker compose exec web python manage.py migrate
    ```

## 課題1: モデルの定義とデータの作成 (Create)

シンプルな `Product` モデルを作成し、ORMを使ってデータベースに新しい商品データを保存しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `orm_project` プロジェクト内に、`shop` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp shop
    ```

2.  **`settings.py` の設定**
    `orm_project/settings.py` を開き、`INSTALLED_APPS` リストに `'shop'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # orm_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'shop' を追加
        # 'shop',
    ]
    ```

3.  **モデルの定義**
    `shop/models.py` を開き、`Product` という名前のモデルを作成してください。このモデルには、`name` (CharField), `price` (DecimalField), `stock` (IntegerField) のフィールドを持たせてください。

    ```python
    # shop/models.py

    from django.db import models

    class Product(models.Model):
        # nameフィールド
        # name = models.CharField(max_length=100)
        
        # priceフィールド (DecimalFieldは通貨などの正確な数値を扱うのに適しています)
        # price = models.DecimalField(max_digits=10, decimal_places=2)
        
        # stockフィールド
        # stock = models.IntegerField(default=0)

        def __str__(self):
            return self.name
    ```

4.  **マイグレーションの作成と適用**
    モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成し、データベースに適用してください。

    ```bash
    docker compose exec web python manage.py makemigrations shop
    docker compose exec web python manage.py migrate
    ```

5.  **データの作成**
    Djangoシェルを使って、`Product` モデルにデータを投入してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from shop.models import Product

    # Productオブジェクトを作成し、保存
    # product1 = Product(name="りんご", price=150.00, stock=100)
    # product1.save()

    # product2 = Product.objects.create(name="バナナ", price=120.50, stock=50)

    print("2つの商品データを保存しました。")
    exit()
    ```

### 確認

シェルで「2つの商品データを保存しました。」と表示されれば成功です。

## 課題2: データの読み取り (Read)

ORMを使って、データベースに保存された商品データを取得し、表示しましょう。

### 手順

1.  **データの取得**
    Djangoシェルを起動し、以下のコードを実行して商品データを取得・表示してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from shop.models import Product

    # すべての商品を取得
    # all_products = Product.objects.all()
    # print("--- すべての商品 ---")
    # for product in all_products:
    #     print(f"ID: {product.id}, 名前: {product.name}, 価格: {product.price}, 在庫: {product.stock}")

    # 特定の条件で商品を取得 (例: 価格が100円以上のもの)
    # expensive_products = Product.objects.filter(price__gte=100)
    # print("\n--- 価格が100円以上の商品 ---")
    # for product in expensive_products:
    #     print(f"名前: {product.name}, 価格: {product.price}")

    # 主キーで単一の商品を取得
    # try:
    #     banana = Product.objects.get(name="バナナ")
    #     print(f"\n--- バナナの情報 ---")
    #     print(f"名前: {banana.name}, 在庫: {banana.stock}")
    # except Product.DoesNotExist:
    #     print("バナナは見つかりませんでした。")

    exit()
    ```

### 確認

シェルに商品データが正しく表示されれば成功です。

## 課題3: データの更新 (Update)

既存の商品データをORMを使って更新しましょう。

### 手順

1.  **データの更新**
    Djangoシェルを起動し、以下のコードを実行して商品データを更新してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from shop.models import Product

    # IDが1の商品（りんご）を取得し、在庫を更新
    # try:
    #     apple = Product.objects.get(id=1)
    #     apple.stock = 120
    #     apple.save() # 変更を保存
    #     print(f"りんごの在庫を更新しました: {apple.stock}")
    # except Product.DoesNotExist:
    #     print("りんごが見つかりませんでした。")

    # 複数の商品を一括で更新 (例: すべての商品の価格を10%値上げ)
    # Product.objects.all().update(price=models.F('price') * 1.1)
    # print("すべての商品の価格を10%値上げしました。")

    exit()
    ```

### 確認

シェルで更新メッセージが表示された後、再度データを取得して変更が反映されていることを確認してください。

## 課題4: データの削除 (Delete)

ORMを使って、不要な商品データをデータベースから削除しましょう。

### 手順

1.  **データの削除**
    Djangoシェルを起動し、以下のコードを実行して商品データを削除してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from shop.models import Product

    # 名前が「バナナ」の商品を削除
    # try:
    #     banana = Product.objects.get(name="バナナ")
    #     banana.delete()
    #     print("バナナを削除しました。")
    # except Product.DoesNotExist:
    #     print("バナナは見つかりませんでした。")

    # すべての商品を削除
    # Product.objects.all().delete()
    # print("すべての商品を削除しました。")

    exit()
    ```

### 確認

シェルで削除メッセージが表示された後、再度データを取得して削除が反映されていることを確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
*   `docker compose exec web python manage.py shell` でシェルに入り、モデルのインポートやデータの確認を試してみてください。
