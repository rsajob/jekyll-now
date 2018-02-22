---
layout: post
title: Consul - поднимаем на одной ноде в Docker
---

На днях немного начал разбираться c Consul. Основная загвоздка в том, что он по умолчанию предназначен для работы на нескольких нодах, а мне нужно было завести его в основном чисто для service-discovery в рамках одного проекта на одном сервере в Docker.

В результате пока пришол к такому:

```ymal

version: '3'
services:

  consul:
    image: progrium/consul
    hostname: consul
    command: -server -bootstrap -ui-dir /ui
    ports:
      - "8500:8500"

  registrator:
    image: gliderlabs/registrator
    links:
      - consul:consul.service.consul
    hostname: registrator
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock
    command: --internal consul://consul.service.consul:8500

```

Флаг `-bootstrap` говорит что нужно поднять только одну ноду, это важно. 

`registrator` - это штука которая автоматически регистрирует поднятые контейнеры в Сonsul

Для конфигурирования, для нужного контейнера нужно добавить переменные окружения `SERVICE_TAGS`, `SERVICE_NAME`, ` SERVICE_X_NAME`, ` SERVICE_X_TAGS`. В последних вместо `X` нужно указать порт(номер порта цифрами).

```ymal
  tarantool:
      image: progaudi/tarantool:1.7
      command: tarantool /opt/tarantool/app.init.lua
      environment:
        - TARANTOOL_USER_NAME=good
        - TARANTOOL_USER_PASSWORD=123456
        - SERVICE_TAGS=tnt
        - SERVICE_NAME=tnt
```
