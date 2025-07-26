# Django学習レッスン: バリデーション - Lesson 5

## 目的

このレッスンでは、Djangoのフォームにおけるバリデーションエラーの国際化（i18n）と、フォームセットを使った複数のフォームのバリデーションについて学びます。これにより、多言語対応のアプリケーションでユーザーフレンドリーなエラーメッセージを提供し、複数の関連するフォームを効率的に扱えるようになります。

## 概要

*   **バリデーションエラーの国際化**: Djangoは、エラーメッセージを翻訳するための仕組みを提供しています。これにより、異なる言語のユーザーに対して、適切な言語でエラーメッセージを表示できます。
*   **フォームセット**: 複数の同じ種類のフォームを一度に処理するための機能です。例えば、複数の商品アイテムを一度に入力するような場合に便利です。

## 環境構築

このレッスンは、`validation_lesson/lesson-5` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `validation_lesson/lesson-5` ディレクトリに移動します。
    ```bash
    cd validation_lesson/lesson-5
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: バリデーションエラーの国際化

フォームのエラーメッセージを日本語に翻訳できるように設定し、動作を確認しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `validation_project` プロジェクト内に、`i18n_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp i18n_app
    ```

2.  **`settings.py` の設定**
    `validation_project/settings.py` を開き、`INSTALLED_APPS` リストに `'i18n_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。また、`LANGUAGE_CODE` を `'ja'` に、`USE_I18N` を `True` に設定してください。

    ```python
    # validation_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'i18n_app' を追加
        # 'i18n_app',
    ]

    # ...

    LANGUAGE_CODE = 'ja' # 日本語に設定

    USE_I18N = True # 国際化を有効にする
    ```

3.  **フォームの定義**
    `i18n_app` ディレクトリ内に `forms.py` という新しいファイルを作成し、以下の内容を記述してください。`gettext_lazy` を使ってエラーメッセージをマークします。

    ```python
    # i18n_app/forms.py

    from django import forms
    from django.utils.translation import gettext_lazy as _

    class MyForm(forms.Form):
        name = forms.CharField(
            label=_('名前'),
            max_length=5,
            error_messages={
                'required': _('名前は必須です。'),
                'max_length': _('名前は%(limit_value)d文字以内で入力してください。'),
            }
        )
    ```

4.  **ビューの作成**
    `i18n_app/views.py` を開き、フォームを処理するビュー関数 `my_form_view` を作成してください。

    ```python
    # i18n_app/views.py

    from django.shortcuts import render
    from .forms import MyForm

    def my_form_view(request):
        if request.method == 'POST':
            form = MyForm(request.POST)
            if form.is_valid():
                message = "入力成功！"
            else:
                message = "入力失敗！エラーを確認してください。"
        else:
            form = MyForm()
            message = "フォームを入力してください。"
        
        return render(request, 'i18n_app/my_form.html', {'form': form, 'message': message})
    ```

5.  **テンプレートファイルの作成**
    `i18n_app/templates/i18n_app/` ディレクトリを作成し、その中に `my_form.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>国際化フォーム</title>
    </head>
    <body>
        <h1>国際化フォーム</h1>
        <p>{{ message }}</p>
        <form action="" method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">送信</button>
        </form>
    </body>
    </html>
    ```

6.  **URL設定の追加**
    `i18n_app/urls.py` を作成し、`my_form_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # i18n_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('form/', views.my_form_view, name='my_form'),
    ]
    ```

    ```python
    # validation_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('i18n_app/', include('i18n_app.urls')),
    ]
    ```

7.  **翻訳ファイルの作成とコンパイル**
    以下のコマンドを実行して、翻訳ファイルを作成し、コンパイルしてください。

    ```bash
    docker compose exec web python manage.py makemessages -l ja
    # 生成されたi18n_app/locale/ja/LC_MESSAGES/django.po を開き、
    # msgid "名前は必須です。" の下の msgstr "" に日本語訳を記入
    # msgid "名前は%(limit_value)d文字以内で入力してください。" の下の msgstr "" に日本語訳を記入
    # 例: msgstr "名前は必須です。"
    # 例: msgstr "名前は%(limit_value)d文字以内で入力してください。"

    docker compose exec web python manage.py compilemessages
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/i18n_app/form/` にアクセスしてください。

*   名前を空にして送信し、「名前は必須です。」と表示されることを確認してください。
*   名前を6文字以上にして送信し、「名前は5文字以内で入力してください。」と表示されることを確認してください。

## 課題2: フォームセットを使った複数のフォームのバリデーション

複数の商品アイテムを一度に入力し、それぞれをバリデーションするフォームセットを作成しましょう。

### 手順

1.  **モデルの定義**
    `i18n_app/models.py` を開き、`Item` という名前のモデルを作成してください。このモデルには、`name` (CharField) と `quantity` (IntegerField) のフィールドを持たせてください。

    ```python
    # i18n_app/models.py

    from django.db import models

    class Item(models.Model):
        name = models.CharField(max_length=100)
        quantity = models.IntegerField()

        def __str__(self):
            return self.name
    ```

2.  **マイグレーションの作成と適用**
    モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成し、データベースに適用してください。

    ```bash
    docker compose exec web python manage.py makemigrations i18n_app
    docker compose exec web python manage.py migrate
    ```

3.  **フォームセットの定義**
    `i18n_app/forms.py` を開き、`Item` モデルに対応する `ItemForm` と、それを使った `ItemFormSet` を定義してください。

    ```python
    # i18n_app/forms.py

    from django import forms
    from django.forms import formset_factory
    from .models import Item

    class ItemForm(forms.ModelForm):
        class Meta:
            model = Item
            fields = ['name', 'quantity']

    ItemFormSet = formset_factory(ItemForm, extra=2) # 初期表示で2つの空のフォームを表示
    ```

4.  **ビューの作成**
    `i18n_app/views.py` を開き、`manage_items_view` という名前の関数を作成してください。この関数は、フォームセットを処理し、データベースへの保存を行います。

    ```python
    # i18n_app/views.py

    from django.shortcuts import render, redirect
    from .forms import ItemFormSet
    from .models import Item

    def manage_items_view(request):
        if request.method == 'POST':
            formset = ItemFormSet(request.POST)
            if formset.is_valid():
                for form in formset:
                    if form.cleaned_data:
                        form.save()
                return redirect('item_list')
        else:
            formset = ItemFormSet()
        
        items = Item.objects.all()
        return render(request, 'i18n_app/manage_items.html', {'formset': formset, 'items': items})
    ```

5.  **テンプレートファイルの作成**
    `i18n_app/templates/i18n_app/` ディレクトリ内に `manage_items.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>アイテム管理</title>
    </head>
    <body>
        <h1>アイテム管理</h1>
        <form method="post">
            {% csrf_token %}
            {{ formset.management_form }}
            {% for form in formset %}
                {{ form.as_p }}
            {% endfor %}
            <button type="submit">保存</button>
        </form>

        <h2>登録済みアイテム</h2>
        <ul>
            {% for item in items %}
                <li>{{ item.name }} ({{ item.quantity }})</li>
            {% empty %}
                <li>アイテムがありません。</li>
            {% endfor %}
        </ul>
    </body>
    </html>
    ```

6.  **URL設定の追加**
    `i18n_app/urls.py` に、`manage_items_view` へのURLを追加してください。

    ```python
    # i18n_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('form/', views.my_form_view, name='my_form'),
        path('items/', views.manage_items_view, name='item_list'),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/i18n_app/items/` にアクセスしてください。複数のフォームが表示され、入力して保存できることを確認してください。バリデーションエラーも正しく表示されるはずです。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
