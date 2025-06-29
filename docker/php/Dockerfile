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