# ==== Stage 1: Node build (webpack, frontend) ====
FROM node:16 as frontend_build

WORKDIR /app
COPY legacy/package*.json ./
RUN npm install
COPY legacy/webpack.config.js ./
COPY legacy/assets/ ./assets/
RUN npm run build    # соберет статику в /app/public/build

# ==== Stage 2: PHP build (composer deps) ====
FROM php:7.4-fpm as php_build

# Установка системных пакетов и расширений
RUN apt-get update && apt-get install -y \
    git unzip libicu-dev libzip-dev libpq-dev libonig-dev \
 && docker-php-ext-install intl pdo_mysql zip opcache \
 && pecl install redis \
 && docker-php-ext-enable redis

WORKDIR /var/www

# Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

COPY legacy/composer.json legacy/composer.lock ./
RUN composer install --no-dev --optimize-autoloader --no-interaction

# ==== Stage 3: Runtime Legacy ====
FROM php:7.4-fpm

# Устанавливаем пакеты
RUN apt-get update && apt-get install -y nginx supervisor \
 && docker-php-ext-install intl pdo_mysql zip \
 && pecl install redis \
 && docker-php-ext-enable redis \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www

# Копируем vendor и код
COPY --from=php_build /var/www/vendor ./vendor
COPY legacy/ ./

# Копируем собранный фронт из Node stage
COPY --from=frontend_build /app/public/build ./public/build

# Nginx конфиг
COPY legacy/docker/nginx.conf /etc/nginx/conf.d/default.conf

# Supervisor будет запускать php-fpm и nginx вместе
COPY legacy/docker/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]