# Base image
FROM php:8.2-fpm-alpine as base

RUN apk update --no-cache && \
    apk upgrade --no-cache
RUN apk add --no-cache \
        supervisor nginx libpq

FROM base as build

RUN apk add --no-cache \
    $PHPIZE_DEPS \
    linux-headers
RUN apk add --no-cache \
    freetype-dev \
    jpeg-dev \
    icu-dev \
    libzip-dev \
    postgresql-dev

#####################################
# PHP Extensions
#####################################
# Install the PHP shared memory driver
RUN pecl install APCu && \
    docker-php-ext-enable apcu

# Install the PHP bcmath extension
RUN docker-php-ext-install bcmath

# Install for image manipulation
RUN docker-php-ext-install exif

# Install the PHP graphics library
RUN docker-php-ext-configure gd \
    --with-freetype \
    --with-jpeg
RUN docker-php-ext-install gd

# Install the PHP intl extention
RUN docker-php-ext-install intl

# Install the PHP mysqli extention
RUN docker-php-ext-install mysqli && \
    docker-php-ext-enable mysqli

# Install the PHP pgsql extention
RUN docker-php-ext-install pgsql

# Install the PHP opcache extention
RUN docker-php-ext-enable opcache

# Install the PHP pcntl extention
RUN docker-php-ext-install pcntl

# Install the PHP pdo extention
RUN docker-php-ext-install pdo

# Install the PHP pdo_pgsql extention
RUN docker-php-ext-install pdo_pgsql

# Install the PHP pdo_mysql extention
RUN docker-php-ext-install pdo_mysql

# Install the PHP redis driver
RUN pecl install redis && \
    docker-php-ext-enable redis

# install XDebug but without enabling
RUN pecl install xdebug

# Install the PHP zip extention
RUN docker-php-ext-install zip

FROM base as target

#####################################
# Copy php-fpm configuration template
#####################################
COPY --chown=www-data:www-data php-fpm.conf.tpl /tmp/php-fpm.conf.tpl

#####################################
# Copy nginx configuration template
#####################################
COPY --chown=www-data:www-data nginx.conf.tpl /tmp/nginx.conf.tpl

#####################################
# Copy supervisor configuration
#####################################
COPY --chown=www-data:www-data supervisord.conf /etc/supervisor/supervisord.conf

#####################################
# Copy entrypoint script
#####################################
COPY --chown=www-data:www-data --chmod=755 entrypoint.sh /entrypoint.sh

#####################################
# Install necessary libraries
#####################################
RUN apk add --no-cache \
    gettext \
    freetype \
    jpeg \
    icu \
    libzip \
    && touch /usr/local/var/log/php-fpm.log \
    && chown -R www-data:www-data \
    /var/www \
    /var/log/nginx \
    /var/lib/nginx \
    /etc/nginx \
    /etc/supervisor \
    /usr/local/etc/php-fpm.conf \
    /usr/local/var/log/php-fpm.log \
    /entrypoint.sh \
    && sed -i 's/\r//' /entrypoint.sh

#####################################
# Copy extensions from build stage
#####################################
COPY --from=build /usr/local/lib/php/extensions/no-debug-non-zts-20220829/* /usr/local/lib/php/extensions/no-debug-non-zts-20220829
COPY --from=build /usr/local/etc/php/conf.d/* /usr/local/etc/php/conf.d

#####################################
# Composer
#####################################
RUN curl -s https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer

ENTRYPOINT ["/entrypoint.sh"]

CMD []

USER www-data

EXPOSE 8080
