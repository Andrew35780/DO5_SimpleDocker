FROM nginx

WORKDIR /home/

COPY ./nginx/nginx.conf /etc/nginx/
COPY server.c .
COPY run.sh .

RUN apt-get update; \
    apt-get install -y gcc spawn-fcgi libfcgi-dev; \
    rm -rf /var/lib/apt/lists/*; \
    chown -R nginx:nginx /etc/nginx/nginx.conf; \
    chown -R nginx:nginx /var/cashe/nginx; \
    chown -R nginx:nginx /home; \
    touch /var/run/nginx.pid; \
    chown -R nginx:nginx /var/run/nginx.pid; \
    chmod +x run.sh

USER nginx

ENTRYPOINT ["sh", "./run.sh"]
