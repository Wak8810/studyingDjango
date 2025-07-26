# Django学習レッスン: ORM (Object-Relational Mapper) - Lesson 4

## 目的

このレッスンでは、Django ORMにおけるリレーションシップ（関連）の扱い方について学びます。特に、一対多（ForeignKey）、多対多（ManyToManyField）、一対一（OneToOneField）のリレーションシップをモデルで定義し、関連するデータをクエリする方法を習得します。

## 概要

リレーショナルデータベースでは、テーブル間に関連を設定することで、データの整合性を保ち、効率的なデータ管理を実現します。Django ORMは、これらのリレーションシップをPythonのモデルで直感的に表現できます。

*	**一対多 (ForeignKey)**: 一つのオブジェクトが複数の他のオブジェクトに関連付けられる場合に使用します。例えば、一人の著者が複数の書籍を持つ場合などです。
*	**多対多 (ManyToManyField)**: 複数のオブジェクトが複数の他のオブジェクトに関連付けられる場合に使用します。例えば、一つの書籍が複数のタグを持つ場合や、一人の学生が複数のコースを受講する場合などです。
*	**一対一 (OneToOneField)**: 一つのオブジェクトが厳密に一つの他のオブジェクトに関連付けられる場合に使用します。例えば、ユーザーが一つだけプロフィールを持つ場合などです。

## 環境構築

このレッスンは、`orm_lesson/lesson-4` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.	`orm_lesson/lesson-4` ディレクトリに移動します。
	```bash
	cd orm_lesson/lesson-4
	```
2.	Dockerコンテナを起動します。
	```bash
	docker compose up -d
	```
	これにより、Django開発サーバーが起動します。

3.	**データベースマイグレーションの実行**
	ORMでデータベースを操作する前に、必要なテーブルを作成するためにマイグレーションを実行してください。
	```bash
	docker compose exec web python manage.py migrate
	```

## 課題1: 一対多 (ForeignKey) リレーションシップ

`Author` モデルと `Book` モデルを作成し、一人の著者が複数の書籍を持つ関係を定義しましょう。そして、関連するデータを取得します。

### 手順

1.	**Djangoアプリケーションの作成**
	`orm_project` プロジェクト内に、`library_relations` という名前の新しいDjangoアプリケーションを作成してください。
	```bash
	docker compose exec web python manage.py startapp library_relations
	```

2.	**`settings.py` の設定**
	`orm_project/settings.py` を開き、`INSTALLED_APPS` リストに `'library_relations'` を追加して、Djangoに新しいアプリケーションを認識させてください。

	```python
	# orm_project/settings.py

	INSTALLED_APPS = [
		# ... 既存のアプリ ...
		# ここに 'library_relations' を追加
		# 'library_relations',
	]
	```

3.	**モデルの定義**
	`library_relations/models.py` を開き、`Author` モデルと `Book` モデルを定義してください。`Book` モデルには `Author` モデルへの `ForeignKey` を設定します。

	```python
	# library_relations/models.py

	from django.db import models

	class Author(models.Model):
		name = models.CharField(max_length=100)

		def __str__(self):
			return self.name

	class Book(models.Model):
		title = models.CharField(max_length=200)
		# authorフィールドをForeignKeyとして定義
		# author = models.ForeignKey(Author, on_delete=models.CASCADE)

		def __str__(self):
			return self.title
	```

4.	**マイグレーションの作成と適用**
	モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成し、データベースに適用してください。

	```bash
	docker compose exec web python manage.py makemigrations library_relations
	docker compose exec web python manage.py migrate
	```

5.	**初期データの投入**
	Djangoシェルを使って、`Author` と `Book` のデータを投入してください。

	```bash
	docker compose exec web python manage.py shell
	```
	シェル内で以下のコードを実行してください。

	```python
	from library_relations.models import Author, Book

	author1 = Author.objects.create(name="山田太郎")
	author2 = Author.objects.create(name="鈴木花子")

	Book.objects.create(title="Python入門", author=author1)
	Book.objects.create(title="Django実践", author=author1)
	Book.objects.create(title="Web開発の基礎", author=author2)
	exit()
	```

6.	**関連データの取得**
	Djangoシェルを起動し、以下のコードを実行して関連するデータを取得してください。

	```bash
	docker compose exec web python manage.py shell
	```
	シェル内で以下のコードを実行してください。

	```python
	from library_relations.models import Author, Book

	# 特定の書籍の著者を取得
	# book = Book.objects.get(title="Python入門")
	# print(f"書籍: {book.title}, 著者: {book.author.name}")

	# 特定の著者の書籍をすべて取得
	# author = Author.objects.get(name="山田太郎")
	# print(f"\n著者: {author.name} の書籍一覧:")
	# for book in author.book_set.all(): # _set はリバースリレーションシップマネージャー
	# 	print(f"- {book.title}")

	exit()
	```

### 確認

シェルに期待通りの関連データが表示されれば成功です。

## 課題2: 多対多 (ManyToManyField) リレーションシップ

`Book` モデルと `Tag` モデルを作成し、一つの書籍が複数のタグを持ち、一つのタグが複数の書籍に関連付けられる関係を定義しましょう。そして、関連するデータを取得します。

