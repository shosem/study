# docker compose

複数のコンテナをまとめて管理。`compose.yml`に各コンテナの設定を書く。

## compose.ymlの基本構文

### ビルドコンテキスト（`build:` ）

指定したディレクトリ内のファイルがすべてビルド対象に。
Dockerfileの検索や、COPY / ADDのパスもここが基準に

```docker
services:                        # サービス（コンテナ）群の定義開始
  web:                           # web サービスの設定
   ** build:** ./app ****                # app ディレクトリをビルドコンテキストに指定
```

### バインドマウント（`volumes:` ）

`ホスト側パス：コンテナ側パス`でファイル・ディレクトリを共有。
ホスト：compose.ymlの場所を起点とする
名前付きvol.はDocker管理下のストレージ

```docker
services:                        
  web:                        
    volumes:                      # ホスト⇔コンテナ間のファイル共有設定
      - ./app:/usr/src/app        # プロジェクトディレクトリをコンテナにマウント
      - logs:/var/log/myapp       # 名前付きボリューム logs をマウント
      　　　　　　　　　　　　　　　　　# あとで作るlogsを使っている（Docker管理下）
```

### 環境変数ファイル（`env_file:` ）

composeファイルと同じディレクトリに置いた`.env`や任意のファイルを指定すると読み込む

```docker
services:                        
  web:                       
   ** build:** ./app   ****             
    env_file:                     # 環境変数を外部ファイルから読み込み
      - .env                      # 同ディレクトリの .env ファイルを使用
 
```

### 全体例

```docker
services:                        # サービス（コンテナ）群の定義開始
  web:                           # web サービスの設定
    build: ./app                 # app ディレクトリをビルドコンテキストに指定
    env_file:                     # 環境変数を外部ファイルから読み込み
      - .env                      # 同ディレクトリの .env ファイルを使用
    ports:                        # ホスト⇔コンテナのポートマッピング
      - "3000:3000"               # ホスト3000番をコンテナ3000番に割り当て
    volumes:                      # ホスト⇔コンテナ間のファイル共有設定
      - ./app:/usr/src/app        # プロジェクトディレクトリをコンテナにマウント
      - logs:/var/log/myapp       # 名前付きボリューム logs をマウント

  db:                            # db サービスの設定
    image: postgres:13           # 使用する PostgreSQL イメージとタグ
    environment:                 # コンテナ内の環境変数設定
      POSTGRES_DB: myapp_development   # データベース名
      POSTGRES_USER: user              # データベースユーザー名
      POSTGRES_PASSWORD: password      # データベースパスワード
    ports:                        # ポートマッピング
      - "5432:5432"               # ホスト5432番をコンテナ5432番に割り当て

volumes:                         # 名前付きボリューム定義ブロック
  logs:                          # 名前付きボリューム logs を宣言
```

### **サービス内の主なキー**

| キー | 説明 |
| --- | --- |
| `build` | ローカルの `Dockerfile` からイメージをビルド |
| `image` | 公開イメージ（例：`nginx:latest`）や事前に作ったイメージを利用 |
| `ports` | `ホスト:コンテナ` 形式でポートをマッピング |
| `env_file` | `.env` ファイルを読み込み、環境変数を一括で設定 |
| `environment` | 環境変数を個別に直接設定（ファイルより優先される） |
| `volumes` | ホスト ↔ コンテナ間でファイル・ディレクトリを共有／永続化 |
| `depends_on` | 起動順制御（ただし「正常起動」は保証しない） |

## imageとbuildの違い

image：既存のイメージを使う。postgres公式が用意したimageを利用

build：自分のDockerfileからimageをビルド

```docker
# 既存イメージを参照
db:
  image: postgres:13

# ローカルソースをビルド
web:
  build: ./app             # appにあるDockerfileからビルド
  image: my-rails-app:dev  # そのイメージの名前をこの名前に保存
```

## データベースの永続化

```docker
services:
  db:
    image: postgres:13
    volumes:
      - db_data:/var/lib/postgresql/data # 「/postgresql/dataはdb_dataにつながるよう手配して！」

volumes:
  db_data:                               # 「db_dataというボリュームを用意して！」
```

> マウントとは：USBメモリのような。db_dataに、/postgres/dataでアクセスできる。
postgresは/postgres/dataに保存しようとするが、そこはdb_dataにつながっている。
つまり、データはdb_data（Docker管理の記憶場所）に保存される。
> 

## 環境変数ファイルと直接指定

`.env`ファイルを`env_file:`で読み込む
`environment:` で個別に`キー：値`を書いて上書きや追加可能

```docker
# .env
POSTGRES_PASSWORD=secret
RAILS_ENV=development
```

```docker
services:
  web:
    build: ./app
    env_file:
      - .env
    environment:
      - SECRET_KEY_BASE=${SECRET_KEY_BASE}
```

### **`command:` / `entrypoint:` で起動コマンドをカスタマイズ**

- デフォルト（Dockerfile）の `CMD` / `ENTRYPOINT` を上書き
- マイグレーション実行やシード投入など、起動時に追加処理を組み込める

```yaml
services:
  web:
    build: ./app
    command: >
      sh -c "bundle exec rails db:migrate &&
             bundle exec rails server -b 0.0.0.0"
```