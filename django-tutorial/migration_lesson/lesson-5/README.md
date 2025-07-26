# Django学習レッスン: マイグレーション - Lesson 5

## 目的

このレッスンでは、マイグレーションの高度なトピックとして、カスタムマイグレーション操作の作成と、マイグレーションの圧縮（squashing）について学びます。これにより、より複雑なデータベース変更シナリオに対応し、マイグレーション履歴を整理できるようになります。

## 概要

*   **カスタムマイグレーション操作**: `RunPython` や `RunSQL` 以外にも、独自のマイグレーション操作を定義することができます。これは、Djangoの標準的な操作では表現できないような、非常に特殊なデータベース変更が必要な場合に役立ちます。
*   **マイグレーションの圧縮 (Squashing)**: 多数のマイグレーションファイルを一つにまとめることで、マイグレーション履歴を簡潔にし、デプロイ時のパフォーマンスを向上させることができます。特に、開発中に頻繁にモデルを変更した場合に有効です。

## 環境構築

このレッスンは、`migration_lesson/lesson-5` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `migration_lesson/lesson-5` ディレクトリに移動します。
    ```bash
    cd migration_lesson/lesson-5
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

## 課題1: カスタムマイグレーション操作の作成

データベース内の特定のデータを一括で更新するカスタムマイグレーション操作を作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `migration_project` プロジェクト内に、`data_ops` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp data_ops
    ```

2.  **`settings.py` の設定**
    `migration_project/settings.py` を開き、`INSTALLED_APPS` リストに `'data_ops'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # migration_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'data_ops' を追加
        # 'data_ops',
    ]
    ```

3.  **モデルの定義**
    `data_ops/models.py` を開き、`Item` という名前のモデルを作成してください。このモデルには、`name` (CharField) と `status` (CharField, default='pending') のフィールドを持たせてください。

    ```python
    # data_ops/models.py

    from django.db import models

    class Item(models.Model):
        name = models.CharField(max_length=100)
        status = models.CharField(max_length=20, default='pending')

        def __str__(self):
            return self.name
    ```

4.  **初期マイグレーションの作成と適用**
    `Item` モデルの初期マイグレーションを作成し、適用してください。

    ```bash
    docker compose exec web python manage.py makemigrations data_ops
    docker compose exec web python manage.py migrate
    ```

5.  **データ投入**
    Djangoシェルを使って、いくつかの `Item` データを投入してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from data_ops.models import Item
    Item.objects.create(name="タスク1", status="pending")
    Item.objects.create(name="タスク2", status="pending")
    Item.objects.create(name="タスク3", status="completed")
    exit()
    ```

6.  **カスタムマイグレーション操作の記述**
    `data_ops` アプリケーションのマイグレーションディレクトリ（例: `data_ops/migrations/`）に、新しい空のマイグレーションファイルを作成します。

    ```bash
    docker compose exec web python manage.py makemigrations data_ops --empty --name update_pending_status
    ```

    生成されたファイル（例: `data_ops/migrations/0002_update_pending_status.py`）を開き、`operations` リストに `RunPython` 操作を追加して、`pending` ステータスのアイテムを `in_progress` に更新するロジックを記述してください。

    ```python
    # data_ops/migrations/0002_update_pending_status.py (一部抜粋)

    from django.db import migrations

    def update_status_to_in_progress(apps, schema_editor):
        Item = apps.get_model('data_ops', 'Item')
        for item in Item.objects.filter(status='pending'):
            item.status = 'in_progress'
            item.save()

    class Migration(migrations.Migration):

        dependencies = [
            ('data_ops', '0001_initial'), # 依存するマイグレーション
        ]

        operations = [
            # migrations.RunPython(update_status_to_in_progress),
        ]
    ```

7.  **マイグレーションの適用**
    カスタムマイグレーションを含むマイグレーションファイルをデータベースに適用してください。

    ```bash
    docker compose exec web python manage.py migrate
    ```

### 確認

マイグレーションが正常に適用されたことを確認するには、以下のコマンドでDjangoシェルを起動し、`pending` だったアイテムのステータスが `in_progress` に更新されていることを確認してください。

```bash
docker compose exec web python manage.py shell
```
シェル内で以下のコードを実行してください。

```python
from data_ops.models import Item
for item in Item.objects.all():
    print(f"Name: {item.name}, Status: {item.status}")
exit()
```

`タスク1: in_progress`, `タスク2: in_progress`, `タスク3: completed` のように表示されれば成功です。

## 課題2: マイグレーションの圧縮 (Squashing)

複数のマイグレーションファイルを一つに圧縮し、マイグレーション履歴を整理しましょう。

### 手順

1.  **モデルの変更を複数回行う**
    `data_ops/models.py` を開き、`Item` モデルに新しいフィールドをいくつか追加し、それぞれ `makemigrations` を実行して複数のマイグレーションファイルを作成してください。

    ```python
    # data_ops/models.py

    from django.db import models

    class Item(models.Model):
        name = models.CharField(max_length=100)
        status = models.CharField(max_length=20, default='pending')
        # ここに新しいフィールドを追加 (例: due_date = models.DateField(null=True, blank=True))
        # due_date = models.DateField(null=True, blank=True)
        # さらに別のフィールドを追加 (例: priority = models.IntegerField(default=1))
        # priority = models.IntegerField(default=1)

        def __str__(self):
            return self.name
    ```

    ```bash
    docker compose exec web python manage.py makemigrations data_ops
    # 繰り返す
    docker compose exec web python manage.py makemigrations data_ops
    ```

2.  **マイグレーションの圧縮**
    `data_ops` アプリケーションのマイグレーションを圧縮します。例えば、`0001_initial` から最新のマイグレーションまでを圧縮する場合、以下のように実行します。

    ```bash
    docker compose exec web python manage.py squashmigrations data_ops 0001
    ```

    このコマンドを実行すると、新しい圧縮されたマイグレーションファイルが生成され、元のマイグレーションファイルは非アクティブ化されます。指示に従って、新しいマイグレーションファイルを編集し、必要に応じて `RunPython` 操作などを追加してください。

3.  **圧縮されたマイグレーションの適用**
    圧縮されたマイグレーションファイルをデータベースに適用してください。

    ```bash
    docker compose exec web python manage.py migrate
    ```

### 確認

`showmigrations` コマンドでマイグレーション履歴を確認し、複数のマイグレーションが一つにまとまっていることを確認してください。

```bash
docker compose exec web python manage.py showmigrations data_ops
```

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
