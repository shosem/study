# Dockerコマンドラインツール

docker compose upとかのソレ。

### docker build

役割：Dockerfileからイメージを作成する。

```docker
基本形：
docker build -t <イメージ名>:<タグ> <Dockerfileのあるディレクトリ>

実例：
docker build -t railspractice:rails . 
```

### docker run

役割：イメージからコンテナを起動

```docker
基本形：
docker run [オプション] <イメージ名>:<タグ名>

実例：
docker run --rm railspractice:rails
```

オプション

- -d：バックグラウンド実行
- —rm：停止後に自動削除
- -p：ホストポート：コンテナポート