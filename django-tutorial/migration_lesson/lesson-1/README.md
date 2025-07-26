# Django学習レッスン: マイグレーション - Lesson 1

## 目的

このレッスンでは、Djangoのマイグレーション機能について学びます。データベースのスキーマ変更をバージョン管理し、安全に適用する方法を習得します。モデルの変更、マイグレーションファイルの作成、そしてデータベースへの適用を実践します。

## マイグレーションの概要

マイグレーションとは、データベースのスキーマ（テーブルの構造、フィールドの種類、関係性など）の変更を、バージョン管理された形で管理するためのDjangoの機能です。

アプリケーションを開発していくと、新しい機能を追加したり、既存の機能を変更したりする際に、データベースの構造も変更する必要が出てきます。例えば、新しいユーザー情報を保存するためにテーブルに列を追加したり、既存の列のデータ型を変更したりするような場合です。

マイグレーションを使うことで、これらのデータベースの変更をPythonのコードとして記述し、Gitなどのバージョン管理システムで管理できます。これにより、複数の開発者が協力して開発を進める際や、本番環境にデプロイする際に、データベースの変更を安全かつ確実に適用できるようになります。

## 環境構築

このレッスンは、`migration_lesson/lesson-1` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `migration_lesson/lesson-1` ディレクトリに移動します。
    ```bash
    cd migration_lesson/lesson-1
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: 新しいモデルの作成とマイグレーション

書籍情報を保存するための `Book` モデルを作成し、そのモデルに対応するデータベーステーブルをマイグレーションで作成しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `migration_project` プロジェクト内に、`books` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp books
    ```

2.  **`settings.py` の設定**
    `migration_project/settings.py` を開き、`INSTALLED_APPS` リストに `'books'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # migration_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'books' を追加
        # 'books',
    ]
    ```

3.  **モデルの定義**
    `books/models.py` を開き、`Book` という名前のモデルを作成してください。このモデルには、`title` (CharField), `author` (CharField), `published_date` (DateField) のフィールドを持たせてください。

    ```python
    # books/models.py

    from django.db import models

    class Book(models.Model):
        # titleフィールド
        # title = models.CharField(max_length=200)
        
        # authorフィールド
        # author = models.CharField(max_length=100)
        
        # published_dateフィールド
        # published_date = models.DateField()

        def __str__(self):
            return self.title
    ```

4.  **マイグレーションファイルの作成**
    モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成してください。

    ```bash
    docker compose exec web python manage.py makemigrations books
    ```

5.  **マイグレーションの適用**
    作成されたマイグレーションファイルをデータベースに適用してください。

    ```bash
    docker compose exec web python manage.py migrate
    ```

### 確認

マイグレーションが正常に適用されたことを確認するには、以下のコマンドでDjangoシェルを起動し、`Book` モデルがインポートできることを確認してください。

```bash
docker compose exec web python manage.py shell
```
シェル内で以下のコードを実行してください。

```python
from books.models import Book
print(Book)
exit()
```

`<class 'books.models.Book'>` と表示されれば成功です。

## 課題2: モデルの変更とマイグレーション

既存の `Book` モデルに新しいフィールドを追加し、その変更をマイグレーションでデータベースに反映させましょう。

### 手順

1.  **モデルの変更**
    `books/models.py` を開き、`Book` モデルに `isbn` (CharField, unique=True, null=True, blank=True) フィールドを追加してください。

    ```python
    # books/models.py

    from django.db import models

    class Book(models.Model):
        title = models.CharField(max_length=200)
        author = models.CharField(max_length=100)
        published_date = models.DateField()
        # ここにisbnフィールドを追加
        # isbn = models.CharField(max_length=13, unique=True, null=True, blank=True)

        def __str__(self):
            return self.title
    ```

2.  **マイグレーションファイルの作成**
    変更されたモデルに基づいて、再度マイグレーションファイルを作成してください。

    ```bash
    docker compose exec web python manage.py makemigrations books
    ```

3.  **マイグレーションの適用**
    作成された新しいマイグレーションファイルをデータベースに適用してください。

    ```bash
    docker compose exec web python manage.py migrate
    ```

### 確認

マイグレーションが正常に適用されたことを確認するには、以下のコマンドでDjangoシェルを起動し、`Book` モデルに `isbn` フィールドが追加されていることを確認してください。

```bash
docker compose exec web python manage.py shell
```
シェル内で以下のコードを実行してください。

```python
from books.models import Book
# 新しいBookオブジェクトを作成し、isbnを設定してみる
book = Book(title="新しい本", author="テスト太郎", published_date="2024-01-01", isbn="978-1234567890")
book.save()
print(book.isbn)
exit()
```

設定したISBNが表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
*   `docker compose exec web python manage.py shell` でシェルに入り、モデルのインポートやデータの確認を試してみてください。
