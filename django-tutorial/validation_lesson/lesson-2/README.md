# Django学習レッスン: バリデーション - Lesson 2

## 目的

このレッスンでは、Djangoのフォームにおけるカスタムバリデーションと、複数のフィールドにまたがるバリデーション（クリーンメソッド）について学びます。これにより、より複雑なビジネスロジックに基づいた入力検証を実装できるようになります。

## 概要

Djangoのフォームは、各フィールドに組み込みのバリデーションルールを提供しますが、それだけでは不十分な場合があります。カスタムバリデーションを使うことで、独自のルールを定義できます。

*   **カスタムバリデーション**: 特定のフィールドに対して、独自の検証ロジックを追加します。例えば、特定の文字列が含まれているか、特定のパターンにマッチするかなどをチェックできます。
*   **`clean()` メソッド**: フォーム全体、または複数のフィールドにまたがるバリデーションを行う際に使用します。例えば、パスワードとその確認用パスワードが一致するかどうかをチェックする際などに便利です。

## 環境構築

このレッスンは、`validation_lesson/lesson-2` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `validation_lesson/lesson-2` ディレクトリに移動します。
    ```bash
    cd validation_lesson/lesson-2
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: カスタムバリデーションの追加

ユーザーが入力するメールアドレスが特定のドメイン（例: `@example.com`）であるかをチェックするカスタムバリデーションを追加しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `validation_project` プロジェクト内に、`custom_form_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp custom_form_app
    ```

2.  **`settings.py` の設定**
    `validation_project/settings.py` を開き、`INSTALLED_APPS` リストに `'custom_form_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # validation_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'custom_form_app' を追加
        # 'custom_form_app',
    ]
    ```

3.  **フォームの定義**
    `custom_form_app` ディレクトリ内に `forms.py` という新しいファイルを作成し、以下の内容を記述してください。`email` フィールドにカスタムバリデーションを追加します。

    ```python
    # custom_form_app/forms.py

    from django import forms
    from django.core.exceptions import ValidationError

    class RegistrationForm(forms.Form):
        username = forms.CharField(label='ユーザー名', max_length=100)
        email = forms.EmailField(label='メールアドレス')

        def clean_email(self):
            email = self.cleaned_data['email']
            # ここにカスタムバリデーションロジックを記述
            # if not email.endswith('@example.com'):
            #     raise ValidationError("メールアドレスは@example.comドメインである必要があります。")
            return email
    ```

4.  **ビューの作成**
    `custom_form_app/views.py` を開き、フォームを処理するビュー関数 `register_view` を作成してください。

    ```python
    # custom_form_app/views.py

    from django.shortcuts import render
    from .forms import RegistrationForm

    def register_view(request):
        if request.method == 'POST':
            form = RegistrationForm(request.POST)
            if form.is_valid():
                message = "登録成功！"
            else:
                message = "登録失敗！エラーを確認してください。"
        else:
            form = RegistrationForm()
            message = "ユーザー登録フォーム"
        
        return render(request, 'custom_form_app/register_form.html', {'form': form, 'message': message})
    ```

5.  **テンプレートファイルの作成**
    `custom_form_app/templates/custom_form_app/` ディレクトリを作成し、その中に `register_form.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>ユーザー登録</title>
    </head>
    <body>
        <h1>ユーザー登録</h1>
        <p>{{ message }}</p>
        <form action="" method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">登録</button>
        </form>
    </body>
    </html>
    ```

6.  **URL設定の追加**
    `custom_form_app/urls.py` を作成し、`register_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # custom_form_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('register/', views.register_view, name='register'),
    ]
    ```

    ```python
    # validation_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('custom_form_app/', include('custom_form_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/custom_form_app/register/` にアクセスしてください。

*   メールアドレスに `@example.com` 以外のドメインを入力して送信し、エラーメッセージが表示されることを確認してください。
*   メールアドレスに `@example.com` ドメインを入力して送信し、成功メッセージが表示されることを確認してください。

## 課題2: `clean()` メソッドを使った複数フィールドのバリデーション

パスワードとその確認用パスワードが一致するかどうかをチェックするフォームを作成しましょう。

### 手順

1.  **フォームの定義の修正**
    `custom_form_app/forms.py` を開き、`PasswordForm` という新しいフォームクラスを作成してください。このフォームには `password` と `password_confirm` フィールドを持たせ、`clean()` メソッドで両者が一致するかを検証します。

    ```python
    # custom_form_app/forms.py

    from django import forms
    from django.core.exceptions import ValidationError

    class PasswordForm(forms.Form):
        password = forms.CharField(label='パスワード', widget=forms.PasswordInput)
        password_confirm = forms.CharField(label='パスワード（確認）', widget=forms.PasswordInput)

        def clean(self):
            cleaned_data = super().clean()
            password = cleaned_data.get('password')
            password_confirm = cleaned_data.get('password_confirm')

            # ここにパスワード一致のバリデーションロジックを記述
            # if password and password_confirm and password != password_confirm:
            #     raise ValidationError("パスワードが一致しません。")

            return cleaned_data
    ```

2.  **ビューの作成**
    `custom_form_app/views.py` を開き、`password_change_view` という名前の関数を作成してください。この関数は、`PasswordForm` を処理します。

    ```python
    # custom_form_app/views.py

    from django.shortcuts import render
    from .forms import PasswordForm # PasswordFormをインポート

    def password_change_view(request):
        if request.method == 'POST':
            form = PasswordForm(request.POST)
            if form.is_valid():
                message = "パスワードが正常に設定されました。"
            else:
                message = "パスワード設定失敗！エラーを確認してください。"
        else:
            form = PasswordForm()
            message = "パスワード設定フォーム"
        
        return render(request, 'custom_form_app/password_form.html', {'form': form, 'message': message})
    ```

3.  **テンプレートファイルの作成**
    `custom_form_app/templates/custom_form_app/` ディレクトリ内に `password_form.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>パスワード設定</title>
    </head>
    <body>
        <h1>パスワード設定</h1>
        <p>{{ message }}</p>
        <form action="" method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">設定</button>
        </form>
    </body>
    </html>
    ```

4.  **URL設定の追加**
    `custom_form_app/urls.py` に、`password_change_view` へのURLを追加してください。

    ```python
    # custom_form_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('register/', views.register_view, name='register'),
        path('password/', views.password_change_view, name='password_change'),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/custom_form_app/password/` にアクセスしてください。

*   パスワードと確認用パスワードが一致しない状態で送信し、エラーメッセージが表示されることを確認してください。
*   パスワードと確認用パスワードが一致する状態で送信し、成功メッセージが表示されることを確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
