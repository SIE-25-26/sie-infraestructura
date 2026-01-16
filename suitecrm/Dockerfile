FROM php:8.2-apache-bookworm

RUN apt-get update && apt-get install -y --no-install-recommends \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libzip-dev \
    libicu-dev \
    libxml2-dev \
    libc-client-dev \
    libkrb5-dev \
    libldap2-dev \
    libcurl4-openssl-dev \
    libonig-dev \
    unzip \
    && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
    && docker-php-ext-install -j$(nproc) \
    gd zip mysqli pdo_mysql intl xml opcache imap soap ldap curl mbstring

RUN a2enmod rewrite
COPY ./SuiteCRM-8.9.1 /var/www/html
RUN chown -R www-data:www-data /var/www/html

#### ToDo: Probar a meter el ZIP y no la carpeta ya descomprimida que ocupa el doble 
## Copiamos el ZIP al contenedor
#COPY ./SuiteCRM-8.9.1.zip /tmp/SuiteCRM.zip
#
## Descomprimimos y limpiamos
#RUN unzip /tmp/SuiteCRM.zip -d /var/www/html \
#    && rm /tmp/SuiteCRM.zip \
#    && chown -R www-data:www-data /var/www/html

# Configuración personalizada de PHP para SuiteCRM
RUN echo "memory_limit=512M" > /usr/local/etc/php/conf.d/suitecrm.ini \
    && echo "upload_max_filesize=20M" >> /usr/local/etc/php/conf.d/suitecrm.ini \
    && echo "post_max_size=20M" >> /usr/local/etc/php/conf.d/suitecrm.ini \
    && echo "max_execution_time=300" >> /usr/local/etc/php/conf.d/suitecrm.ini \
    && echo "display_errors=Off" >> /usr/local/etc/php/conf.d/suitecrm.ini \
    && echo "error_reporting=E_ALL & ~E_DEPRECATED & ~E_STRICT & ~E_NOTICE & ~E_WARNING" >> /usr/local/etc/php/conf.d/suitecrm.ini

WORKDIR /var/www/html
EXPOSE 80