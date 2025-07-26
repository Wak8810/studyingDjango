# Django学習レッスン: バリデーション - Lesson 4

## 目的

このレッスンでは、Djangoのフォームにおけるカスタムバリデーターの作成と、モデルフォームでのバリデーションについて学びます。これにより、再利用可能なバリデーションロジックを定義し、モデルと連携したフォームのバリデーションを効率的に行えるようになります。

## 概要

*   **カスタムバリデーター**: `django.core.validators` モジュールを使って、独自のバリデーター関数やクラスを作成し、フォームフィールドに適用できます。これにより、特定のビジネスルールに基づいた複雑なバリデーションを再利用可能な形で実装できます。
*   **モデルフォームのバリデーション**: `ModelForm` は、対応するモデルのフィールド定義に基づいて自動的にバリデーションルールを生成します。さらに、`ModelForm` の `clean()` メソッドや `clean_field()` メソッドをオーバーライドすることで、カスタムバリデーションを追加できます。

## 環境構築

このレッスンは、`validation_lesson/lesson-4` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `validation_lesson/lesson-4` ディレクトリに移動します。
    ```bash
    cd validation_lesson/lesson-4
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: カスタムバリデーターの作成と適用

ユーザーが入力する数値が偶数であるかをチェックするカスタムバリデーターを作成し、フォームフィールドに適用しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `validation_project` プロジェクト内に、`custom_validator_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp custom_validator_app
    ```

2.  **`settings.py` の設定**
    `validation_project/settings.py` を開き、`INSTALLED_APPS` リストに `'custom_validator_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # validation_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'custom_validator_app' を追加
        # 'custom_validator_app',
    ]
    ```

3.  **カスタムバリデーターの作成**
    `custom_validator_app` ディレクトリ内に `validators.py` という新しいファイルを作成し、以下の内容を記述してください。`validate_even` 関数は、入力が偶数でない場合に `ValidationError` を発生させます。

    ```python
    # custom_validator_app/validators.py

    from django.core.exceptions import ValidationError
    from django.utils.translation import gettext_lazy as _

    def validate_even(value):
        # ここに偶数チェックのバリデーションロジックを記述
        # if value % 2 != 0:
        #     raise ValidationError(
        #         _('%(value)s は偶数ではありません'),
        #         params={'value': value},
        #     )
        pass
    ```

4.  **フォームの定義**
    `custom_validator_app` ディレクトリ内に `forms.py` という新しいファイルを作成し、以下の内容を記述してください。`number` フィールドに `validate_even` カスタムバリデーターを適用します。

    ```python
    # custom_validator_app/forms.py

    from django import forms
    from .validators import validate_even # カスタムバリデーターをインポート

    class NumberForm(forms.Form):
        number = forms.IntegerField(
            label='偶数を入力してください',
            validators=[validate_even] # ここにカスタムバリデーターを適用
        )
    ```

5.  **ビューの作成**
    `custom_validator_app/views.py` を開き、フォームを処理するビュー関数 `number_input_view` を作成してください。

    ```python
    # custom_validator_app/views.py

    from django.shortcuts import render
    from .forms import NumberForm

    def number_input_view(request):
        if request.method == 'POST':
            form = NumberForm(request.POST)
            if form.is_valid():
                message = f"入力された偶数: {form.cleaned_data['number']}"
            else:
                message = "入力エラー！偶数を入力してください。"
        else:
            form = NumberForm()
            message = "偶数入力フォーム"
        
        return render(request, 'custom_validator_app/number_form.html', {'form': form, 'message': message})
    ```

6.  **テンプレートファイルの作成**
    `custom_validator_app/templates/custom_validator_app/` ディレクトリを作成し、その中に `number_form.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>偶数入力</title>
    </head>
    <body>
        <h1>偶数入力フォーム</h1>
        <p>{{ message }}</p>
        <form action="" method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">送信</button>
        </form>
    </body>
    </html>
    ```

7.  **URL設定の追加**
    `custom_validator_app/urls.py` を作成し、`number_input_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # custom_validator_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('number/', views.number_input_view, name='number_input'),
    ]
    ```

    ```python
    # validation_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('custom_validator_app/', include('custom_validator_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/custom_validator_app/number/` にアクセスしてください。

