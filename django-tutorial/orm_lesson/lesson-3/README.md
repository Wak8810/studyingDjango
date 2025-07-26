# Django学習レッスン: ORM (Object-Relational Mapper) - Lesson 3

## 目的

このレッスンでは、Django ORMにおけるクエリセットの高度な操作、特にフィルタリング、ソート、チェーンについて学びます。これにより、より複雑な条件でデータを取得し、効率的に操作できるようになります。

## 概要

クエリセットは、データベースからオブジェクトのリストを取得するためのDjango ORMの強力なツールです。クエリセットは遅延評価され、様々なメソッドをチェーン（連結）して、より複雑なクエリを構築できます。

*   **フィルタリング (`filter()`, `exclude()`)**: 特定の条件に一致するオブジェクトを絞り込みます。フィールドルックアップ（`__gt`, `__contains` など）を使って、より柔軟な条件を指定できます。
*   **ソート (`order_by()`)**: 取得したオブジェクトを指定したフィールドでソートします。
*   **チェーン**: 複数のクエリセットメソッドを連結して呼び出すことで、複雑なクエリを構築できます。

## 環境構築

このレッスンは、`orm_lesson/lesson-3` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `orm_lesson/lesson-3` ディレクトリに移動します。
    ```bash
    cd or_lesson/lesson-3
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

## 課題1: 高度なフィルタリングとソート

`Book` モデルを作成し、特定の条件で書籍をフィルタリングしたり、ソートしたりするクエリを実行しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `orm_project` プロジェクト内に、`library` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp library
    ```

2.  **`settings.py` の設定**
    `orm_project/settings.py` を開き、`INSTALLED_APPS` リストに `'library'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # orm_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'library' を追加
        # 'library',
    ]
    ```

3.  **モデルの定義**
    `library/models.py` を開き、`Book` という名前のモデルを作成してください。このモデルには、`title` (CharField), `author` (CharField), `published_date` (DateField), `pages` (IntegerField) のフィールドを持たせてください。

    ```python
    # library/models.py

    from django.db import models

    class Book(models.Model):
        title = models.CharField(max_length=200)
        author = models.CharField(max_length=100)
        published_date = models.DateField()
        pages = models.IntegerField()

        def __str__(self):
            return self.title
    ```

4.  **マイグレーションの作成と適用**
    モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成し、データベースに適用してください。

    ```bash
    docker compose exec web python manage.py makemigrations library
    docker compose exec web python manage.py migrate
    ```

5.  **初期データの投入**
    Djangoシェルを使って、いくつかの `Book` データを投入してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from library.models import Book
    from datetime import date

    Book.objects.create(title="Python入門", author="山田太郎", published_date=date(2022, 1, 1), pages=300)
    Book.objects.create(title="Django実践", author="鈴木花子", published_date=date(2023, 5, 10), pages=450)
    Book.objects.create(title="Web開発の基礎", author="山田太郎", published_date=date(2021, 11, 20), pages=250)
    Book.objects.create(title="データベース設計", author="田中一郎", published_date=date(2023, 2, 1), pages=380)
    exit()
    ```

6.  **データのフィルタリングとソート**
    Djangoシェルを起動し、以下のコードを実行して書籍データをフィルタリング・ソートしてください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from library.models import Book
    from datetime import date

    # 2023年以降に出版された書籍をタイトル順で取得
    # recent_books = Book.objects.filter(published_date__gte=date(2023, 1, 1)).order_by('title')
    # print("--- 2023年以降に出版された書籍（タイトル順）---")
    # for book in recent_books:
    #     print(f"タイトル: {book.title}, 出版日: {book.published_date}")

    # ページ数が350ページ以上の書籍を著者名の逆順で取得
    # long_books = Book.objects.filter(pages__gte=350).order_by('-author')
    # print("\n--- 350ページ以上の書籍（著者名逆順）---")
    # for book in long_books:
    #     print(f"タイトル: {book.title}, 著者: {book.author}, ページ数: {book.pages}")

    exit()
    ```

### 確認

シェルに期待通りのフィルタリングとソート結果が表示されれば成功です。

## 課題2: クエリセットのチェーン

複数のフィルタリング条件をチェーンして、より複雑なクエリを構築しましょう。

### 手順

1.  **クエリセットのチェーン**
    Djangoシェルを起動し、以下のコードを実行して複数の条件をチェーンして書籍データを取得してください。

    ```bash
    docker compose exec web python manage.py shell
    ```
    シェル内で以下のコードを実行してください。

    ```python
    from library.models import Book
    from datetime import date

    # 著者「山田太郎」が書いた、かつ2022年以降に出版された書籍をページ数の昇順で取得
    # yamada_recent_books = Book.objects.filter(author="山田太郎").filter(published_date__gte=date(2022, 1, 1)).order_by('pages')
    # print("--- 山田太郎が書いた2022年以降の書籍（ページ数昇順）---")
    # for book in yamada_recent_books:
    #     print(f"タイトル: {book.title}, 出版日: {book.published_date}, ページ数: {book.pages}")

    exit()
    ```

### 確認

シェルに期待通りのチェーンされたクエリ結果が表示されれば成功です。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
