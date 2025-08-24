# ==== Stage 1: Build ====
FROM php:8.4-fpm as build

RUN apt-get update && apt-get install -y \
    git unzip libicu-dev libzip-dev libpq-dev libssl-dev \
 && docker-php-ext-install intl pdo_mysql zip opcache \
 && pecl install redis \
 && docker-php-ext-enable redis \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www

# Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

COPY new-api/composer.json new-api/composer.lock ./
RUN composer install --no-dev --optimize-autoloader --no-interaction

COPY new-api/ ./

# ==== Stage 2: Runtime ====
FROM php:8.4-fpm

RUN apt-get update && apt-get install -y libicu-dev \
 && docker-php-ext-install intl pdo_mysql zip opcache \
 && pecl install redis \
 && docker-php-ext-enable redis \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www

COPY --from=build /var/www /var/www

CMD ["php-fpm"]