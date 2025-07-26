# Django学習レッスン: ORM (Object-Relational Mapper) - Lesson 5

## 目的

このレッスンでは、Django ORMにおける集計（Aggregation）とアノテーション（Annotation）について学びます。これにより、データベースからより複雑な統計情報や計算結果を取得できるようになります。

## 概要

*   **集計 (Aggregation)**: クエリセット全体またはグループ化されたデータに対して、合計、平均、最大、最小、カウントなどの統計的な計算を行う機能です。`aggregate()` メソッドを使用します。
*   **アノテーション (Annotation)**: クエリセットの各オブジェクトに、追加の計算フィールド（アノテーション）を追加する機能です。例えば、各書籍のコメント数を計算して、その情報を書籍オブジェクトに含めることができます。`annotate()` メソッドを使用します。

## 環境構築

このレッスンは、`orm_lesson/lesson-5` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `orm_lesson/lesson-5` ディレクトリに移動します。
    ```bash
    cd orm_lesson/lesson-5
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

## 課題1: 集計 (Aggregation)

`Order` モデルと `OrderItem` モデルを作成し、注文の合計金額や、特定の商品が注文された総数などを集計しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `orm_project` プロジェクト内に、`sales` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp sales
    ```

2.  **`settings.py` の設定**
    `orm_project/settings.py` を開き、`INSTALLED_APPS` リストに `'sales'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # orm_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'sales' を追加
        # 'sales',
    ]
    ```

3.  **モデルの定義**
    `sales/models.py` を開き、`Product`、`Order`、`OrderItem` モデルを定義してください。`Order` は複数の `OrderItem` を持ち、`OrderItem` は `Product` に関連付けられます。

    ```python
    # sales/models.py

    from django.db import models

    class Product(models.Model):
        name = models.CharField(max_length=100)
        price = models.DecimalField(max_digits=10, decimal_places=2)

        def __str__(self):
            return self.name

    class Order(models.Model):
        order_date = models.DateTimeField(auto_now_add=True)

        def __str__(self):
            return f"Order {self.id} on {self.order_date.strftime('%Y-%m-%d')}"

    class OrderItem(models.Model):
        order = models.ForeignKey(Order, on_delete=models.CASCADE)
        product = models.ForeignKey(Product, on_delete=models.CASCADE)
        quantity = models.IntegerField()

        def __str__(self):
            return f"{self.product.name} ({self.quantity})"
    ```

4.  **マイグレーションの作成と適用**
    モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成し、データベースに適用してください。

    ```bash
    docker compose exec web python manage.py makemigrations sales
    docker compose exec web python manage.py migrate
    ```

5.  **初期データの投入**
    Djangoシェルを使って、いくつかの `Product`、`Order`、`OrderItem` データを投入してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from sales.models import Product, Order, OrderItem

    # 商品を作成
    product1 = Product.objects.create(name="りんご", price=100.00)
    product2 = Product.objects.create(name="バナナ", price=150.00)
    product3 = Product.objects.create(name="みかん", price=80.00)

    # 注文を作成
    order1 = Order.objects.create()
    OrderItem.objects.create(order=order1, product=product1, quantity=2)
    OrderItem.objects.create(order=order1, product=product2, quantity=1)

    order2 = Order.objects.create()
    OrderItem.objects.create(order=order2, product=product1, quantity=3)
    OrderItem.objects.create(order=order2, product=product3, quantity=5)
    exit()
    ```

6.  **データの集計**
    Djangoシェルを起動し、以下のコードを実行してデータを集計してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from sales.models import Product, Order, OrderItem
    from django.db.models import Sum, Avg, Count, Max, Min

    # すべての注文の合計金額を計算
    # total_sales = OrderItem.objects.aggregate(total_price=Sum('product__price'))
    # print(f"総売上: {total_sales['total_price']}円")

    # 特定の商品（りんご）が注文された総数を計算
    # apple_ordered_count = OrderItem.objects.filter(product__name='りんご').aggregate(total_quantity=Sum('quantity'))
    # print(f"りんごの総注文数: {apple_ordered_count['total_quantity']}個")

    exit()
    ```

### 確認

シェルに期待通りの集計結果が表示されれば成功です。

## 課題2: アノテーション (Annotation)

各注文の合計金額を計算し、その情報を注文オブジェクトにアノテーションとして追加しましょう。

### 手順

1.  **データの集計とアノテーション**
    Djangoシェルを起動し、以下のコードを実行してデータをアノテーションしてください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from sales.models import Product, Order, OrderItem
    from django.db.models import Sum, F

    # 各注文の合計金額を計算し、アノテーションとして追加
    # orders_with_total = Order.objects.annotate(total_amount=Sum(F('orderitem__quantity') * F('orderitem__product__price')))
    # print("--- 各注文の合計金額 ---")
    # for order in orders_with_total:
    #     print(f"注文ID: {order.id}, 合計金額: {order.total_amount}円")

    exit()
    ```

### 確認

シェルに期待通りのアノテーション結果が表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
