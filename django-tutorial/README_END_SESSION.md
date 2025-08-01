# 学習セッションを終了する

このファイルは、Django学習セッションを終了する際の手順を説明します。

## 1. 現在のディレクトリからDockerコンテナを停止する

現在作業しているレッスンのディレクトリ内で、以下のコマンドを実行してDockerコンテナを停止します。

```bash
docker compose down
```

これにより、そのレッスン用のDjango開発サーバーが停止し、関連するリソースが解放されます。

## 2. 親ディレクトリに戻る (オプション)

必要であれば、親ディレクトリに戻ります。

```bash
cd ..
```

## 3. 次の学習セッションのために

次回学習を再開する際は、`README_START_SESSION.md` を参照してください。