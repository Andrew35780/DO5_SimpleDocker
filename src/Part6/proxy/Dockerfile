FROM nginx

WORKDIR /proxy

RUN apt-get update

COPY nginx.conf /etc/nginx/nginx.conf

CMD nginx -g 'daemon off;'
