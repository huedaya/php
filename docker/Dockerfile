FROM composer:2 AS composer
COPY --from=composer /usr/bin/composer /usr/local/bin/compose

FROM php:8.0-apache

WORKDIR /app

#-----  Install unzip utility and libs needed by zip PHP extension
RUN apt-get update --fix-missing && apt-get install -y zlib1g-dev libzip-dev zip unzip libicu-dev git g++
RUN docker-php-ext-install zip intl pdo pdo_mysql mysqli opcache bcmath iconv ctype
RUN docker-php-ext-configure intl
RUN docker-php-ext-enable intl
RUN a2enmod rewrite

#-----  Skip Host verification for git
ARG USER_HOME_DIR=/root
RUN mkdir ${USER_HOME_DIR}/.ssh/ \
    && echo "StrictHostKeyChecking no " > ${USER_HOME_DIR}/.ssh/config

#-----  Apache config
ARG APACHE_DOCUMENT_ROOT=/app/public
RUN a2enmod rewrite deflate headers \
    # setup web document root
    && sed -ri -e "s!/var/www/html!${APACHE_DOCUMENT_ROOT}!g" /etc/apache2/sites-available/*.conf \
    && sed -ri -e "s!/var/www/!${APACHE_DOCUMENT_ROOT}!g" /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf \
    # enable sites
    && a2ensite 000-default

#----- Setup php.ini
RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
    && sed -i "s|error_reporting\s=\sE_ALL|error_reporting= E_ALL \| E_STRICT|g" /usr/local/etc/php/php.ini \
    && sed -i "s|memory_limit = 1024M|memory_limit = 2024M|g" /usr/local/etc/php/php.ini \
    && sed -i "s|max_execution_time = 30|max_execution_time = 300|g" /usr/local/etc/php/php.ini

# Setup OPCache
RUN sed -i \
    -e "s/\$/\nopcache.enable=1/" \
    -e "s/\$/\nopcache.enable_cli=1/" \
    -e "s/\$/\nopcache.memory_consumption=200/" \
    -e "s/\$/\nopcache.interned_strings_buffer=8/" \
    -e "s/\$/\nopcache.max_accelerated_files=10000/" \
    -e "s/\$/\nopcache.revalidate_freq=2/" \
    -e "s/\$/\nopcache.validate_timestamps=1/" \
    -e "s/\$/\nopcache.max_wasted_percentage=10/" \
    -e "s/\$/\nopcache.interned_strings_buffer=10/" \
    /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini

#----- Cleaning
RUN apt-get clean -y