FROM nginx:stable

ADD ./project /var/www/project
WORKDIR /var/www/project

COPY ./docker/nginx/nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
