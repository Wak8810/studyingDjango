# Django学習レッスン: マイグレーション - Lesson 2

## 目的

このレッスンでは、既存のモデルに新しいフィールドを追加する方法と、フィールドの変更（例: データ型の変更、null許容の変更）をマイグレーションで扱う方法を学びます。これにより、アプリケーションの進化に合わせてデータベーススキーマを安全に更新できるようになります。

## 概要

マイグレーションは、モデルの変更を検知し、データベースに適用するためのスクリプトを生成します。フィールドの追加、削除、変更など、様々なスキーマ変更に対応できます。

*   **フィールドの追加**: 新しいフィールドをモデルに追加し、`makemigrations` を実行すると、そのフィールドを追加するマイグレーションが生成されます。
*   **フィールドの変更**: 既存のフィールドの属性（`max_length`, `null`, `default` など）を変更した場合も、`makemigrations` が変更を検知し、適切なマイグレーションを生成します。

## 環境構築

このレッスンは、`migration_lesson/lesson-2` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `migration_lesson/lesson-2` ディレクトリに移動します。
    ```bash
    cd migration_lesson/lesson-2
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

## 課題1: 既存モデルへのフィールド追加

`Product` モデルに `description` フィールド（テキストエリア）を追加し、マイグレーションでデータベースに反映させましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `migration_project` プロジェクト内に、`store` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp store
    ```

2.  **`settings.py` の設定**
    `migration_project/settings.py` を開き、`INSTALLED_APPS` リストに `'store'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # migration_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'store' を追加
        # 'store',
    ]
    ```

3.  **モデルの定義**
    `store/models.py` を開き、`Product` という名前のモデルを作成してください。このモデルには、`name` (CharField), `price` (DecimalField) のフィールドを持たせてください。

    ```python
    # store/models.py

    from django.db import models

    class Product(models.Model):
        name = models.CharField(max_length=100)
        price = models.DecimalField(max_digits=10, decimal_places=2)

        def __str__(self):
            return self.name
    ```

4.  **初期マイグレーションの作成と適用**
    `Product` モデルの初期マイグレーションを作成し、適用してください。

    ```bash
    docker compose exec web python manage.py makemigrations store
    docker compose exec web python manage.py migrate
    ```

5.  **モデルの変更（フィールド追加）**
    `store/models.py` を開き、`Product` モデルに `description` フィールド（`models.TextField`, `null=True`, `blank=True`）を追加してください。

    ```python
    # store/models.py

    from django.db import models

    class Product(models.Model):
        name = models.CharField(max_length=100)
        price = models.DecimalField(max_digits=10, decimal_places=2)
        # ここにdescriptionフィールドを追加
        # description = models.TextField(null=True, blank=True)

        def __str__(self):
            return self.name
    ```

6.  **マイグレーションファイルの作成**
    変更されたモデルに基づいて、再度マイグレーションファイルを作成してください。

    ```bash
    docker compose exec web python manage.py makemigrations store
    ```

7.  **マイグレーションの適用**
    作成された新しいマイグレーションファイルをデータベースに適用してください。

    ```bash
    docker compose exec web python manage.py migrate
    ```

### 確認

マイグレーションが正常に適用されたことを確認するには、以下のコマンドでDjangoシェルを起動し、`Product` モデルに `description` フィールドが追加されていることを確認してください。

```bash
docker compose exec web python manage.py shell
```
シェル内で以下のコードを実行してください。

```python
from store.models import Product
# 新しいProductオブジェクトを作成し、descriptionを設定してみる
product = Product.objects.create(name="テスト商品", price=99.99, description="これはテスト用の説明です。")
print(product.description)
exit()
```

設定した説明が表示されれば成功です。

## 課題2: フィールドの変更

既存の `Product` モデルの `name` フィールドの `max_length` を変更し、マイグレーションでデータベースに反映させましょう。

### 手順

1.  **モデルの変更（フィールド属性変更）**
    `store/models.py` を開き、`Product` モデルの `name` フィールドの `max_length` を `100` から `200` に変更してください。

    ```python
    # store/models.py

    from django.db import models

    class Product(models.Model):
        name = models.CharField(max_length=200) # ここを100から200に変更
        price = models.DecimalField(max_digits=10, decimal_places=2)
        description = models.TextField(null=True, blank=True)

        def __str__(self):
            return self.name
    ```

2.  **マイグレーションファイルの作成**
    変更されたモデルに基づいて、再度マイグレーションファイルを作成してください。

    ```bash
    docker compose exec web python manage.py makemigrations store
    ```

3.  **マイグレーションの適用**
    作成された新しいマイグレーションファイルをデータベースに適用してください。

    ```bash
    docker compose exec web python manage.py migrate
    ```

### 確認

マイグレーションが正常に適用されたことを確認するには、以下のコマンドでDjangoシェルを起動し、`Product` モデルの `name` フィールドの `max_length` が変更されていることを確認してください。

```bash
docker compose exec web python manage.py shell
```
シェル内で以下のコードを実行してください。

```python
from store.models import Product
# フィールドのmax_lengthを確認
print(Product._meta.get_field('name').max_length)
exit()
```

`200` と表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
