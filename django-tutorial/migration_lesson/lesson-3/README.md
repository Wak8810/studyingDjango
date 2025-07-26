# Django学習レッスン: マイグレーション - Lesson 3

## 目的

このレッスンでは、マイグレーションにおけるフィールドの削除と、データマイグレーション（データベースのスキーマだけでなく、データ自体を変更するマイグレーション）について学びます。これにより、より複雑なデータベース変更シナリオに対応できるようになります。

## 概要

*   **フィールドの削除**: モデルからフィールドを削除し、`makemigrations` を実行すると、そのフィールドを削除するマイグレーションが生成されます。データが失われる可能性があるため、注意が必要です。
*   **データマイグレーション**: `RunPython` や `RunSQL` 操作を使って、データベースのスキーマ変更だけでなく、既存のデータ自体を変更するロジックをマイグレーションに含めることができます。これは、データ形式の変更や、データの移行が必要な場合に特に役立ちます。

## 環境構築

このレッスンは、`migration_lesson/lesson-3` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `migration_lesson/lesson-3` ディレクトリに移動します。
    ```bash
    cd migration_lesson/lesson-3
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

3.  **データベースマイグレーションの実行**
    このレッスンを開始する前に、既存のマイグレーションを適用してください。
    ```bash
    docker compose exec web python manage.py migrate
    ```

## 課題1: フィールドの削除

`Product` モデルから `description` フィールドを削除し、マイグレーションでデータベースに反映させましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `migration_project` プロジェクト内に、`inventory` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp inventory
    ```

2.  **`settings.py` の設定**
    `migration_project/settings.py` を開き、`INSTALLED_APPS` リストに `'inventory'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # migration_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'inventory' を追加
        # 'inventory',
    ]
    ```

3.  **モデルの定義**
    `inventory/models.py` を開き、`Product` という名前のモデルを作成してください。このモデルには、`name` (CharField), `price` (DecimalField), `description` (TextField) のフィールドを持たせてください。

    ```python
    # inventory/models.py

    from django.db import models

    class Product(models.Model):
        name = models.CharField(max_length=100)
        price = models.DecimalField(max_digits=10, decimal_places=2)
        description = models.TextField(null=True, blank=True)

        def __str__(self):
            return self.name
    ```

4.  **初期マイグレーションの作成と適用**
    `Product` モデルの初期マイグレーションを作成し、適用してください。

    ```bash
    docker compose exec web python manage.py makemigrations inventory
    docker compose exec web python manage.py migrate
    ```

5.  **モデルの変更（フィールド削除）**
    `inventory/models.py` を開き、`Product` モデルから `description` フィールドを削除してください。

    ```python
    # inventory/models.py

    from django.db import models

    class Product(models.Model):
        name = models.CharField(max_length=100)
        price = models.DecimalField(max_digits=10, decimal_places=2)
        # descriptionフィールドを削除

        def __str__(self):
            return self.name
    ```

6.  **マイグレーションファイルの作成**
    変更されたモデルに基づいて、再度マイグレーションファイルを作成してください。フィールドを削除するマイグレーションが生成されます。

    ```bash
    docker compose exec web python manage.py makemigrations inventory
    ```

7.  **マイグレーションの適用**
    作成された新しいマイグレーションファイルをデータベースに適用してください。

    ```bash
    docker compose exec web python manage.py migrate
    ```

### 確認

マイグレーションが正常に適用されたことを確認するには、以下のコマンドでDjangoシェルを起動し、`Product` モデルから `description` フィールドが削除されていることを確認してください。

```bash
docker compose exec web python manage.py shell
```
シェル内で以下のコードを実行してください。

```python
from inventory.models import Product
# フィールドが存在しないことを確認
try:
    Product._meta.get_field('description')
except Exception as e:
    print(f"descriptionフィールドは存在しません: {e}")
exit()
```

`descriptionフィールドは存在しません` のようなメッセージが表示されれば成功です。

## 課題2: データマイグレーション

既存の `Product` モデルに `full_name` フィールドを追加し、既存の `name` フィールドのデータを `full_name` にコピーするデータマイグレーションを作成しましょう。

### 手順

1.  **モデルの変更（フィールド追加）**
    `inventory/models.py` を開き、`Product` モデルに `full_name` (CharField, `max_length=200`, `null=True`, `blank=True`) フィールドを追加してください。

    ```python
    # inventory/models.py

    from django.db import models

    class Product(models.Model):
        name = models.CharField(max_length=100)
        price = models.DecimalField(max_digits=10, decimal_places=2)
        # ここにfull_nameフィールドを追加
        # full_name = models.CharField(max_length=200, null=True, blank=True)

        def __str__(self):
            return self.name
    ```

2.  **マイグレーションファイルの作成**
    変更されたモデルに基づいて、マイグレーションファイルを作成してください。

    ```bash
    docker compose exec web python manage.py makemigrations inventory
    ```

3.  **データマイグレーションの記述**
    生成されたマイグレーションファイル（例: `inventory/migrations/000X_auto_YYYYMMDD_HHMM.py`）を開き、`operations` リストに `RunPython` 操作を追加して、既存の `name` データを `full_name` にコピーするロジックを記述してください。

    ```python
    # inventory/migrations/000X_auto_YYYYMMDD_HHMM.py (一部抜粋)

    from django.db import migrations, models

    def copy_name_to_full_name(apps, schema_editor):
        Product = apps.get_model('inventory', 'Product')
        for product in Product.objects.all():
            product.full_name = product.name
            product.save()

    class Migration(migrations.Migration):

        dependencies = [
            # ...
        ]

        operations = [
            # ... AddField操作など ...
            # migrations.RunPython(copy_name_to_full_name),
        ]
    ```

4.  **マイグレーションの適用**
    データマイグレーションを含むマイグレーションファイルをデータベースに適用してください。

    ```bash
    docker compose exec web python manage.py migrate
    ```

### 確認

マイグレーションが正常に適用されたことを確認するには、以下のコマンドでDjangoシェルを起動し、`Product` モデルの `full_name` フィールドに `name` のデータがコピーされていることを確認してください。

```bash
docker compose exec web python manage.py shell
```
シェル内で以下のコードを実行してください。

```python
from inventory.models import Product
# データをいくつか作成しておく
Product.objects.create(name="商品A", price=100)
Product.objects.create(name="商品B", price=200)

# full_nameがコピーされていることを確認
for product in Product.objects.all():
    print(f"Name: {product.name}, Full Name: {product.full_name}")
exit()
```

`Name: 商品A, Full Name: 商品A` のように表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