*   奇数を入力して送信し、エラーメッセージが表示されることを確認してください。
*   偶数を入力して送信し、成功メッセージが表示されることを確認してください。

## 課題2: モデルフォームでのバリデーション

モデルフォームを使って、モデルのフィールドにカスタムバリデーターを適用し、フォーム経由でバリデーションが機能することを確認しましょう。

### 手順

1.  **モデルの定義**
    `custom_validator_app/models.py` を開き、`MyModel` という名前のモデルを作成してください。このモデルには、`even_number` という `IntegerField` を持たせ、これに `validate_even` カスタムバリデーターを適用します。

    ```python
    # custom_validator_app/models.py

    from django.db import models
    from .validators import validate_even # カスタムバリデーターをインポート

    class MyModel(models.Model):
        even_number = models.IntegerField(validators=[validate_even])

        def __str__(self):
            return str(self.even_number)
    ```

2.  **マイグレーションの作成と適用**
    モデルを定義したら、以下のコマンドを実行してマイグレーションファイルを作成し、データベースに適用してください。

    ```bash
    docker compose exec web python manage.py makemigrations custom_validator_app
    docker compose exec web python manage.py migrate
    ```

3.  **モデルフォームの定義**
    `custom_validator_app/forms.py` を開き、`MyModel` に対応する `MyModelForm` を定義してください。

    ```python
    # custom_validator_app/forms.py

    from django import forms
    from .models import MyModel # MyModelをインポート
    from .validators import validate_even

    class MyModelForm(forms.ModelForm):
        class Meta:
            model = MyModel
            fields = ['even_number']
    ```

4.  **ビューの作成**
    `custom_validator_app/views.py` を開き、`mymodel_form_view` という名前の関数を作成してください。この関数は、`MyModelForm` を処理し、データベースへの保存を行います。

    ```python
    # custom_validator_app/views.py

    from django.shortcuts import render
    from .forms import MyModelForm
    from .models import MyModel

    def mymodel_form_view(request):
        if request.method == 'POST':
            form = MyModelForm(request.POST)
            if form.is_valid():
                form.save()
                message = "データが正常に保存されました。"
            else:
                message = "保存エラー！偶数を入力してください。"
        else:
            form = MyModelForm()
            message = "モデルフォーム入力フォーム"
        
        # 既存のデータを表示
        all_data = MyModel.objects.all()
        return render(request, 'custom_validator_app/mymodel_form.html', {'form': form, 'message': message, 'all_data': all_data})
    ```

5.  **テンプレートファイルの作成**
    `custom_validator_app/templates/custom_validator_app/` ディレクトリ内に `mymodel_form.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>モデルフォームバリデーション</title>
    </head>
    <body>
        <h1>モデルフォームバリデーション</h1>
        <p>{{ message }}</p>
        <form action="" method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">保存</button>
        </form>

        <h2>保存済みデータ</h2>
        <ul>
            {% for data in all_data %}
                <li>{{ data.even_number }}</li>
            {% empty %}
                <li>データがありません。</li>
            {% endfor %}
        </ul>
    </body>
    </html>
    ```

6.  **URL設定の追加**
    `custom_validator_app/urls.py` に、`mymodel_form_view` へのURLを追加してください。

    ```python
    # custom_validator_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('number/', views.number_input_view, name='number_input'),
        path('mymodel/', views.mymodel_form_view, name='mymodel_form'),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/custom_validator_app/mymodel/` にアクセスしてください。

*   奇数を入力して送信し、エラーメッセージが表示され、データが保存されないことを確認してください。
*   偶数を入力して送信し、データが保存され、リストに表示されることを確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
