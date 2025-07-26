# Django学習レッスン: マイグレーション - Lesson 4

## 目的

このレッスンでは、マイグレーションの依存関係と、マイグレーションの取り消し（リバート）について学びます。これにより、より複雑なプロジェクトでのマイグレーション管理や、開発中のデータベーススキーマの調整を安全に行えるようになります。

## 概要

*   **依存関係**: マイグレーションは、他のマイグレーションに依存する場合があります。例えば、あるアプリのモデルが別のアプリのモデルを参照している場合などです。Djangoは自動的に依存関係を解決しますが、手動で調整が必要な場合もあります。
*   **リバート**: `migrate` コマンドにマイグレーション名を指定することで、特定のマイグレーションまでデータベースの状態を戻すことができます。これは、開発中にスキーマの変更を元に戻したい場合や、デプロイ後に問題が見つかった場合に役立ちます。

## 環境構築

このレッスンは、`migration_lesson/lesson-4` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `migration_lesson/lesson-4` ディレクトリに移動します。
    ```bash
    cd migration_lesson/lesson-4
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

## 課題1: マイグレーションの依存関係

2つのアプリケーション間でモデルが関連し合う場合のマイグレーションの依存関係を確認しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `migration_project` プロジェクト内に、`authors` と `books` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp authors
    docker compose exec web python manage.py startapp books
    ```

2.  **`settings.py` の設定**
    `migration_project/settings.py` を開き、`INSTALLED_APPS` リストに `'authors'` と `'books'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # migration_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'authors' と 'books' を追加
        # 'authors',
        # 'books',
    ]
    ```

3.  **モデルの定義**
    `authors/models.py` を開き、`Author` モデルを定義してください。`books/models.py` を開き、`Book` モデルを定義し、`Author` モデルへの外部キー（ForeignKey）を設定してください。

    ```python
    # authors/models.py

    from django.db import models

    class Author(models.Model):
        name = models.CharField(max_length=100)

        def __str__(self):
            return self.name
    ```

    ```python
    # books/models.py

    from django.db import models
    from authors.models import Author # Authorモデルをインポート

    class Book(models.Model):
        title = models.CharField(max_length=200)
        # authorフィールドをForeignKeyとして定義
        # author = models.ForeignKey(Author, on_delete=models.CASCADE)

        def __str__(self):
            return self.title
    ```

4.  **マイグレーションファイルの作成**
    両方のアプリケーションに対してマイグレーションファイルを作成してください。

    ```bash
    docker compose exec web python manage.py makemigrations authors
    docker compose exec web python manage.py makemigrations books
    ```

    `books` アプリケーションのマイグレーションファイル（例: `books/migrations/0001_initial.py`）を開き、`dependencies` に `authors` アプリケーションのマイグレーションが追加されていることを確認してください。

5.  **マイグレーションの適用**
    すべてのマイグレーションをデータベースに適用してください。

    ```bash
    docker compose exec web python manage.py migrate
    ```

### 確認

マイグレーションが正常に適用されたことを確認するには、以下のコマンドでDjangoシェルを起動し、`Author` と `Book` モデルが連携して動作することを確認してください。

```bash
docker compose exec web python manage.py shell
```
シェル内で以下のコードを実行してください。

```python
from authors.models import Author
from books.models import Book

author = Author.objects.create(name="山田太郎")
book = Book.objects.create(title="Djangoの教科書", author=author)

print(f"書籍: {book.title}, 著者: {book.author.name}")
exit()
```

書籍と著者の情報が表示されれば成功です。

## 課題2: マイグレーションのリバート

作成したマイグレーションを元に戻し、データベースの状態を以前のバージョンに戻しましょう。

### 手順

1.  **マイグレーションの確認**
    現在のマイグレーションの状態を確認します。

    ```bash
    docker compose exec web python manage.py showmigrations
    ```

    `[X]` と表示されているマイグレーションが適用済みです。

2.  **マイグレーションのリバート**
    `books` アプリケーションの最新のマイグレーションを取り消してください。例えば、`0001_initial` を取り消す場合は、以下のように実行します。

    ```bash
    docker compose exec web python manage.py migrate books 0000_empty
    ```
    または、特定のマイグレーション名まで戻すこともできます。

3.  **マイグレーションの状態確認**
    再度マイグレーションの状態を確認し、`books` アプリケーションのマイグレーションが取り消されていることを確認してください。

    ```bash
    docker compose exec web python manage.py showmigrations
    ```

### 確認

`books` アプリケーションのマイグレーションが `[ ]` と表示されていれば、リバートは成功です。その後、Djangoシェルで `Book` モデルがインポートできなくなっていることを確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