### 手順

1.	**モデルの定義の修正**
	`library_relations/models.py` を開き、`Tag` モデルを定義し、`Book` モデルに `Tag` モデルへの `ManyToManyField` を設定してください。

	```python
	# library_relations/models.py

	from django.db import models

	class Tag(models.Model):
		name = models.CharField(max_length=50, unique=True)

		def __str__(self):
			return self.name

	class Book(models.Model):
		title = models.CharField(max_length=200)
		author = models.ForeignKey(Author, on_delete=models.CASCADE)
		# tagsフィールドをManyToManyFieldとして定義
		# tags = models.ManyToManyField(Tag)

		def __str__(self):
			return self.title
	```

2.	**マイグレーションの作成と適用**
	モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成し、データベースに適用してください。

	```bash
	docker compose exec web python manage.py makemigrations library_relations
	docker compose exec web python manage.py migrate
	```

3.	**初期データの投入**
	Djangoシェルを使って、`Tag` と `Book` のデータを投入し、関連付けを行ってください。

	```bash
	docker compose exec web python manage.py shell
	```
	シェル内で以下のコードを実行してください。

	```python
	from library_relations.models import Author, Book, Tag

	# 既存の著者を取得
	author1 = Author.objects.get(name="山田太郎")

	# タグを作成
	tag1 = Tag.objects.create(name="プログラミング")
	tag2 = Tag.objects.create(name="Web開発")
	tag3 = Tag.objects.create(name="Python")

	# 書籍を作成し、タグを関連付け
	book1 = Book.objects.create(title="Python入門", author=author1)
	book1.tags.add(tag1, tag3) # タグを追加

	book2 = Book.objects.create(title="Django実践", author=author1)
	book2.tags.add(tag1, tag2, tag3)
	exit()
	```

4.	**関連データの取得**
	Djangoシェルを起動し、以下のコードを実行して関連するデータを取得してください。

	```bash
	docker compose exec web python manage.py shell
	```
	シェル内で以下のコードを実行してください。

	```python
	from library_relations.models import Book, Tag

	# 特定の書籍のタグをすべて取得
	# book = Book.objects.get(title="Python入門")
	# print(f"書籍: {book.title} のタグ:")
	# for tag in book.tags.all():
	# 	print(f"- {tag.name}")

	# 特定のタグを持つ書籍をすべて取得
	# tag = Tag.objects.get(name="Web開発")
	# print(f"\nタグ: {tag.name} の書籍一覧:")
	# for book in tag.book_set.all():
	# 	print(f"- {book.title}")

	exit()
	```

### 確認

シェルに期待通りの関連データが表示されれば成功です。

## 課題3: 一対一 (OneToOneField) リレーションシップ

`User` モデルと `UserProfile` モデルを作成し、一人のユーザーが一つだけプロフィールを持つ関係を定義しましょう。そして、関連するデータを取得します。

### 手順

1.	**モデルの定義の修正**
	`library_relations/models.py` を開き、`UserProfile` モデルを定義し、Djangoの組み込み `User` モデルへの `OneToOneField` を設定してください。

	```python
	# library_relations/models.py

	from django.db import models
	from django.contrib.auth.models import User # Userモデルをインポート

	class UserProfile(models.Model):
		# userフィールドをOneToOneFieldとして定義
		# user = models.OneToOneField(User, on_delete=models.CASCADE)
		bio = models.TextField(blank=True)
		location = models.CharField(max_length=100, blank=True)

		def __str__(self):
			return self.user.username
	```

2.	**マイグレーションの作成と適用**
	モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成し、データベースに適用してください。

	```bash
	docker compose exec web python manage.py makemigrations library_relations
	docker compose exec web python manage.py migrate
	```

3.	**初期データの投入**
	Djangoシェルを使って、`User` と `UserProfile` のデータを投入し、関連付けを行ってください。

	```python
	from django.contrib.auth.models import User
	from library_relations.models import UserProfile

	# ユーザーを作成
	user = User.objects.create_user(username='testuser', password='password123')

	# ユーザープロフィールを作成し、ユーザーに関連付け
	profile = UserProfile.objects.create(user=user, bio="Django開発者", location="東京")
	exit()
	```

4.	**関連データの取得**
	Djangoシェルを起動し、以下のコードを実行して関連するデータを取得してください。

	```python
	from django.contrib.auth.models import User
	from library_relations.models import UserProfile

	# ユーザーからプロフィールを取得
	# user = User.objects.get(username='testuser')
	# print(f"ユーザー: {user.username}, プロフィール: {user.userprofile.bio}, {user.userprofile.location}")

	# プロフィールからユーザーを取得
	# profile = UserProfile.objects.get(location="東京")
	# print(f"プロフィール: {profile.location}, ユーザー: {profile.user.username}")

	exit()
	```

### 確認

シェルに期待通りの関連データが表示されれば成功です。

## 困った時は

*	Dockerコンテナが起動しているか確認してください: `docker compose ps`
*	エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*	`docker compose logs web` でコンテナのログを確認してください。
