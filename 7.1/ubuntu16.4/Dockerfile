FROM ubuntu:xenial-20200807

LABEL name="nmoeini/php:7.1-xenial-20200807" mainteiner="Navid Moeini <nmoeiny@yahoo.com>" version="0.1"

ARG DEBIAN_FRONTEND
ARG PHP_VERSION
ARG GOSU_VERSION

ENV PHP_VERSION=${PHP_VERSION:-7.1} \
    DEBIAN_FRONTEND=${DEBIAN_FRONTEND:-noninteractive} \
    GOSU_VERSION=${GOSU_VERSION:-1.12}

# add gosu for easy step-down from root
# https://github.com/tianon/gosu/releases
RUN set -eux \
	&& savedAptMark="$(apt-mark showmanual)" \
    && apt-get update -y \
	&& apt-get install -y --no-install-recommends \
        ca-certificates \
        dirmngr \
        gnupg \
        wget \
        vim \
        nano \
	&& dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& gpgconf --kill all \
	&& rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& apt-mark auto '.*' > /dev/null \
	&& [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark > /dev/null \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu --version \
	&& gosu nobody true \
##  Clear cache \
##  -----------------------------------------------------------------------  ## \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV PHP_EXTRA_CONFIGURE_ARGS --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data --nodaemonize

RUN set -eux \
	&& savedAptMark="$(apt-mark showmanual)" \
	&& apt-get update \
    && apt-get install -y --no-install-recommends \
        software-properties-common \
    \
##  Add PHP Repository\
##  ------------------------------------------------------------------------  ## \
    && LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php \
    && apt-get update -y \
    && apt-get install -y \
        aptitude \
        \
##  Install PHP Packages \
##  ------------------------------------------------------------------------  ## \
        "php$PHP_VERSION" \
        "php$PHP_VERSION-fpm" \
        "php$PHP_VERSION-intl" \
        "php$PHP_VERSION-mbstring" \
        "php$PHP_VERSION-xml" \
        "php$PHP_VERSION-curl" \
        "php$PHP_VERSION-mysql" \
        "php$PHP_VERSION-soap" \
        "php$PHP_VERSION-bcmath" \
        "php$PHP_VERSION-zip" \
        "php$PHP_VERSION-tidy" \
        php-apcu \
        php-apcu-bc \
    && "php-fpm$PHP_VERSION" -m \
    \
##  Clear cache \
##  -----------------------------------------------------------------------  ## \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN set -eux \
    && mkdir -p /run/php \
    && chown -R www-data:www-data /run/php \
    && chmod -R 0755 /run/php \
    && chmod -R 0777 /var/log \
    \
## Update config to listen to port 9000 instead of socket \
##  -----------------------------------------------------------------------  ## \
    &&  sed -i 's/listen = \/\(.*$\)/; listen = \/\1\nlisten = 9000/' /etc/php/$PHP_VERSION/fpm/pool.d/www.conf \
    \
    && if [ ! -d /var/www/html ]; then \
            mkdir -p /var/www/html; \
            touch /var/www/html/info.php; \
            echo '<?php \
                    phpinfo();' > /var/www/html/info.php; \
            chmod +x /var/www/html/info.php; \
            chown -R www-data:www-data /var/www/html; \
        fi \
    && ln -s /usr/sbin/php-fpm$PHP_VERSION /usr/sbin/php-fpm

ADD ./docker-php* /usr/local/bin/

RUN chmod +x /usr/local/bin/docker-php*

ENTRYPOINT ["docker-php-entrypoint"]

WORKDIR /var/www/html

STOPSIGNAL SIGQUIT

EXPOSE 9000

CMD ["php-fpm"]