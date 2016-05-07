FROM php:7.0-fpm
ENV apt_get_install "apt-get install -y --force-yes"
ENV apt_get_remove "apt-get remove -y --force-yes"
ENV apt_get_autoremove "apt-get autoremove -y --force-yes"
ENV apt_get_clean "apt-get clean -y --force-yes"
RUN apt-get update -y
RUN $apt_get_install supervisor
RUN docker-php-ext-install opcache

RUN $apt_get_install nginx

ADD ./project /var/www/project
RUN if [ -f /var/www/project/.env ]; then rm /var/www/project/.env; fi
WORKDIR /var/www/project

RUN $apt_get_autoremove
RUN $apt_get_clean

COPY ./docker/web/php.ini /usr/local/etc/php/
COPY ./docker/web/www.conf /usr/local/etc/php-fpm.d/zzz-www.conf
COPY ./docker/web/nginx.conf /etc/nginx/nginx.conf
COPY ./docker/web/supervisord.conf /etc/supervisor/
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]

EXPOSE 80
