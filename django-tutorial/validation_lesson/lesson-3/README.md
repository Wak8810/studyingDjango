# Django学習レッスン: バリデーション - Lesson 3

## 目的

このレッスンでは、Djangoのフォームにおけるエラーメッセージのカスタマイズと、バリデーションエラーをテンプレートでより分かりやすく表示する方法を学びます。これにより、ユーザーフレンドリーな入力フォームを作成できるようになります。

## 概要

Djangoのフォームは、バリデーションエラーが発生した場合に自動的にエラーメッセージを生成します。しかし、これらのデフォルトメッセージは必ずしもユーザーにとって分かりやすいとは限りません。エラーメッセージをカスタマイズすることで、より具体的な指示やヒントを提供できます。

*   **エラーメッセージのカスタマイズ**: フィールド定義時に `error_messages` 引数を使用したり、`clean_field()` メソッド内で `ValidationError` を発生させる際にメッセージを指定したりします。
*   **テンプレートでのエラー表示**: `{{ form.as_p }}` だけでなく、`{{ field.errors }}` や `{{ form.non_field_errors }}` を使って、個々のフィールドのエラーやフォーム全体のエラーを細かく表示できます。

## 環境構築

このレッスンは、`validation_lesson/lesson-3` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `validation_lesson/lesson-3` ディレクトリに移動します。
    ```bash
    cd validation_lesson/lesson-3
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: エラーメッセージのカスタマイズ

ユーザー名とパスワードのフォームを作成し、それぞれのフィールドのバリデーションエラーメッセージをカスタマイズしましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `validation_project` プロジェクト内に、`custom_error_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp custom_error_app
    ```

2.  **`settings.py` の設定**
    `validation_project/settings.py` を開き、`INSTALLED_APPS` リストに `'custom_error_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # validation_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'custom_error_app' を追加
        # 'custom_error_app',
    ]
    ```

3.  **フォームの定義**
    `custom_error_app` ディレクトリ内に `forms.py` という新しいファイルを作成し、以下の内容を記述してください。`username` と `password` フィールドの `error_messages` をカスタマイズします。

    ```python
    # custom_error_app/forms.py

    from django import forms

    class LoginForm(forms.Form):
        username = forms.CharField(
            label='ユーザー名',
            max_length=10,
            error_messages={
                'required': 'ユーザー名は必須です。',
                'max_length': 'ユーザー名は10文字以内で入力してください。',
            }
        )
        password = forms.CharField(
            label='パスワード',
            min_length=8,
            widget=forms.PasswordInput,
            error_messages={
                'required': 'パスワードは必須です。',
                'min_length': 'パスワードは8文字以上で入力してください。',
            }
        )
    ```

4.  **ビューの作成**
    `custom_error_app/views.py` を開き、フォームを処理するビュー関数 `login_view` を作成してください。

    ```python
    # custom_error_app/views.py

    from django.shortcuts import render
    from .forms import LoginForm

    def login_view(request):
        if request.method == 'POST':
            form = LoginForm(request.POST)
            if form.is_valid():
                message = "ログイン成功！"
            else:
                message = "ログイン失敗！エラーを確認してください。"
        else:
            form = LoginForm()
            message = "ログインフォーム"
        
        return render(request, 'custom_error_app/login_form.html', {'form': form, 'message': message})
    ```

5.  **テンプレートファイルの作成**
    `custom_error_app/templates/custom_error_app/` ディレクトリを作成し、その中に `login_form.html` という名前で以下の内容を記述してください。`{{ form.as_p }}` を使ってエラーメッセージが表示されることを確認します。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>ログイン</title>
    </head>
    <body>
        <h1>ログイン</h1>
        <p>{{ message }}</p>
        <form action="" method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">ログイン</button>
        </form>
    </body>
    </html>
    ```

6.  **URL設定の追加**
    `custom_error_app/urls.py` を作成し、`login_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # custom_error_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('login/', views.login_view, name='login'),
    ]
    ```

    ```python
    # validation_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('custom_error_app/', include('custom_error_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/custom_error_app/login/` にアクセスしてください。

*   ユーザー名を空にして送信し、「ユーザー名は必須です。」と表示されることを確認してください。
*   ユーザー名を10文字以上にして送信し、「ユーザー名は10文字以内で入力してください。」と表示されることを確認してください。
*   パスワードを7文字以下にして送信し、「パスワードは8文字以上で入力してください。」と表示されることを確認してください。

## 課題2: テンプレートでの詳細なエラー表示

個々のフィールドのエラーメッセージを、より細かく制御して表示するようにテンプレートを修正しましょう。

### 手順

1.  **テンプレートファイルの修正**
    `custom_error_app/templates/custom_error_app/login_form.html` を開き、`{{ form.as_p }}` の代わりに、各フィールドとエラーメッセージを個別に表示するように修正してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>ログイン</title>
    </head>
    <body>
        <h1>ログイン</h1>
        <p>{{ message }}</p>
        <form action="" method="post">
            {% csrf_token %}
            
            {% comment %} フォーム全体のエラーを表示 {% endcomment %}
            {# 例: {% if form.non_field_errors %}
                <div class="errorlist">
                    {% for error in form.non_field_errors %}
                        <p>{{ error }}</p>
                    {% endfor %}
                </div>
            {% endif %} #}

            <p>
                {{ form.username.label_tag }}
                {{ form.username }}
                {% comment %} ユーザー名フィールドのエラーを表示 {% endcomment %}
                {# 例: {% if form.username.errors %}
                    <ul class="errorlist">
                        {% for error in form.username.errors %}
                            <li>{{ error }}</li>
                        {% endfor %}
                    </ul>
                {% endif %} #}
            </p>
            <p>
                {{ form.password.label_tag }}
                {{ form.password }}
                {% comment %} パスワードフィールドのエラーを表示 {% endcomment %}
                {# 例: {% if form.password.errors %}
                    <ul class="errorlist">
                        {% for error in form.password.errors %}
                            <li>{{ error }}</li>
                        {% endfor %}
                    </ul>
                {% endif %} #}
            </p>
            <button type="submit">ログイン</button>
        </form>
    </body>
    </html>
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/custom_error_app/login/` に再度アクセスしてください。エラーメッセージが各フィールドの下に表示されることを確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
