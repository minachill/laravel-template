# laravel-template
Laravel Docker 開発テンプレート v2025‑06

🚀 はじめに: プロジェクト/リポジトリの作成手順

新しいディレクトリを作成（例: my-laravel-app）し、その中にこのテンプレート一式を配置します。

Git を使う場合は以下で初期化します。

cd my-laravel-app
git init
git add .
git commit -m "Initial commit: bootstrap Laravel + Docker"
# 任意: GitHub/GitLab などにリモートを作成して push
git remote add origin git@github.com:<your-account>/<repo>.git
git push -u origin main

これで テンプレート＝リポジトリのルート が完成。docker-compose.yml と src/（Laravel 本体）が同じ階層にある構造になります。

📁 ディレクトリ構成

├── docker-compose.yml
├── .env.example
├── docker
│   ├── nginx
│   │   └── default.conf
│   ├── php
│   │   └── Dockerfile
│   └── mysql
│       ├── my.cnf
│       └── data/            (初回起動時に自動生成)
└── src/                      (Laravel 本体)

🐳 docker-compose.yml

services:
  nginx:
    image: nginx:1.25-alpine           # ARM/AMD 両対応
    ports:
      - "${NGINX_PORT:-8081}:80"      # ホスト側ポートは環境変数で柔軟に
    volumes:
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./src:/var/www/
    depends_on:
      - php

  php:
    build: ./docker/php
    volumes:
      - ./src:/var/www/

  mysql:
    image: mysql:8.0.35
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel_db
      MYSQL_USER: laravel_user
      MYSQL_PASSWORD: laravel_pass
    command: mysqld --default-authentication-plugin=mysql_native_password
    volumes:
      - ./docker/mysql/data:/var/lib/mysql
      - ./docker/mysql/my.cnf:/etc/mysql/conf.d/my.cnf
    # ports:
    #   - "${MYSQL_PORT:-3307}:3306"   # 必要になったら公開

  phpmyadmin:
    image: lscr.io/linuxserver/phpmyadmin:latest
    environment:
      - PMA_ARBITRARY=1
      - PMA_HOST=mysql
      - PMA_USER=laravel_user
      - PMA_PASSWORD=laravel_pass
    depends_on:
      - mysql
    ports:
      - "${PMA_PORT:-8082}:80"

📦 docker/php/Dockerfile

ARG PHP_VERSION=8.3
FROM php:${PHP_VERSION}-fpm

# --- パッケージ & PHP 拡張 -----------------------------------
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        default-mysql-client \
        libzip-dev zlib1g-dev unzip \
    && docker-php-ext-install pdo_mysql zip \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# --- Composer ------------------------------------------------
COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

# --- PHP 設定 ------------------------------------------------
COPY php.ini /usr/local/etc/php/

WORKDIR /var/www

📝 docker/nginx/default.conf

server {
    listen 80;
    server_name localhost;

    root /var/www/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
        deny all;
    }
}

⚙️ docker/mysql/my.cnf

[mysqld]
character-set-server = utf8mb4
collation-server     = utf8mb4_unicode_ci

🌱 .env.example 追記（DB 周り）

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306            # コンテナ間通信なので 3306 固定
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=laravel_pass

🚀 使い方

ビルド & 起動

docker compose pull          # バイナリイメージ取得
docker compose build --no-cache php
docker compose up -d

Laravel プロジェクト作成（src が空の場合）

docker compose exec php bash -c "composer create-project laravel/laravel:\^12.0 . --prefer-dist"

ブラウザ確認

アプリ:   http://localhost:${NGINX_PORT:-8081}

phpMyAdmin: http://localhost:${PMA_PORT:-8082}

ポート変更 環境変数を付けて NGINX_PORT=8083 PMA_PORT=8084 docker compose up -d のように起動。

🛠 カスタマイズメモ

MySQL をホスト公開したいときは docker-compose.yml 内のコメント行を有効化。

PHP 拡張を追加する場合は Dockerfile の docker-php-ext-install 部分を編集。

CI 用のテストには docker compose run --rm php vendor/bin/pest などを利用。

更新履歴

2025‑06‑29  初版作成

🛠️ 開発環境構築ガイド（Quick Start）

1. このテンプレートを取得

# GitHub から clone
git clone git@github.com:<your‑account>/laravel-docker-template.git
cd laravel-docker-template

2. Laravel 本体をインストール

cd src
composer create-project laravel/laravel:"^12.0" . --prefer-dist
cp .env.example .env && php artisan key:generate
cd ..

3. コンテナ起動

docker compose build php   # 初回のみ
docker compose up -d       # 全サービス起動

4. ブラウザで動作確認

役割

URL

期待される画面

Laravel

http://localhost:${NGINX_PORT:-8081}

Laravel ロゴページ

phpMyAdmin

http://localhost:${PMA_PORT:-8082}

phpMyAdmin ログイン

🔧 カスタマイズメモ

項目

方法

ホスト側ポート

NGINX_PORT / PMA_PORT を .env または compose 変数で変更

PHP 拡張を追加

docker/php/Dockerfile の docker-php-ext-install に追加

MySQL ポート公開

docker-compose.yml の mysql サービスに ports: をコメントイン

Xdebug

docker/php/Dockerfile に pecl install xdebug→docker-php-ext-enable xdebug, 同時に php.ini 追記

Laravel 設定

.env や config/*.php を編集。コミットするとき .env は除外

ファイル配置の推奨

このドキュメント自体は プロジェクトのルート (README.md もしくは docs/SETUP_GUIDE.md) に置くと良い。

IDE の自動生成ファイルや実行時データは .gitignore で管理。

