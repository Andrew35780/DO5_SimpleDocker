FROM debian

WORKDIR /server

RUN apt-get update; \
    apt-get install -y nginx gcc libfcgi-dev spawn-fcgi;\
    rm -rf /var/lib/apt/lists

COPY server.c .
COPY nginx.conf /etc/nginx/nginx.conf

RUN adduser --disabled-password base_user; \
    chown -R base_user:base_user /etc/nginx/; \
    chown -R base_user:base_user /server; \
    chown -R base_user:base_user /var/lib/nginx/; \
    chown -R base_user:base_user /var/log/nginx/; \
    chown -R base_user:base_user /run/ 

USER base_user

CMD gcc -o server server.c -lfcgi && spawn-fcgi -p 8080 server && nginx -g 'daemon off;'
