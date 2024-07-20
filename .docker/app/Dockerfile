# Node Builder phase
FROM node:22.5 as node-builder

# Settings
USER 0

# Prepare workfir
WORKDIR /build

# Copy files
COPY package.json .
COPY yarn*.lock .

RUN yarn install


# Composer Builder phase
FROM php:8.3 AS composer-builder

# Settings
ENV COMPOSER_ALLOW_SUPERUSER=1
USER 0

# Prepare workfir
WORKDIR /build

# Copy files
COPY composer.json .
COPY composer*.lock .
COPY --from=node-builder /build .

# Install base
RUN apt-get update && \
    apt-get install -y --force-yes --no-install-recommends \
        libmemcached-dev \
        libzip-dev \
        libz-dev \
        libzip-dev \
        libpq-dev \
        libjpeg-dev \
        libpng-dev \
        libfreetype6-dev \
        libssl-dev \
        openssh-server \
        libmagickwand-dev \
        git \
        cron \
        nano \
        libxml2-dev \
        libreadline-dev \
        libgmp-dev \
        mariadb-client \
        unzip

# Install soap extention
RUN docker-php-ext-install soap  \
    exif \
    pcntl \
    zip \
    pdo_mysql \
    bcmath \
    intl \
    gmp

# Install PHPRedis extention
RUN pecl install redis && docker-php-ext-enable redis

# Install GD extention
RUN docker-php-ext-install gd && docker-php-ext-configure gd --with-freetype --with-jpeg

# Install the xdebug extension
RUN pecl install xdebug

# Install the php memcached extension
RUN pecl install memcached && docker-php-ext-enable memcached

RUN rm -r /var/lib/apt/lists/*

# Install composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer

RUN composer install

# Nginx Unit Final build
FROM unit:php8.3

# Settings
ARG USER_ID=1001
ENV TZ=UTC
ENV HOME /var/www/html
ENV APP_ENV=local
USER 0

# Prepare workfir
WORKDIR /var/www/html

# Copy files
COPY ".docker/app/php-${APP_ENV}.ini" /usr/local/etc/php/conf.d/php.ini
COPY .docker/app/config.json /docker-entrypoint.d/config.json
COPY .docker/app/entrypoint.sh /docker-entrypoint.d/entrypoint.sh
COPY . /var/www/html
COPY --from=composer-builder --chown=unit:unit /build .

# Update package lists and install required dependencies
RUN apt-get update && \
    apt-get install -y --force-yes --no-install-recommends \
        libmemcached-dev \
        libzip-dev \
        libz-dev \
        libzip-dev \
        libpq-dev \
        libjpeg-dev \
        libpng-dev \
        libfreetype6-dev \
        libssl-dev \
        openssh-server \
        libmagickwand-dev \
        git \
        cron \
        nano \
        libxml2-dev \
        libreadline-dev \
        libgmp-dev \
        mariadb-client \
        unzip

# Install soap extention
RUN docker-php-ext-install soap  \
    exif \
    pcntl \
    zip \
    pdo_mysql \
    bcmath \
    intl \
    gmp

# Install PHPRedis extention
RUN pecl install redis && docker-php-ext-enable redis

# Install GD extention
RUN docker-php-ext-install gd && docker-php-ext-configure gd --with-freetype --with-jpeg

# Install the xdebug extension
RUN pecl install xdebug

# Install the php memcached extension
RUN pecl install memcached && docker-php-ext-enable memcached

RUN rm -r /var/lib/apt/lists/*

RUN chown -R unit /run \
    && mkdir -p /var/lib/unit \
    && chown -R unit /var/lib/unit \
    && chmod o+x /docker-entrypoint.d/entrypoint.sh

RUN rm -rf /var/www/html/.docker
RUN php artisan storage:link
RUN chown -R unit:unit /var/www/html

# USER unit

EXPOSE 8080
