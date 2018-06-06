FROM phusion/baseimage:latest

RUN DEBIAN_FRONTEND=noninteractive
RUN locale-gen en_US.UTF-8

ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV LC_CTYPE=en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV TERM xterm

# Get PPA for PHP extensions.
RUN apt-get install -y software-properties-common && \
    add-apt-repository -y ppa:ondrej/php

# Install PHP extensions.
RUN apt-get update && \
    apt-get install -y --allow-downgrades --allow-remove-essential \
        --allow-change-held-packages \
        php7.2-cli \
        php7.2-common \
        php7.2-curl \
        php7.2-intl \
        php7.2-json \
        php7.2-xml \
        php7.2-mbstring \
        php7.2-mysql \
        php7.2-pgsql \
        php7.2-sqlite \
        php7.2-sqlite3 \
        php7.2-zip \
        php7.2-bcmath \
        php7.2-memcached \
        php7.2-gd \
        php7.2-dev \
        pkg-config \
        libcurl4-openssl-dev \
        libedit-dev \
        libssl-dev \
        libxml2-dev \
        xz-utils \
        libsqlite3-dev \
        sqlite3 \
        git \
        curl \
        vim \
        nano \
        postgresql-client \
    && apt-get clean

# Install Composer.
RUN curl -s http://getcomposer.org/installer | php && \
    echo "export PATH=${PATH}:/var/www/vendor/bin" >> ~/.bashrc && \
    mv composer.phar /usr/local/bin/composer
RUN . ~/.bashrc

# Create a non-root user.
USER root

ARG PUID=1000
ARG PGID=1000

ENV PUID ${PUID}
ENV PGID ${PGID}

RUN groupadd -g ${PGID} whale && \
    useradd -u ${PUID} -g whale -m whale -G docker_env && \
    usermod -p "*" whale

# Set timezone.
ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# User aliases.
USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/whale/aliases.sh
RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/whale/aliases.sh && \
    chown whale:whale /home/whale/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc

USER whale

RUN echo "" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc

# Composer touches.
USER root

COPY ./composer.json /home/whale/.composer/composer.json
RUN chown -R whale:whale /home/whale/.composer

USER whale

RUN composer global install
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

# Setup Crontab.
USER root

COPY ./crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d

# Install XDebug.
USER root

RUN apt-get install -y --force-yes php7.2-xdebug && \
    sed -i 's/^;//g' /etc/php/7.2/cli/conf.d/20-xdebug.ini && \
    echo "alias phpunit='php -dzend_extension=xdebug.so /var/www/vendor/bin/phpunit'" >> ~/.bashrc

COPY ./ini/xdebug.ini /etc/php/7.2/cli/conf.d/xdebug.ini

# Setup SSH.
COPY ./ssh/id_rsa /tmp/id_rsa
COPY ./ssh/id_rsa.pub /tmp/id_rsa.pub

RUN rm -f /etc/service/sshd/down && \
    cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys \
        && cat /tmp/id_rsa.pub >> /root/.ssh/id_rsa.pub \
        && cat /tmp/id_rsa >> /root/.ssh/id_rsa \
        && rm -f /tmp/id_rsa* \
        && chmod 644 /root/.ssh/authorized_keys /root/.ssh/id_rsa.pub \
    && chmod 400 /root/.ssh/id_rsa \
    && cp -rf /root/.ssh /home/whale \
    && chown -R whale:whale /home/whale/.ssh

# Install MongoDB.
RUN pecl -q install mongodb && \
    echo "extension=mongodb.so" >> /etc/php/7.2/mods-available/mongodb.ini && \
    ln -s /etc/php/7.2/mods-available/mongodb.ini /etc/php/7.2/cli/conf.d/30-mongodb.ini

# Install PHP Redis.
RUN printf "\n" | pecl -q install -o -f redis && \
    echo "extension=redis.so" >> /etc/php/7.2/mods-available/redis.ini && \
    phpenmod redis

# Install Swoole.
RUN pecl -q install swoole && \
    echo "extension=swoole.so" >> /etc/php/7.2/mods-available/swoole.ini && \
    ln -s /etc/php/7.2/mods-available/swoole.ini /etc/php/7.2/cli/conf.d/20-swoole.ini

# Install Node.
ENV NVM_DIR /home/whale/.nvm

RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash && \
    . $NVM_DIR/nvm.sh && \
    nvm install stable && \
    nvm use stable && \
    nvm alias stable && \
    npm install -g gulp bower vue-cli

RUN  echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="/home/whale/.nvm"' >> ~/.bashrc && \
    echo '[ -s "/home/whale/.nvm/nvm.sh" ] && . "/home/whale/.nvm/nvm.sh"' >> ~/.bashrc

USER root

RUN echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="/home/whale/.nvm"' >> ~/.bashrc && \
    echo '[ -s "/home/whale/.nvm/nvm.sh" ] && . "/home/whale/.nvm/nvm.sh"' >> ~/.bashrc

ENV PATH $PATH:/home/whale/.nvm/versions/node/vstable/bin

# Install YARN.
USER whale

RUN [ -s "/home/whale/.nvm/nvm.sh" ] && . "/home/whale/.nvm/nvm.sh" && \
    curl -o- -L https://yarnpkg.com/install.sh | bash && \
    echo "" >> ~/.bashrc && \
    echo 'export PATH="/home/whale/.yarn/bin:$PATH"' >> ~/.bashrc

USER root
RUN echo "" >> ~/.bashrc && \
    echo 'export YARN_DIR="/home/whale/.yarn"' >> ~/.bashrc && \
    echo 'export PATH="/home/whale/.yarn/bin:$PATH"' >> ~/.bashrc

# Install Laravel Envoy.
USER whale

RUN composer global require "laravel/envoy"

# Install Laravel installer.
USER root
RUN composer global require "laravel/installer"

# Install image optimizers.
USER root

RUN apt-get install -y --force-yes jpegoptim optipng pngquant gifsicle && \
    . ~/.bashrc && npm install -g svgo

# Final check-ups.
RUN php -v | head -n 1 | grep -q "PHP 7.2."

USER root

RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

WORKDIR /var/www
