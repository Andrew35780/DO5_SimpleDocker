# Отчёт по DO5. Simple Docker

## Оглавление

   1. [Готовый докер](#part-1-готовый-докер)
   2. [Операции с контейнером](#part-2-операции-с-контейнером)
   3. [Мини веб-сервер](#part-3-мини-веб-сервер)
   4. [Свой докер](#part-4-свой-докер)
   5. [Dockle](#part-5-dockle)
   6. [Базовый Docker Compose](#part-6-базовый-docker-compose)


## Part 1. Готовый докер

- Установил **docker** командой `sudo snap install docker`.

- Взял официальный образ Docker с **nginx** и загрузил его с помощью `docker pull nginx`:

![alt text](Screenshots/image.png)

- Проверил наличие **докер-образа** через `docker images`:

![alt text](Screenshots/image-1.png)

- Запустил докер-образ через `docker run -d [image_id|repository]`:

![alt text](Screenshots/image-2.png)

- Проверил, что образ запустился через `docker ps`:

![alt text](Screenshots/image-3.png)

- Посмотрел информацию о контейнере через `docker inspect [container_id|container_name]`:

**Размер контейнера**
![alt text](Screenshots/image-4.png)

**Список замапленных портов**
![alt text](Screenshots/image-5.png)

**IP-адрес контейнера**
![alt text](Screenshots/image-6.png)

- Остановил докер-контейнер через `docker stop [container_id|container_name]`:

![alt text](Screenshots/image-7.png)

- Проверил, что контейнер остановился через `docker ps`

![alt text](Screenshots/image-8.png)

- Запустил докер с портами *80* и *443* в контейнере, замапленными на такие же порты на локальной машине, через команду 
`docker run -d -p 80:80 -p 443:443 nginx`:

![alt text](Screenshots/image-9.png)

- Проверил, что в браузере по адресу `localhost:80` доступна стартовая страница **nginx**:

![alt text](Screenshots/image-10.png)

![alt text](Screenshots/image-11.png)

- Перезапустил докер-контейнер через `docker restart [container_id|container_name]`
и проверил, что он запустился через `docker ps`:

![alt text](Screenshots/image-12.png)


## Part 2. Операции с контейнером

- Прочитал конфигурационный файл **nginx.conf** внутри докер-контейнера через команду `docker exec -it [container_id] cat /etc/nginx/nginx.conf`:

![alt text](Screenshots/image-13.png)

- Создал на локальной машине файл **nginx.conf** через *перенаправление вывода*:

![alt text](Screenshots/image-14.png)

- Настроил в нем по пути **/status** отдачу страницы статуса сервера nginx:

![alt text](Screenshots/image-15.png)

- Скопировал созданный файл **nginx.conf** внутрь докер-образа через команду `docker cp [container_id]:/etc/nginx`

![alt text](Screenshots/image-16.png)

- Проверил **nginx.conf** на корректность синтаксиса через `docker exec -it [container_id] nginx -t`:

![alt text](Screenshots/image-17.png)

- Перезагрузил **nginx** через `docker exec -it [container_id] nginx -s reload`:

![alt text](Screenshots/image-18.png)

- Проверил, что по адресу *localhost:80/status* отдается страничка со статусом сервера nginx:

![alt text](Screenshots/image-19.png)

- Экспортировал контейнер в файл *container.tar* через команду `docker export [container_id] > container.tar`.
- Затем, остановил контейнер через `docker stop [container_id]`:

![alt text](Screenshots/image-20.png)

- Проверяем что контейнер остановлен. После чего удаляем образ *nginx* командой `docker rmi [image_id|repository]`. При попытке удалить образ этой командой, появляется сообщение о том, что данный образ используется контейнером. Поэтому, выполняем ту же команду, но уже используя флаг `-f` для *принудительного удаления*:

![alt text](Screenshots/image-21.png)

- Далее, удалил остановленный контейнер командой `docker rm [container_id|container_name]`. Проверил отсуствие контейнера командой `docker ps -a`:

![alt text](Screenshots/image-22.png)

- Импортировал образ обратно через команду `docker import -c 'cmd ["nginx", "-g", "daemon off;"]' container.tar [image_name]`:

![alt text](Screenshots/image-23.png)

- Запустил импортированный образ командой `docker run -d -p 80:80 -p 443:443 [image_name]`:

![alt text](Screenshots/image-24.png)

- Запустил в контейнере терминал через команду `docker exec -it eager_colden bash`. В нём проверил состояние сервера nginx через команду `service nginx status`:

![alt text](Screenshots/image-25.png)

- Проверил, что по адресу `localhost:80/status` отдается страничка со статусом сервера nginx:

![alt text](Screenshots/image-26.png)


## Part 3. Мини веб-сервер

- Установил пакет **libfcgi-dev** на локальную машину.

- Написал мини-сервер на **C** и **FastCgi**, который возвращает простейшую страничку с надписью `Hello World!`:

![alt text](Screenshots/image-27.png)

- Запустил контейнер с помощью команды `docker run -d -p 81:81 custom_nginx`.

- С помощью команды `docker cp server.c [container id|container name]:/home` скопировал файл сервера в домашнюю директорию контейнера:

![alt text](Screenshots/image-28.png)

- Написал свой *nginx.conf*, который проксирует все запросы с *81* порта на *127.0.0.1:8080*:

![alt text](Screenshots/image-29.png)

- C помощью команды `docker cp nginx.conf [container id/container name]:/etc/nginx` скопировал конфиг в директорию nginx контейнера:

![alt text](Screenshots/image-30.png)

- Зашёл в контейнер командой `docker exec -it [container id|container name] bash` и терминале контейнера обновляем репозитории, и устанавливаем gcc, spawn-fcgi и libfcgi-dev:

![alt text](Screenshots/image-31.png)

- Скомпилировал исходный файл сервера в контейнере с помощью `gcc -o server server.c -lfcgi`. Перезапустил сервер nginx с помощью `nginx -s reload`. Запустил написанный мини-сервер через команду `spawn-fcgi -p 8080 ./server`:

![alt text](Screenshots/image-32.png)

- Проверил работу сервера в браузере по адресу `localhost:81`:

![alt text](Screenshots/image-33.png)

- Положил файл *nginx.conf* по пути `./nginx/nginx.conf`:

![alt text](Screenshots/image-34.png)


## Part 4. Свой докер

Написал свой докер-образ, основанный на базовом образе **nginx**, который:

1) Собирает исходники мини-сервера на FastCgi из Части 3;

2) Запускает его на 8080 порту;

3) Копирует внутрь образа написанный ./nginx/nginx.conf;

4) Запускает nginx.

![alt text](Screenshots/image-35.png)

- Собрал написанный *докер-образ* через `docker build . -t [image_name:image_tag]` при этом указав имя и тег:

![alt text](Screenshots/image-36.png)

- Проверил через `docker images`, что все собралось корректно:

![alt text](Screenshots/image-37.png)

- Запустил собранный Docker-образ с сопоставлением *81* порта с *80* на локальной машине, и сопоставлением папки `./nginx` внутри контейнера по адресу, где находятся конфигурационные файлы `nginx` с помощью команды `docker run -it -p 80:81 -v ./nginx/nginx.conf:/etc/nginx/nginx.conf -d quickbes_custom_nginx:1.0 bash`. Проверил что контейнер запустился через `docker ps`:

![alt text](Screenshots/image-38.png)

- Проверил, что по адресу `localhost:80` доступна страница написанного сервера:

![alt text](Screenshots/image-39.png)

- Дописал в `nginx.conf` проксирование странички */status*, по которой надо отдавать статус сервера nginx:

![alt text](Screenshots/image-40.png)

- Перезапустил контейнер через `docker restart [container_id|container_name]`:

![alt text](Screenshots/image-41.png)

- Проверил, что теперь по адресу `localhost:80/status` отображается страница со статусом nginx:

![alt text](Screenshots/image-42.png)


## Part 5. Dockle

- Установил утилиту *dockle* с помощью команд `VERSION=$(curl --silent "https://api.github.com/repos/goodwithtech/dockle/releases/latest" | \grep '"tag_name":' | \sed -E 's/.*"v([^"]+)".*/\1/' \) && curl -L -o dockle.deb https://github.com/goodwithtech/dockle/releases/download/v${VERSION}/dockle_${VERSION}_Linux-64bit.deb` и
`sudo dpkg -i dockle.deb && rm dockle.deb`

- Запустил проверку Docker-образа через *dockle* с помощью команды `dockle [image_id|repository]`:

![alt text](Screenshots/image-43.png)

- Исправил ошибки и предупреждения в `Dockerfile`:

![alt text](Screenshots/image-44.png)

```
Поскольку образ создается на основе официальной последней версии nginx, в Dockerfile которой используются NGINX_GPGKEY и NGINX_GPGKEY_PATH, это приводит к ложным срабатываниям предупреждений в Dockle, который в CIS-DI-0010 ищет ключевые слова (ключ, пароль).
```
- Поэтому, для проверки отсуствия ошибок и предупреждений, запустил Dockle с флагом -ak `dockle -ak NGINX_GPGKEY -ak NGINX_GPGKEY_PATH -ak NGINX_GPGKEYS quickbes_custom_nginx:1.1`:

![alt text](Screenshots/image-45.png)


## Part 6. Базовый Docker Compose

- Для того, чтобы собрать докер-контейнер на nginx, портирующий запросы с *8080* порта на *81* порт ранее запущенного контейнера необходимо изменить содержания конфигурационных файлов.

- Cоздал директорию `server`, где поменял настройки в файле *nginx.conf*:

![alt text](Screenshots/image-46.png)

- Поменял настройки в *Dockerfile* server:

![alt text](Screenshots/image-47.png)


- Создал файл **docker-compose.yml**:

![alt text](Screenshots/image-48.png)

- Остановил все запущенные контейнеры через команду `docker stop $(docker ps -q)`:

![alt text](Screenshots/image-49.png)

- Создал директорию `proxy`, где поменял настройки в файле *nginx.conf*:

![alt text](Screenshots/image-50.png)

- Поменял настройки в *Dockerfile* proxy:

![alt text](Screenshots/image-51.png)

- Собрал проект командой `docker-compose build`:

![alt text](Screenshots/image-52.png)

- Запустил проект командой `docker-compose up`:

![alt text](Screenshots/image-53.png)

- Проверил, что в браузере по адресу `localhost:80` доступна страница сервера:

![alt text](Screenshots/image-54.png)
