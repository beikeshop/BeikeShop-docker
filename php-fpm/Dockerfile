ARG PHP_VERSION
ARG BASE_IMAGE_TAG_PREFIX=latest
# FROM laradock/php-fpm:${BASE_IMAGE_TAG_PREFIX}-8.0
FROM laradock/php-fpm:${BASE_IMAGE_TAG_PREFIX}-${PHP_VERSION}

ENV DEBIAN_FRONTEND noninteractive

ARG CHANGE_SOURCE=true
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    # Change application source from deb.debian.org to aliyun source
    sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security-cdn.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
;fi

RUN set -eux; \
    apt-get update; \
    apt-get upgrade -y; \
    apt-get install -y --no-install-recommends \
            curl \
            libmemcached-dev \
            libz-dev \
            libpq-dev \
            libjpeg-dev \
            libpng-dev \
            libfreetype6-dev \
            libssl-dev \
            libxml2-dev \
            libmcrypt-dev; \
     rm -rf /var/lib/apt/lists/*

RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    apt-get install -yqq \
      apt-utils libzip-dev zip unzip && \
      if [ ${PHP_VERSION} = "7.3" ] || [ ${PHP_VERSION} = "7.4" ] || [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
        docker-php-ext-configure zip; \
      else \
        docker-php-ext-configure zip --with-libzip; \
      fi && \
      docker-php-ext-install zip && \
      php -m | grep -q 'zip'

###########################################################################
# XMLRPC:
###########################################################################
ARG INSTALL_XMLRPC=false
RUN if [ ${INSTALL_XMLRPC} = true ]; then \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  pecl install xmlrpc-1.0.0RC3; \
  docker-php-ext-enable xmlrpc; \
  else \
  docker-php-ext-install xmlrpc; \
  fi; \
  php -m | grep -r 'xmlrpc'; \
  fi

###########################################################################
# PHP REDIS EXTENSION
###########################################################################

ARG INSTALL_PHPREDIS=true
RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    # Install Php Redis Extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl install -o -f redis-4.3.0; \
    else \
      pecl install -o -f redis; \
    fi \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis \
;fi


# RUN pecl install -o -f redis \
#     && rm -rf /tmp/pear \
#     && docker-php-ext-enable redis \
#     # memcached
#     && pecl install memcached-3.1.3 \
#     && docker-php-ext-enable memcached \
#     # exif
#     && docker-php-ext-install exif \
#     # && docker-php-ext-install opcache \
#     && docker-php-ext-install mysqli  \
#     # intl
#     && apt-get install -y zlib1g-dev libicu-dev g++ \
#     && docker-php-ext-configure intl \
#     && docker-php-ext-install intl

###########################################################################
# PHP Memcached:
###########################################################################

ARG INSTALL_MEMCACHED=true

RUN if [ ${INSTALL_MEMCACHED} = true ]; then \
    # Install the php memcached extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      echo '' | pecl -q install memcached-2.2.0; \
    else \
      echo '' | pecl -q install memcached; \
    fi \
    && docker-php-ext-enable memcached \
;fi

###########################################################################
# Exif:
###########################################################################

ARG INSTALL_EXIF=true

RUN if [ ${INSTALL_EXIF} = true ]; then \
    # Enable Exif PHP extentions requirements
    docker-php-ext-install exif \
;fi

ARG INSTALL_INTL=true

RUN if [ ${INSTALL_INTL} = true ]; then \
    # Install intl and requirements
    apt-get install -yqq zlib1g-dev libicu-dev g++ && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl \
;fi

###########################################################################
# Mysqli Modifications:
###########################################################################

ARG INSTALL_MYSQLI=true

RUN if [ ${INSTALL_MYSQLI} = true ]; then \
    docker-php-ext-install mysqli \
;fi

# Image optimizers:
###########################################################################

USER root

ARG INSTALL_IMAGE_OPTIMIZERS=true

RUN if [ ${INSTALL_IMAGE_OPTIMIZERS} = true ]; then \
    apt-get install -yqq jpegoptim optipng pngquant gifsicle \
;fi

###########################################################################
# ImageMagick:
###########################################################################
USER root
RUN apt-get install -y libmagickwand-dev imagemagick \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
    # yaml
    && apt-get install libyaml-dev -y \
    && pecl install yaml \
    && docker-php-ext-enable yaml \
    # locales
    && apt-get install -y locales \
    && echo '' >> /usr/share/locale/locale.alias \
    && temp="${ADDITIONAL_LOCALES%\"}" \
    && temp="${temp#\"}" \
    && for i in ${temp}; do sed -i "/$i/s/^#//g" /etc/locale.gen; done \
    && locale-gen \
    # ffmpeg
    && apt-get -y install ffmpeg

###########################################################################
# ImageMagick:
###########################################################################

USER root

ARG INSTALL_IMAGEMAGICK=false
ARG IMAGEMAGICK_VERSION=latest
ENV IMAGEMAGICK_VERSION ${IMAGEMAGICK_VERSION}

RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get install -yqq libmagickwand-dev imagemagick && \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
      cd /tmp && \
      if [ ${IMAGEMAGICK_VERSION} = "latest" ]; then \
        git clone https://github.com/Imagick/imagick; \
      else \
        git clone --branch ${IMAGEMAGICK_VERSION} https://github.com/Imagick/imagick; \
      fi && \
      cd imagick && \
      phpize && \
      ./configure && \
      make && \
      make install && \
      rm -r /tmp/imagick; \
    else \
      pecl install imagick; \
    fi && \
    docker-php-ext-enable imagick; \
    php -m | grep -q 'imagick' \
;fi

RUN set -xe; php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/
# COPY ./opcache.ini /usr/local/etc/php/conf.d/opcache.ini

# Clean up
USER root
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN groupadd -g ${PGID} php-fpm && \
    useradd -u ${PUID} -g php-fpm -m php-fpm

ARG LOCALE=POSIX
ENV LC_ALL ${LOCALE}

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000
