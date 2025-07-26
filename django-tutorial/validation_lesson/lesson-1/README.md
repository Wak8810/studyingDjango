# Django学習レッスン: バリデーション - Lesson 1

## 目的

このレッスンでは、Djangoにおける入力データのバリデーション（検証）について学びます。特に、Djangoのフォーム機能を使って、ユーザーからの入力を安全かつ効率的に検証する方法を習得します。

## バリデーションの概要

バリデーションとは、ユーザーからの入力データが、アプリケーションが期待する形式や条件を満たしているかを確認するプロセスです。例えば、メールアドレスの形式が正しいか、パスワードが一定の長さを満たしているか、数値が範囲内にあるかなどをチェックします。

Djangoでは、主に**フォーム (Forms)** を使ってバリデーションを行います。フォームは、HTMLのフォームを生成するだけでなく、入力データのバリデーションとクリーンアップ（安全なデータに変換すること）も担当します。

## 環境構築

このレッスンは、`validation_lesson/lesson-1` ディレクトリ内で独立したDjangoプロジェクトとして動作します。

1.  `validation_lesson/lesson-1` ディレクトリに移動します。
    ```bash
    cd validation_lesson/lesson-1
    ```
2.  Dockerコンテナを起動します。
    ```bash
    docker compose up -d
    ```
    これにより、Django開発サーバーが起動します。

## 課題1: シンプルなフォームとバリデーション

ユーザーの名前と年齢を入力するフォームを作成し、名前の長さを制限するバリデーションを実装しましょう。

### 手順

1.  **Djangoアプリケーションの作成**
    `validation_project` プロジェクト内に、`my_form_app` という名前の新しいDjangoアプリケーションを作成してください。
    ```bash
    docker compose exec web python manage.py startapp my_form_app
    ```

2.  **`settings.py` の設定**
    `validation_project/settings.py` を開き、`INSTALLED_APPS` リストに `'my_form_app'` を追加して、Djangoに新しいアプリケーションを認識させてください。

    ```python
    # validation_project/settings.py

    INSTALLED_APPS = [
        # ... 既存のアプリ ...
        # ここに 'my_form_app' を追加
        # 'my_form_app',
    ]
    ```

3.  **フォームの定義**
    `my_form_app` ディレクトリ内に `forms.py` という新しいファイルを作成し、以下の内容を記述してください。`name` フィールドの `max_length` を `5` に設定し、5文字を超える名前を入力するとエラーになるようにします。

    ```python
    # my_form_app/forms.py

    from django import forms

    class MySimpleForm(forms.Form):
        # nameフィールド: 最大5文字
        # name = forms.CharField(label='お名前', max_length=5)
        # ageフィールド: 0から150までの整数
        # age = forms.IntegerField(label='年齢', min_value=0, max_value=150)
        pass
    ```

4.  **ビューの作成**
    `my_form_app/views.py` を開き、フォームを処理するビュー関数 `my_form_view` を作成してください。GETリクエストでは空のフォームを表示し、POSTリクエストではバリデーションを実行し、結果に応じてメッセージを表示するようにします。

    ```python
    # my_form_app/views.py

    from django.shortcuts import render
    from .forms import MySimpleForm # MySimpleFormをインポート

    def my_form_view(request):
        if request.method == 'POST':
            form = MySimpleForm(request.POST)
            if form.is_valid():
                name = form.cleaned_data['name']
                age = form.cleaned_data['age']
                message = f"バリデーション成功！ 名前: {name}, 年齢: {age}"
            else:
                message = "バリデーション失敗！エラーを確認してください。"
        else:
            form = MySimpleForm()
            message = "フォームを入力してください。"
        
        return render(request, 'my_form_app/form_template.html', {'form': form, 'message': message})
    ```

5.  **テンプレートファイルの作成**
    `my_form_app/templates/my_form_app/` ディレクトリを作成し、その中に `form_template.html` という名前で以下の内容を記述してください。

    ```html
    <!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>バリデーションフォーム</title>
    </head>
    <body>
        <h1>入力フォーム</h1>
        <p>{{ message }}</p>
        <form action="" method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <input type="submit" value="送信">
        </form>
    </body>
    </html>
    ```

6.  **URL設定の追加**
    `my_form_app/urls.py` を作成し、`my_form_view` へのURLを設定してください。プロジェクトの `urls.py` からもインクルードするのを忘れないでください。

    ```python
    # my_form_app/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
        path('input/', views.my_form_view, name='my_form_input'),
    ]
    ```

    ```python
    # validation_project/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('my_form_app/', include('my_form_app.urls')),
    ]
    ```

### 確認

すべての変更を保存したら、ブラウザで `http://localhost:8000/my_form_app/input/` にアクセスしてください。

*   「お名前」に5文字を超える名前を入力して送信し、エラーメッセージが表示されることを確認してください。
*   「お名前」に5文字以内の名前を入力して送信し、成功メッセージが表示されることを確認してください。

## 困った時は

*   Dockerコンテナが起動しているか確認してください: `docker compose ps`
*   エラーメッセージをよく読んでください。Djangoのエラーページは非常に詳細です。
*   `docker compose logs web` でコンテナのログを確認してください。
