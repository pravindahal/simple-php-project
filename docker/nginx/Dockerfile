FROM nginx:stable

ADD ./project /var/www/project
RUN if [ -f /var/www/project/.env ]; then rm /var/www/project/.env; fi
WORKDIR /var/www/project

COPY ./docker/nginx/nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
