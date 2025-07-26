# Django学習レッスン: ORM (Object-Relational Mapper) - Lesson 2

## 目的

このレッスンでは、Django ORMを使ったデータの更新（Update）と削除（Delete）操作について学びます。これにより、データベース内の既存のレコードを効率的に操作できるようになります。

## 概要

Django ORMは、Pythonオブジェクトを使ってデータベースのCRUD（Create, Read, Update, Delete）操作を直感的に行えるようにします。前回のレッスンではCreateとReadを学びましたので、今回はUpdateとDeleteに焦点を当てます。

*   **データの更新**: 既存のモデルインスタンスの属性を変更し、`save()` メソッドを呼び出すことでデータベースに反映させます。また、クエリセットに対して一括更新を行うことも可能です。
*   **データの削除**: モデルインスタンスの `delete()` メソッドを呼び出すことで、そのレコードをデータベースから削除します。クエリセットに対して一括削除を行うことも可能です。

## 環境構築

このレッスンは、`orm_lesson/lesson-2` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `orm_lesson/lesson-2` ディレクトリに移動します。
    ```bash
    cd orm_lesson/lesson-2
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

## 課題1: データの更新 (Update)

既存の `Product` モデルのデータをORMを使って更新しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `orm_project` プロジェクト内に、`shop_update` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp shop_update
    ```

2.  **`settings.py` の設定**
    `orm_project/settings.py` を開き、`INSTALLED_APPS` リストに `'shop_update'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # orm_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'shop_update' を追加
        # 'shop_update',
    ]
    ```

3.  **モデルの定義**
    `shop_update/models.py` を開き、`Product` という名前のモデルを作成してください。このモデルには、`name` (CharField), `price` (DecimalField), `stock` (IntegerField) のフィールドを持たせてください。

    ```python
    # shop_update/models.py

    from django.db import models

    class Product(models.Model):
        name = models.CharField(max_length=100)
        price = models.DecimalField(max_digits=10, decimal_places=2)
        stock = models.IntegerField(default=0)

        def __str__(self):
            return self.name
    ```

4.  **マイグレーションの作成と適用**
    モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成し、データベースに適用してください。

    ```bash
    docker compose exec web python manage.py makemigrations shop_update
    docker compose exec web python manage.py migrate
    ```

5.  **初期データの投入**
    Djangoシェルを使って、いくつかの `Product` データを投入してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from shop_update.models import Product
    Product.objects.create(name="りんご", price=150.00, stock=100)
    Product.objects.create(name="バナナ", price=120.50, stock=50)
    exit()
    ```

6.  **データの更新**
    Djangoシェルを起動し、以下のコードを実行して商品データを更新してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from shop_update.models import Product

    # IDが1の商品（りんご）を取得し、在庫を更新
    # try:
    #     apple = Product.objects.get(id=1)
    #     apple.stock = 120
    #     apple.save() # 変更を保存
    #     print(f"りんごの在庫を更新しました: {apple.stock}")
    # except Product.DoesNotExist:
    #     print("りんごが見つかりませんでした。")

    # 複数の商品を一括で更新 (例: すべての商品の価格を10%値上げ)
    # from django.db.models import F # Fオブジェクトをインポート
    # Product.objects.all().update(price=F('price') * 1.1)
    # print("すべての商品の価格を10%値上げしました。")

    exit()
    ```

### 確認

シェルで更新メッセージが表示された後、再度データを取得して変更が反映されていることを確認してください。

## 課題2: データの削除 (Delete)

ORMを使って、不要な商品データをデータベースから削除しましょう。

### 手順

1.  **データの削除**
    Djangoシェルを起動し、以下のコードを実行して商品データを削除してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from shop_update.models import Product

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
