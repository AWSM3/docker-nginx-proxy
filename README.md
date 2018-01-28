Nginx Proxy
==========
Прокси-сервис.
--------------

Для локального взаимодействия между сервисами, запущенными под этим прокси необходимо объединить эти сервисы в одну сеть, предоставляемую прокси-сервисом.

Пример:
```
# File: ./docker-compose.yml -- конфиг какого-нибудь сервиса

services:
  # Web server
  web:
    image: nginx
    environment:
      - 'VIRTUAL_HOST=${NGINX_HOST}' # из .env
    restart: always
    networks:
        nginxproxy_proxy:
            aliases:
                - ${NGINX_HOST}
        default

  # PHP
  php:
    build: php
    networks:
      - nginxproxy_proxy
      - default


# ...
networks:
  # Название сети зависит от директории, в котором запущен прокси-сервис
  nginxproxy_proxy:
    external: true
```
Приводим конфигурацию второго сервиса к подобному виду.

Обратите внимание на алиас сети, он совпадает с хостом из `.env`:
```
networks:
    nginxproxy_proxy:
        aliases:
            - ${NGINX_HOST}
    default
```
Указание в качестве алиаса хоста нашего сервиса (локальный хост `api.local`, например)
позволит сервисам, объединённым таким образом общаться между собой по имени хоста.

Например, из одного http-сервиса, находящегося под управлением этого прокси
можно обратиться (допустим, через PHP) к другому сервису по его хост-имени:
```
# PHP, host: front.local

file_get_content('http://api.local/);
```
И наоборот, с `api.local` к `front.local`.