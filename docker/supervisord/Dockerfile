FROM php:7.2-alpine

RUN apk --update add wget \
    curl \
    git \
    build-base \
    libmemcached-dev \
    libmcrypt-dev \
    libxml2-dev \
    zlib-dev \
    autoconf \
    cyrus-sasl-dev \
    libgsasl-dev \
    supervisor

RUN docker-php-ext-install mysqli mbstring pdo pdo_mysql tokenizer xml
RUN pecl channel-update pecl.php.net && pecl install memcached mcrypt-1.0.1 && docker-php-ext-enable memcached

RUN rm /var/cache/apk/* \
    && mkdir -p /var/www

# Supervisord setup.
COPY ./supervisord.conf /etc/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisord.conf"]

# Final touches.
RUN php -v | head -n 1 | grep -q "PHP 7.2."

WORKDIR /etc/supervisor/conf.d/
