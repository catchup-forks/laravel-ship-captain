FROM php:7.2-fpm

# Install PHP extensions.
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    curl \
    libmemcached-dev \
    libz-dev \
    libpq-dev \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    libssl-dev \
    libmcrypt-dev \
  && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install pdo_mysql \
    && docker-php-ext-install pdo_pgsql \
    && docker-php-ext-configure gd \
    --enable-gd-native-ttf \
    --with-jpeg-dir=/usr/lib \
    --with-freetype-dir=/usr/include/freetype2 && \
    docker-php-ext-install gd

# Install XDebug.
RUN pecl install xdebug && \
    docker-php-ext-enable xdebug

COPY ./ini/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini
    
# Install PHP Redis.
RUN printf "\n" | pecl install -o -f redis \
    &&  rm -rf /tmp/pear \
    &&  docker-php-ext-enable redis

# Install Swoole.
RUN pecl install swoole \
    &&  docker-php-ext-enable swoole

# Install MongoDB.
RUN pecl install mongodb && \
    docker-php-ext-enable mongodb

# Install ZIP.
RUN docker-php-ext-install zip

# Install BCMath.
RUN docker-php-ext-install bcmath

# Install Memcached.
RUN curl -L -o /tmp/memcached.tar.gz "https://github.com/php-memcached-dev/php-memcached/archive/php7.tar.gz" \
    && mkdir -p memcached \
    && tar -C memcached -zxvf /tmp/memcached.tar.gz --strip 1 \
    && ( \
        cd memcached \
        && phpize \
        && ./configure \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r memcached \
    && rm /tmp/memcached.tar.gz \
    && docker-php-ext-enable memcached

# Install EXIF.
RUN docker-php-ext-install exif

# Install OPCache.
RUN docker-php-ext-install opcache

COPY ./ini/opcache.ini /usr/local/etc/php/conf.d/opcache.ini

# Install MySQLi.
RUN docker-php-ext-install mysqli

# Install Tokenizer.
RUN docker-php-ext-install tokenizer

# Install INTL.
RUN apt-get update -yqq && \
    apt-get install -y zlib1g-dev libicu-dev g++ && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl

# Install image optimizers.
RUN apt-get install -y --force-yes jpegoptim optipng pngquant gifsicle

# Final touches.
RUN php -v | head -n 1 | grep -q "PHP 7.2."
COPY ./ini/laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

USER root

RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

RUN usermod -u 1000 www-data

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000