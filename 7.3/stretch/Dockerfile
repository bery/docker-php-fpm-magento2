FROM php:7.3.18-fpm-stretch

MAINTAINER Lukas Beranek <lukas@beecom.io>

ENV APCU_VERSION 5.1.18
ENV REDIS_VERSION 4.3.0
ENV MCRYPT_VERSION 1.0.3
ENV PHP_MEMORY_LIMIT 2G
ENV PHP_PORT 9000
ENV PHP_PM dynamic
ENV PHP_PM_MAX_CHILDREN 20
ENV PHP_PM_START_SERVERS 10
ENV PHP_PM_MIN_SPARE_SERVERS 4
ENV PHP_PM_MAX_SPARE_SERVERS 10

RUN apt-get update && apt-get install -y gnupg2 && \
    export GCSFUSE_REPO=gcsfuse-stretch && \
    echo "deb http://packages.cloud.google.com/apt $GCSFUSE_REPO main" | tee /etc/apt/sources.list.d/gcsfuse.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    gcsfuse \
    wget \
    libxml2 \
    libxml2-dev \
    patch \
    libzip-dev \
    libsodium-dev \
    zlib1g-dev \
    libxslt-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    unzip \
    procps \
    git && \
    rm -rf /var/lib/apt/lists/*

RUN  docker-php-ext-configure gd \
    --with-gd \
    --with-freetype-dir=/usr/include/ \
    --with-png-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ && \
  NPROC=$(getconf _NPROCESSORS_ONLN) && \
  docker-php-ext-install -j${NPROC} gd

RUN curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$REDIS_VERSION.tar.gz \
    && tar xfz /tmp/redis.tar.gz \
    && rm -r /tmp/redis.tar.gz \
    && mkdir -p /usr/src/php/ext \
    && mv phpredis-* /usr/src/php/ext/redis

RUN docker-php-ext-install \
  bcmath \
  opcache \
  pdo_mysql \
  soap \
  zip \
  xsl \
  intl \
  redis \
  sodium \
  sockets \
  exif \
  pcntl && \
  pecl install \
  apcu-${APCU_VERSION} && \
  docker-php-ext-enable --ini-name 20-apcu.ini apcu && \
  pecl install mcrypt-${MCRYPT_VERSION} && \
  docker-php-ext-enable \
  mcrypt \
  sodium \
  sockets

RUN \
  export NEWRELIC_VERSION=$(curl -sS https://download.newrelic.com/php_agent/release/ | sed -n 's/.*>\(.*linux\).tar.gz<.*/\1/p') && \
      curl -L -o newrelic.tar.gz https://download.newrelic.com/php_agent/release/$NEWRELIC_VERSION.tar.gz && \
      gzip -dc newrelic.tar.gz | tar xf - && \
      cd $NEWRELIC_VERSION && \
      NR_INSTALL_SILENT=true NR_INSTALL_USE_CP_NOT_LN=true ./newrelic-install install && \
      cd .. && \
      rm -rf $NEWRELIC_VERSION && \
      rm newrelic.tar.gz

RUN curl -sS https://getcomposer.org/installer | \
  php -- --install-dir=/usr/local/bin --filename=composer

RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

COPY conf/php.ini /usr/local/etc/php/conf.d/10_php.ini
COPY conf/php-fpm.conf /usr/local/etc/php-fpm.d/www.conf
COPY ./conf/30_newrelic.ini /usr/local/etc/php/conf.d/30_newrelic.ini