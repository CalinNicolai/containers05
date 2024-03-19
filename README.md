# Отчет по выполнению лабораторной работы №5

## Цель работы

Целью работы было создание PHP приложения на базе двух контейнеров: nginx и php-fpm, а также организация их
взаимодействия.

## Задание

1\. Создать репозиторий `containers05` и скопировать его на компьютер.\
2\. Перенести сайт на PHP в директорию `mounts/site.`\
3\. Создать файл `.gitignore` и добавить в него строку `mounts/site/*.`\
4\. Создать файл `nginx/default.conf` с конфигурацией nginx.\
5\. Создать сеть `internal` для контейнеров.\
6\. Создать контейнер `backend` на базе образа `php:7.4-fpm,`примонтировав директорию `mounts/site` и подключив его к
сети `internal.`\
7\. Создать контейнер `frontend` на базе образа `nginx:1.23-alpine,`примонтировав директорию `mounts/site` и
файл `nginx/default.conf,`пробросив порт 80 на хост, и подключив его к сети `internal.`

## Выполнение

1\. Создал репозиторий `containers05` и скопировал его на компьютер.\
2\. Перенес сайт на PHP в директорию `mounts/site.` сайт взят
из `https://github.com/CalinNicolai/Web-Programming-Lab-5`\
3\. Создал файл `.gitignore` и добавил в него строку `mounts/site/*`.\
4\. Создал файл `nginx/default.conf` с конфигурацией сервера nginx со следующим содержимым.

```conf
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

5\. Создал сеть `internal` для контейнеров.

```bash
docker network create internal
```

6\. Создал контейнер `backend` на базе образа `php:7.4-fpm`примонтировав директорию `mounts/site` и подключив его к
сети `internal`.

```bash
docker run -d --name backend --network internal -v /mounts/site:/var/www/html php:7.4-fpm
```

7\. Создал контейнер `frontend` на базе образа `nginx:1.23-alpine,`примонтировав директорию `mounts/site` и
файл `nginx/default.conf,`пробросив порт 80 на хост, и подключив его к сети `internal.`

```bash
docker run -d --name frontend --network internal -v /mounts/site:/var/www/html -v /nginx/default.conf:/etc/nginx/conf.d/default.conf -p 80:80 nginx:1.23-alpine
```

## Вопросы и ответы

1.** Каким образом в данном примере контейнеры могут взаимодействовать друг с другом?\

- Контейнер `frontend` обращается к контейнеру `backend` через сеть `internal,`используя сетевой адрес
  контейнера `backend.`

2.** Как видят контейнеры друг друга в рамках сети internal?\

- Контейнеры видят друг друга по именам контейнеров, которые являются их DNS именами внутри сети `internal.`

3.** Почему необходимо было переопределять конфигурацию nginx?\

- Необходимо было переопределить конфигурацию nginx для того, чтобы настроить проксирование запросов PHP скриптам на
  контейнере `backend.`

## Выводы

В результате выполнения лабораторной работы было успешно создано PHP приложение на базе двух контейнеров, обеспечивающих
работу веб-сервера nginx и PHP-обработчика php-fpm. Взаимодействие между контейнерами осуществляется через
сеть `internal,`что позволяет им обмениваться данными и обеспечивать работу веб-приложения.
