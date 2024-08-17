# README

This README would normally document whatever steps are necessary to get the
application up and running.

Things you may want to cover:

- Ruby version

- System dependencies

- Configuration

- Database creation

- Database initialization

- How to run the test suite

- Services (job queues, cache servers, search engines, etc.)

- Deployment instructions

- ...

### Docker 環境の構築手順

以下の手順で、Rails 7、Ruby 3、PostgreSQL、pgAdmin 4、Nginx を含む開発環境を Docker 上に構築できます。

#### 1. ディレクトリ構成

まず、プロジェクトディレクトリを作成し、以下のような構成にします。

```
myapp/
├── Dockerfile
├── docker-compose.yml
├── nginx.conf
└── src/
    └── myapp/
```

#### 2. Dockerfile の作成

`myapp/Dockerfile`を以下の内容で作成します。

```dockerfile
FROM ruby:3.0

RUN apt-get update -qq && apt-get install -y nodejs postgresql-client

WORKDIR /myapp
COPY src/myapp/Gemfile /myapp/Gemfile
COPY src/myapp/Gemfile.lock /myapp/Gemfile.lock
RUN bundle install

COPY src/myapp /myapp
```

#### 3. docker-compose.yml の作成

`myapp/docker-compose.yml`を以下の内容で作成します。

```yaml
version: "3"
services:
  db:
    image: postgres:latest
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp_development

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"

  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bin/rails server -b 0.0.0.0"
    volumes:
      - ./src/myapp:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db

  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - web

volumes:
  postgres_data:
```

#### 4. nginx.conf の作成

`myapp/nginx.conf`を以下の内容で作成します。

```nginx
events {}

http {
    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://web:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

#### 5. データベースの作成

データベースが見つからない場合、以下のコマンドでデータベースを作成します。

```sh
docker-compose run web bin/rails db:create
```

### 環境完成前のディレクトリ構成図

```
myapp/
├── Dockerfile
├── docker-compose.yml
├── nginx.conf
└── src/
    └── myapp/
        ├── Gemfile
        └── Gemfile.lock
```

### 環境完成後のディレクトリ構成図

```
myapp/
├── Dockerfile
├── docker-compose.yml
├── nginx.conf
├── src/
│   └── myapp/
│       ├── Gemfile
│       ├── Gemfile.lock
│       ├── app/
│       ├── bin/
│       ├── config/
│       │   └── database.yml
│       ├── db/
│       ├── lib/
│       ├── log/
│       ├── public/
│       ├── storage/
│       ├── test/
│       ├── tmp/
│       └── vendor/
└── volumes/
    └── postgres_data/
```

これで、Docker 上に最新の安定版を使用した開発環境が構築できます。何か他に質問があれば教えてください！

Source: Conversation with Copilot, 2024/8/17
(1) 【入門】Docker で MySQL 環境を構築する方法とデータの永続化手順まとめ - カゴヤのサーバー研究室. https://www.kagoya.jp/howto/cloud/container/dockermysql/.
(2) Docker で "no such file or directory" が出たときの対処法 - Qiita. https://qiita.com/bgpat/items/63f4d5220ef340b88790.
(3) 【Docker + mysql】データベースへの接続ができない - Qiita. https://qiita.com/baby-0105/items/9967ef03fc7ffeadfd10.
(4) 【Docker】WordPress のエラー対処法：「Unknown database ‘DB 名’」「Can’t select database .... https://prograshi.com/wordpress/docker-wp-unknown-db/.
(5) Docker で PHP 環境を作ったら”could not find driver”と出た時の対処 | がぶろぐ. https://tech.ateruimashin.com/blog/2020/11/pdo-driver/.
