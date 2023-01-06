FROM php:8-fpm-alpine
RUN apk update
RUN apk add php81-dev gcc make g++ zlib-dev icu-dev bash git openssl yaml-dev php81-pecl-xdebug
RUN apk add --update linux-headers

# PHP extensions
RUN docker-php-ext-install intl pdo pdo_mysql

RUN pecl channel-update pecl.php.net

# For YAML
RUN apk add --update --no-cache \
          yaml && \
      # Build dependency for YAML \
      apk add --update --no-cache --virtual .yaml-build \
          yaml-dev && \
      pecl install yaml && \
      docker-php-ext-enable yaml; \
      apk del .yaml-build

# For Xdebug
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN echo "xdebug.mode=develop,coverage,debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.client_host=host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# For code coverage
RUN echo "memory_limit=-1" >> /usr/local/etc/php/conf.d/docker-php-memory-limit.ini

COPY --from=composer /usr/bin/composer /usr/bin/composer

SHELL ["/bin/bash", "-c"]