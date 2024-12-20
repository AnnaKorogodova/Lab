# Использование Docker swarm и секретов
Docker swarm фактически является расширением docker обеспечивающее окрестрацию приложений внутри кластера и имеет отличительные особенности, которые управляют репликами приложений.
## Практические задания
### 1. Создайте собственный стек, подобный примеру.
Для создания нашего стэка я возьму в качестве примера compose файл для node js web server, который прослушивает порт 4003:
```sh
# docker-compose.stage.yaml

version: "3.9"

services:
  back:
    image: docker-registry.ru:5000/ptm:stage
    ports:
      - "4003:4003"
    environment:
      TZ: "Europe/Moscow"
    extra_hosts:
      - host.docker.internal:host-gateway
    command: make server_start
    volumes:
      - /p/ptm/config/config.yaml:/p/ptm/config/config.yaml
      - /p/ptm/stat/web:/p/ptm/stat/web
```
В начале необходимо достать image из registry и только затем задеплоить в наш кластер:
```sh
docker pull docker-registry.ru:5000/ptm:stage;
docker stack deploy --with-registry-auth -c ./docker-compose.stage.yaml stage;
```
Все эти команды надо выполнять на ```manager``` node. Опция ```--with-registry-auth``` позволяет передать авторизационные данные на worker ноды, для того чтобы использовался один и тот же образ из регистра. ```stage``` это имя нашего стэка.

Посмотреть список стэков можно с помощью:
```sh
docker stack ls
NAME        SERVICES   ORCHESTRATOR
stage       1          Swarm
```

