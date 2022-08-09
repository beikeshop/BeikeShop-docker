ARG PHP_VERSION
FROM php:${PHP_VERSION}-alpine


# ARG LARADOCK_PHP_VERSION
# FROM php:${LARADOCK_PHP_VERSION}-alpine

# ARG LARADOCK_PHP_VERSION

# ARG CHANGE_SOURCE=false
ARG CHANGE_SOURCE=true
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories \
;fi

RUN apk --update add wget \
  curl \
  git \
  build-base \
  libmcrypt-dev \
  libxml2-dev \
  linux-headers \
  pcre-dev \
  zlib-dev \
  autoconf \
  cyrus-sasl-dev \
  libgsasl-dev \
  oniguruma-dev \
  libressl \
  libressl-dev \
  supervisor

RUN pecl channel-update pecl.php.net; \
  docker-php-ext-install mysqli mbstring pdo pdo_mysql xml pcntl; \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "1" ]; then \
    php -m | grep -q 'tokenizer'; \
  else \
    docker-php-ext-install tokenizer; \
  fi

#RUN docker-php-ext-install mysqli mbstring pdo pdo_mysql tokenizer xml pcntl xmlrpc
#RUN pecl channel-update pecl.php.net && pecl install memcached mcrypt-1.0.1 && docker-php-ext-enable memcached

###########################################################################
# XMLRPC:
###########################################################################

ARG INSTALL_XMLRPC=true

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
# PHP Memcached:
###########################################################################

ARG INSTALL_MEMCACHED=true

RUN if [ ${INSTALL_MEMCACHED} = true ]; then \
  apk --update add libmemcached-dev; \
  # Install the php memcached extension
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
  pecl install memcached-2.2.0; \
  else \
  pecl install memcached; \
  fi; \
  docker-php-ext-enable memcached; \
  php -m | grep -r 'memcached'; \
  fi















# Add a non-root user to help install ffmpeg:
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN addgroup -g ${PGID} worker && \
    adduser -D -G worker -u ${PUID} worker

#Install BZ2:
ARG INSTALL_BZ2=true
RUN if [ ${INSTALL_BZ2} = true ]; then \
  apk --update add bzip2-dev; \
  docker-php-ext-install bz2; \
  fi
#Install BZ2:
# RUN apk --update add bzip2-dev; \
#   docker-php-ext-install bz2 && \
  # #Install GD package:  
  # apk add --update --no-cache freetype libpng libjpeg-turbo freetype-dev libpng-dev libjpeg-turbo-dev; \
  # docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/; \
  # docker-php-ext-install gd && \

  #Install GD package:  
ARG INSTALL_GD=true
RUN if [ ${INSTALL_GD} = true ]; then \
  apk add --update --no-cache freetype-dev libjpeg-turbo-dev jpeg-dev libpng-dev; \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "4" ]; then \
  docker-php-ext-configure gd --with-freetype --with-jpeg; \
  else \
  docker-php-ext-configure gd --with-freetype-dir=/usr/lib/ --with-jpeg-dir=/usr/lib/ --with-png-dir=/usr/lib/; \
  fi; \
  docker-php-ext-install gd \
  ;fi


#Install GMP package:
ARG INSTALL_GMP=true
RUN if [ ${INSTALL_GMP} = true ]; then \
  apk add --update --no-cache gmp gmp-dev \
  && docker-php-ext-install gmp \
  ;fi

# Install ZipArchive:
# Install ZipArchive:
ARG INSTALL_ZIP_ARCHIVE=true
RUN set -eux; \
  if [ ${INSTALL_ZIP_ARCHIVE} = true ]; then \
  apk --update add libzip-dev && \
  if [ ${PHP_VERSION} = "7.3" ] || [ ${PHP_VERSION} = "7.4" ] || [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  docker-php-ext-configure zip; \
  else \
  docker-php-ext-configure zip --with-libzip; \
  fi && \
  # Install the zip extension
  docker-php-ext-install zip \
  ;fi

# Install PhpRedis package:
ARG INSTALL_PHPREDIS=true
RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
  printf "\n" | pecl install -o -f redis \
  &&  rm -rf /tmp/pear \
  &&  docker-php-ext-enable redis \
;fi

 # Add ffmpeg to horizon
ARG INSTALL_FFMPEG=true
RUN if [ ${INSTALL_FFMPEG} = true ]; then \
  apk add ffmpeg \
;fi

# Install MongoDB drivers:
ARG INSTALL_MONGO=false
RUN if [ ${INSTALL_MONGO} = true ]; then \
  pecl install mongodb \
  && docker-php-ext-enable mongodb \
;fi

# YAML: extension
ARG INSTALL_YAML=true
RUN if [ ${INSTALL_YAML} = true ]; then \
  apk --update add -U --no-cache --virtual temp yaml-dev \
  && apk add --no-cache yaml \
  && docker-php-source extract \
  && pecl channel-update pecl.php.net \
  && pecl install yaml \
  && docker-php-ext-enable yaml \
  && pecl clear-cache \
  && docker-php-source delete \
  && apk del temp \
;fi

RUN rm /var/cache/apk/* \
  && mkdir -p /var/www

COPY supervisord.conf /etc/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisord.conf"]

RUN php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

WORKDIR /etc/supervisor/conf.d/
