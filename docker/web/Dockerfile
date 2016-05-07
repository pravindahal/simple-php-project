FROM php:7.0-fpm
RUN docker-php-ext-install opcache

ADD ./project /var/www/project
RUN if [ -f /var/www/project/.env ]; then rm /var/www/project/.env; fi
WORKDIR /var/www/project

COPY ./docker/web/php.ini /usr/local/etc/php/
COPY ./docker/web/www.conf /usr/local/etc/php-fpm.d/zzz-www.conf

EXPOSE 80
